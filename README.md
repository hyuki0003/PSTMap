# Physiological STmap

> Official implementation of **"생리학적 혈류 방향 기반 시공간 특징맵과 교차 주의 집중을 이용한 비접촉 원격 심박수 추정"**
> (*Non-contact Remote Heart Rate Estimation via Physiology-guided Spatio-Temporal Feature Maps and Cross-Attention*) — 2025 IEIE Summer Conference.
>
> 📄 [Paper (DBpia)](https://www.dbpia.co.kr/Journal/articleDetail?nodeId=NODE12332223) · 이동혁, 최영석 (광운대학교)

This repository contains code for generating **Spatio-Temporal maps (STmaps)** from facial videos and for training neural networks for **remote photoplethysmography (rPPG)** and physiological signal estimation.

---

## Overview

Remote photoplethysmography (rPPG) is a non-contact technique for recovering physiological signals (heart rate, pulse waveform) from the subtle chromatic variations present in facial video. Conventional STmaps encode temporal dynamics well, but their **spatial layout typically ignores physiology** — facial regions are usually ordered by index or position, not by how blood actually flows through them.

This work addresses that gap with two ideas:

1. **Physiology-guided STmap.** Facial ROIs are re-arranged along the real direction of blood flow (**chin → forehead**), so that the spatial axis of the STmap carries a physiologically meaningful ordering in addition to the temporal axis.
2. **Cross-STmap Attention.** A 2D CNN with a cross-attention module that fuses complementary information from STmaps computed in **different color spaces (RGB and YUV)**.

Evaluated on **PURE** and **UBFC-rPPG**, the proposed approach outperforms conventional STmap-based baselines on heart rate estimation.

---

## Features

### STmap generation
- Frame extraction from `.avi` / `.mp4` videos
- Facial landmark detection and tracking with [`face_alignment`](https://github.com/1adrianb/face-alignment) for bounding-box cropping
- **YUV-based STmaps** encoding regional (or global) spatial pixel variations horizontally, stacked temporally along the x-axis
- **Physiology-guided region ordering** (chin → forehead) reflecting blood-flow direction
- Specialized pipelines for widely used rPPG datasets: **UBFC-rPPG**, **PURE**, **PHYS**

### Model architectures (`model/`)
- `STNet` — STmap-based regression backbone
- `Transfuser` — multi-stream STmap fusion model
- `STEM` — spatio-temporal extractor used with the proposed attention module
- `ResNet` variants for backbone comparison and ablation
- Attention modules: `SelfAttn`, `CrossAttn` (the Cross-STmap Attention used for RGB ↔ YUV fusion)

### Training pipeline
- `pretrain.py`, `train.py`, `train2.py`, `finetuning.py` — flexible end-to-end training phases
- `dataloader.py` — efficient batching for STmap datasets
- `eval.py` — benchmarking against ground-truth PPG / HR
- Configurable loss functions (`utils/`)

---

## Repository structure

```
├── STmap/        # Core STmap algorithms and dataset-specific scripts
│                 # (produces *_stmap_yuv.png and RGB variants)
├── STmap_lmks/   # Facial landmark caching and processing helpers
├── model/        # Network architectures and attention modules (SelfAttn, CrossAttn)
├── evaluation/   # Validation parsing and dataset inference scripts
└── utils/        # Training metrics and custom loss definitions
```

---

## Getting started

### 1. Environment

```bash
pip install torch torchvision opencv-python face-alignment numpy scipy pandas
```

Tested with PyTorch ≥ 1.13, CUDA 11.x.

### 2. Generate STmaps

Edit the raw-video path inside the dataset-specific script under `STmap/`, then run the one matching your dataset:

```bash
python STmap/ubfc_stmap.py     # UBFC-rPPG
python STmap/pure_stmap.py     # PURE
python STmap/phys_stmap.py     # PHYS
```

This produces `*_stmap_yuv.png` (and the RGB counterpart) per subject / session.

### 3. Train

Once the STmaps are built, feed them into the training scripts:

```bash
python train.py        # main supervised training
python pretrain.py     # optional pretraining phase
python finetuning.py   # dataset-specific finetuning
```

### 4. Evaluate

```bash
python eval.py
```

---

## Datasets

- [UBFC-rPPG](https://sites.google.com/view/ybenezeth/ubfcrppg)
- [PURE](https://www.tu-ilmenau.de/neurob/data-sets-code/pulse/)
- PHYS (multimodal physiological dataset)

Each dataset has its own landmark caching and STmap generation script under `STmap/`.

---

## Citation

If this repository is useful for your research, please cite:

```bibtex
@inproceedings{lee2025physiological,
  title     = {생리학적 혈류 방향 기반 시공간 특징맵과 교차 주의 집중을 이용한 비접촉 원격 심박수 추정},
  author    = {이동혁 and 최영석},
  booktitle = {2025년도 대한전자공학회 하계학술대회 논문집},
  pages     = {3471--3475},
  year      = {2025},
  organization = {대한전자공학회}
}
```

---

## Contact

Kwangwoon University — Dept. of Electronic Engineering
For questions, issues, or collaboration, please open a GitHub issue.
