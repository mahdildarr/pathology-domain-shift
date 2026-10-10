# Breast-Cancer Metastasis Detection Across Hospitals

Can a CNN that finds breast-cancer metastases in lymph-node slides from one set of hospitals be trusted in another? This project studies that question on **Camelyon17-WILDS** (five hospitals): it compares stain normalisation, stain augmentation and test-time adaptation, and evaluates not only accuracy on the unseen hospital but also **calibration** and **selective prediction** — whether the model knows when to defer to a pathologist.

<p align="center"><img src="docs/assets/pipeline.svg" width="100%" alt="Pipeline"></p>

> **Status:** Work in progress — the pipeline is implemented and tested; results will be added once the Camelyon17-WILDS experiments are completed.
> This repository presents the project and its design. **The source code is kept in a private repository** and can be shared on request.

## Why this project

Lymph-node status is central to breast-cancer staging, and screening whole-slide images for small metastases is slow and error-prone. Deep learning can help — but every laboratory stains and scans differently, and a model that is excellent at home can degrade sharply elsewhere while remaining confident. For clinical use, the questions that matter are how much performance is lost on a new hospital, which remedies actually help, and whether the model's confidence can still be used to decide which patches a pathologist must review.

## What is inside

| | |
|---|---|
| **Data** | Camelyon17-WILDS: 96×96 px patches from 50 lymph-node slides of 5 hospitals |
| **Protocols** | official WILDS split · leave-one-hospital-out over all five hospitals |
| **Leakage control** | whole patients held out for validation; the unseen hospital never used for model selection |
| **Stain handling** | Macenko normalisation · HED augmentation · colour jitter · greyscale · Macenko + HED |
| **Test-time adaptation** | AdaBN: batch-norm statistics re-estimated on unlabelled patches of the new hospital |
| **Networks** | DenseNet-121 · ResNet-50 · EfficientNet-B0 (ImageNet-pretrained) |
| **Evaluation** | accuracy, AUROC, per-slide accuracy · expected calibration error, temperature scaling · accuracy at reduced coverage |

## Design

The full design — data, split protocols, stain-handling methods, networks, evaluation and limitations — is in **[docs/DESIGN.md](docs/DESIGN.md)**.

## Results

Result tables will be published here when the experiments are completed.

| Stain handling | ID accuracy | Unseen-hospital accuracy | + AdaBN | ECE (unseen) | Accuracy @ 80 % coverage |
|---|---|---|---|---|---|
| none | _TBD_ | _TBD_ | _TBD_ | _TBD_ | _TBD_ |
| HED augmentation | _TBD_ | _TBD_ | _TBD_ | _TBD_ | _TBD_ |
| Macenko normalisation | _TBD_ | _TBD_ | _TBD_ | _TBD_ | _TBD_ |
| Macenko + HED | _TBD_ | _TBD_ | _TBD_ | _TBD_ | _TBD_ |

## Tech stack

Python · PyTorch · torchvision · scikit-image · NumPy · pandas · Matplotlib · Google Colab (T4 GPU)

## Author

**Şule Özdamar** — Computer Engineering, İstanbul Arel University · [github.com/mahdildarr](https://github.com/mahdildarr)

Data: Camelyon17-WILDS (Koh et al., 2021), derived from the CAMELYON17 challenge (Bándi et al., 2019); CC0.
