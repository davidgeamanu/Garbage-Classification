<div align="center">

# Garbage Image Classification

### Transfer learning compared with a small CNN trained from scratch, on a 10-class household-waste dataset.

[![Python](https://img.shields.io/badge/Python-3.10+-blue?style=for-the-badge&logo=python&logoColor=white)](https://www.python.org/)
[![PyTorch](https://img.shields.io/badge/PyTorch-2.x-EE4C2C?style=for-the-badge&logo=pytorch&logoColor=white)](https://pytorch.org/)
[![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-F37626?style=for-the-badge&logo=jupyter&logoColor=white)](https://jupyter.org/)

*ResNet18 fine-tuning vs a VGG-style CNN from scratch - side-by-side training, evaluation, and per-class breakdown on 10 categories of household waste*

[Overview](#overview) •
[Dataset](#dataset) •
[Getting Started](#getting-started) •
[Results](#results)

</div>

---

## Table of Contents

- [Overview](#overview)
- [Dataset](#dataset)
- [Models](#models)
- [Getting Started](#getting-started)
- [Project Structure](#project-structure)
- [Results](#results)
- [Author](#author)

---

## Overview

This project compares two approaches to image classification on real-world garbage photos:

<table>
<tr>
<td width="50%">

### Transfer Learning (ResNet18)
- Load pretrained ImageNet weights
- Freeze the backbone, replace the final layer
- Train only the new classifier head
- 5 epochs, ~5k trainable parameters

</td>
<td width="50%">

### CNN From Scratch
- VGG-style architecture built by hand
- 4 conv blocks (32 to 256 channels)
- Train everything from random init
- 15 epochs, ~440k parameters

</td>
</tr>
</table>

Both models share the same data pipeline, class-weighted loss, and cosine LR schedule so the comparison is fair.

---

## Dataset

The notebook uses [Garbage Classification v2](https://www.kaggle.com/datasets/sumn2u/garbage-classification-v2) from Kaggle.

| Class | Images | Class | Images |
|-------|--------|-------|--------|
| battery | 756 | metal | 930 |
| biological | 699 | paper | 1336 |
| cardboard | 1411 | plastic | 1597 |
| clothes | 1892 | shoes | 1449 |
| glass | 1736 | trash | 453 |

**Total: 12,259 images across 10 classes**

The classes are noticeably imbalanced (clothes has roughly 4x more images than trash). A class-weighted cross-entropy loss is used to compensate.

### Download

You need a Kaggle account and an API token saved at `~/.kaggle/kaggle.json`. Then either:

```bash
# Kaggle CLI
kaggle datasets download -d sumn2u/garbage-classification-v2
unzip garbage-classification-v2.zip -d data/
```

or uncomment the relevant cell near the top of the notebook to download automatically with `opendatasets`.

The notebook expects images at `data/standardized_256/<class_name>/`.

---

## Models

### ResNet18 (Transfer Learning)

The pretrained ResNet18 backbone is frozen. Only the final fully-connected layer (512 x 10) is trained.

| Parameter | Value |
|-----------|-------|
| Backbone | ResNet18 pretrained on ImageNet |
| Trainable params | ~5,100 (head only) |
| Optimizer | Adam lr=1e-3 |
| LR schedule | Cosine annealing |
| Epochs | 5 |

### GarbageCNN (From Scratch)

A small VGG-style network: 4 conv blocks followed by a two-layer classifier with dropout.

| Parameter | Value |
|-----------|-------|
| Architecture | 4x [Conv3x3 + BatchNorm + ReLU + MaxPool] |
| Channel progression | 3 -> 32 -> 64 -> 128 -> 256 |
| Total params | ~440,000 |
| Optimizer | Adam lr=1e-3, weight_decay=1e-4 |
| LR schedule | Cosine annealing |
| Epochs | 15 |

---

## Getting Started

### Prerequisites

| Tool | Version |
|------|---------|
| Python | 3.10+ |
| PyTorch | 2.x |
| torchvision | 0.x |
| Pillow | - |
| matplotlib | - |
| numpy | - |

Install all dependencies:

```bash
pip install -r requirements.txt
```

### Running the Notebook

1. Download the dataset (see [Dataset](#dataset) above)
2. Confirm images are at `data/standardized_256/<class_name>/`
3. Open `garbage_classification.ipynb` and run top to bottom

Training runs on CPU by default. Expect around 50-75 minutes for ResNet18 (5 epochs) and 60-120 minutes for the custom CNN (15 epochs).

---

## Project Structure

```
garbage-classification/
├── garbage_classification.ipynb    # main notebook - data prep, training, evaluation
├── requirements.txt                # python dependencies
├── .gitignore
└── README.md
```

The `data/` and `models/` folders are gitignored. You generate them locally by downloading the dataset and running the notebook.

---

## Results

| Model | Trainable Params | Epochs | Test Accuracy |
|-------|-----------------|--------|---------------|
| ResNet18 (transfer) | ~5,000 | 5 | **83.4%** |
| GarbageCNN (scratch) | ~440,000 | 15 | 55.3% |

The gap comes down to pretrained features. ResNet18 already knows how to find edges, textures, and shapes - the visual primitives needed to separate cardboard from glass or shoes from plastic. The custom CNN has to learn all of that from scratch on a dataset that is relatively small, with some classes having under 500 training examples.

### Per-class breakdown (ResNet18)

| Class | Precision | Recall | F1 |
|-------|-----------|--------|----|
| clothes | 0.98 | 0.96 | 0.97 |
| shoes | 0.90 | 0.97 | 0.94 |
| biological | 0.98 | 0.85 | 0.91 |
| metal | 0.56 | 0.92 | 0.70 |
| trash | 0.78 | 0.60 | 0.68 |

Metal and trash are the hardest classes. Metal gets confused with other hard materials, and trash has the fewest training examples so the model has less to work with.

---

## Author

**David Geamanu**

---

## License

This project is available for educational purposes.
