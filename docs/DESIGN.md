# Design

How the experiments are designed: the data and split protocols, the stain-handling methods, the networks, and the evaluation of accuracy, calibration and selective prediction on hospitals the model has never seen.

## 1. Problem

Whether breast cancer has spread to the lymph nodes decides staging and treatment, and finding small metastases in whole-slide images is slow, tiring work for pathologists. CNNs detect metastases well **inside the hospital they were trained in** — but slides from another hospital look different: staining protocols, reagents, section thickness and scanners all change the colours and textures. A model that is 95 % accurate at home can drop sharply elsewhere, and, worse, it often stays confident while it is wrong.

This project measures that drop and compares the main remedies on the same footing: what helps a CNN generalise to an unseen hospital, and does it know when it is unsure?

## 2. Data

**Camelyon17-WILDS** (Koh et al., 2021), derived from the CAMELYON17 challenge (Bándi et al., 2019): 96 × 96 px patches from lymph-node whole-slide images of **five hospitals** (50 slides). Label: whether the central 32 × 32 px region contains tumour tissue. Every patch carries its hospital and slide. Licence: CC0.

## 3. Split protocols

| Protocol | Train | Model selection | Test | Question |
|---|---|---|---|---|
| **Official hospitals** | hospitals 0, 3, 4 | held-out patients of 0, 3, 4 | hospital 2 | the standard WILDS hospital split |
| **Leave-one-hospital-out** | four hospitals | held-out patients of those four | the fifth | does the conclusion depend on *which* hospital is unseen? |

**Validation without leakage.** The official in-distribution validation set consists of random patches from the *same slides* as the training patches; neighbouring patches of one slide are near-duplicates, so it overestimates performance and can mislead model selection. Here all slides of held-out **patients** are used instead (one patient per training hospital — several patients contribute more than one slide, so holding out single slides would still share patients and staining batches with training). The official variant is kept as a switch so the two can be compared.

**Model selection.** The unseen hospital is **never** used to select the reported model. WILDS itself selects on the out-of-distribution validation hospital 1; that checkpoint is also stored and its test result reported separately as "oracle selection", which makes the comparison with WILDS numbers possible and shows how much the choice of selection data matters. Other deliberate differences from the WILDS reference: ImageNet-pretrained networks (WILDS trains DenseNet-121 from scratch; `pretrained: false` reproduces that) and, for the Colab budget, a fresh random 30 % of the training patches in each epoch.

## 4. Stain handling — the main comparison

| Method | Training | Test | Idea |
|---|---|---|---|
| none | – | – | baseline |
| colour jitter | brightness / contrast / saturation / hue | – | generic colour augmentation |
| **HED augmentation** (Tellez et al., 2019) | random scaling and shift of the haematoxylin, eosin and DAB channels | – | make the network invariant to stain variation |
| **Macenko normalisation** (Macenko et al., 2009) | normalise to a reference | normalise to the same reference | remove stain variation from the input |
| Macenko + HED | both | normalise | combination |
| greyscale | greyscale | greyscale | lower bound on how much colour matters |

The Macenko reference is fitted on tissue-rich patches balanced over the training hospitals only. Geometric augmentation (90° rotations and flips — tissue has no canonical orientation) is used in every setting.

**Test-time adaptation.** In addition, **AdaBN** (Li et al., 2018) re-estimates the batch-normalisation statistics on the unlabelled patches of the unseen hospital — one pass over all of its patches in shuffled batches, with only the batch-norm layers in training mode. It uses no labels and no gradient steps, so it is applicable in practice whenever a batch of the new hospital's slides is available.

## 5. Networks and training

| Network | Pretraining | Parameters | Role |
|---|---|---|---|
| DenseNet-121 | ImageNet | ≈ 7 M | WILDS reference architecture |
| ResNet-50 | ImageNet | ≈ 24 M | common transfer-learning baseline |
| EfficientNet-B0 | ImageNet | ≈ 4 M | compact model |

Binary cross-entropy, AdamW, one-cycle learning-rate schedule, mixed precision. Camelyon17-WILDS results are known to vary strongly between random seeds, so every configuration is trained with **three seeds** (same data split, different initialisation and sampling) and reported as mean ± standard deviation.

## 6. Evaluation

| Aspect | Metric |
|---|---|
| Discrimination | accuracy, balanced accuracy, AUROC; per-slide accuracy on the unseen hospital |
| Generalisation gap | in-distribution validation vs unseen-hospital accuracy |
| Calibration | expected calibration error (15 bins), negative log-likelihood, reliability diagrams; before and after temperature scaling (Guo et al., 2017) fitted on in-distribution validation |
| Selective prediction | accuracy on the 90 / 80 / 70 / 50 % most confident patches — the setting in which uncertain patches are referred to a pathologist |
| Stain shift | per-hospital haematoxylin / eosin profiles; example patches before and after normalisation |

Temperature scaling is fitted on the training hospitals. Whether it still helps on an unseen hospital — or whether calibration breaks down exactly where it is needed — is one of the questions of the study.

## 7. Limitations

* Patch-level labels only; slide- and patient-level metastasis classification (as in CAMELYON17) is not addressed.
* Five hospitals from one country and one challenge; other tissues and scanners may shift differently.
* Macenko normalisation fails on patches with little tissue; these are passed through unchanged.

## References

1. P. Bándi et al., "From Detection of Individual Metastases to Classification of Lymph Node Status at the Patient Level: The CAMELYON17 Challenge," *IEEE Transactions on Medical Imaging*, 38(2), 2019.
2. P. W. Koh et al., "WILDS: A Benchmark of in-the-Wild Distribution Shifts," *ICML*, 2021.
3. M. Macenko et al., "A method for normalizing histology slides for quantitative analysis," *IEEE ISBI*, 2009.
4. D. Tellez et al., "Quantifying the effects of data augmentation and stain color normalization in convolutional neural networks for computational pathology," *Medical Image Analysis*, 58, 2019.
5. Y. Li, N. Wang, J. Shi, X. Hou, J. Liu, "Adaptive Batch Normalization for practical domain adaptation," *Pattern Recognition*, 80, 2018.
6. C. Guo, G. Pleiss, Y. Sun, K. Q. Weinberger, "On Calibration of Modern Neural Networks," *ICML*, 2017.
7. Y. Geifman, R. El-Yaniv, "Selective Classification for Deep Neural Networks," *NeurIPS*, 2017.
