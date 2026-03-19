# ClearSkies 🛰️
### Wildfire Smoke Removal from Satellite Imagery Using Deep Graph Total Variation

ClearSkies is a satellite image restoration system that removes wildfire smoke from Sentinel-2 imagery using Deep Graph Total Variation (DGTV) — a hybrid architecture combining graph signal processing with compact deep learning. Compared to pure deep learning methods (e.g., DnCNN), DGTV achieves competitive performance with 80% fewer parameters.

> **Target region:** British Columbia, Canada — 2023 wildfire season  
> **Success criteria:** PSNR ≥ 28 dB, SSIM ≥ 0.75, model size < 25 MB

---

## Repository Structure

```
ClearSkies/
├── data/
│   └── README_data.md          # Instructions to download Sentinel-2 data
├── patches_augmented/          # Preprocessed .npy files (generated locally)
│   ├── train_clean_aug.npy
│   ├── train_noisy_aug.npy
│   ├── test_clean.npy
│   └── test_noisy.npy
├── DGTV_Preprocessing_Augmentation.py   # Step 1–6: preprocessing pipeline (run locally)
├── DGTV_Model.py               # DGTV model architecture (Colab)  [coming soon]
├── DGTV_Train.py               # Training script (Colab)           [coming soon]
├── DGTV_Evaluate.py            # Evaluation & visualization        [coming soon]
├── requirements.txt
└── README.md
```

---

## Dependencies

Install all required libraries with:

```bash
pip install -r requirements.txt
```

**requirements.txt**
```
rasterio
numpy
matplotlib
scikit-image
scikit-learn
torch
torchvision
```

> **Note:** Model training is run on Google Colab (GPU). Preprocessing is run locally.

---

## Dataset

### Source
Sentinel-2 Level-1C imagery from the [Copernicus Data Space Ecosystem](https://dataspace.copernicus.eu).

### Coverage
| Type | Period | Description |
|---|---|---|
| Clean (5 images) |
| Smoky (5 images) |

**Regions covered:** Kelowna/West Kelowna, Shuswap Lake, Interior BC

### How to access the data
1. Create a free account at https://dataspace.copernicus.eu
2. Search for Sentinel-2 Level-1C tiles over BC wildfire regions(T10UGA)
3. Filter by date 
4. Download `.SAFE` format files and place them as follows:

```
DGTV_Project/
├── clean/
│   ├── clean_20230501.SAFE
│   ├── clean_20230515.SAFE
│   └── ...
└── smoky/
    ├── smoky_20230801.SAFE
    ├── smoky_20230815.SAFE
    └── ...
```

---

## How to Run

### Step 1 — Preprocessing (Local)

Run the full preprocessing pipeline locally:

```bash
python DGTV_Preprocessing_Augmentation.py
```

This script performs the following steps:
1. Loads Sentinel-2 `.SAFE` files and extracts RGB bands (B02, B03, B04)
2. Separates clean and smoky images
3. Extracts real smoke layers via pixel-wise subtraction (`smoky − clean`)
4. Extracts 256×256 patches (50 per image for train, 20 for test)
5. Normalizes patches and synthesizes noisy/clean pairs using random smoke intensity
6. Applies data augmentation ×8 (rotations + flips)

Output is saved to `./DGTV_Project/patches_augmented/`.

> Edit `BASE_DIR` at the top of the script to match your local folder path.

### Step 2 — Upload to Google Drive

Upload the `patches_augmented/` folder to your Google Drive:
```
MyDrive/DGTV_Project/patches_augmented/
```

### Step 3 — Model Training (Google Colab)

Open `DGTV_Train.py` in Google Colab and run all cells.  
*(Script coming in Week 8)*

### Step 4 — Evaluation (Google Colab)

Run `DGTV_Evaluate.py` to compute PSNR/SSIM and generate visualizations.  
*(Script coming in Week 9)*

---

## Scripts

| Script | Run on | Description |
|---|---|---|
| `DGTV_Preprocessing_Augmentation.py` | Local | Full preprocessing pipeline: band extraction, smoke synthesis, patch extraction, augmentation |
| `DGTV_Model.py` | Colab | DGTV model architecture (CNN_F, CNN_μ, graph construction, GTV filter, Lanczos approximation) |
| `DGTV_Train.py` | Colab | End-to-end training with SGD, MSE loss, early stopping |
| `DGTV_Evaluate.py` | Colab | PSNR/SSIM evaluation and visual comparison against BM3D, DnCNN, DGLR |

---

## Method Overview

ClearSkies implements the DGTV architecture with the following components:

1. **Feature learning (CNN_F):** A lightweight 4-layer CNN extracts 3 features per pixel, capturing semantic similarity between neighboring pixels.
2. **Graph construction:** An adaptive graph is built using Gaussian kernel edge weights based on feature distances.
3. **GTV filtering:** Graph Total Variation denoising is solved iteratively via l1-Laplacian reformulation, promoting piecewise-smooth reconstruction.
4. **Lanczos approximation:** Fast spectral filter implementation (order M=20) replaces matrix inversion for efficient inference.
5. **Weight estimation (CNN_μ):** A small CNN adaptively estimates the regularization parameter per patch.

---

## Results

*(To be updated after model training in Week 9)*

| Method | # Parameters | PSNR (dB) | SSIM |
|---|---|---|---|
| BM3D | — | — | — |
| DnCNN | 560K | — | — |
| DGLR | 120K | — | — |
| **DGTV (ours)** | **120K** | **—** | **—** |

---

## References

- Vu, H., Cheung, G., & Eldar, Y. C. (2021). Unrolling of Deep Graph Total Variation for Image Denoising. *arXiv:2010.11290.*

---
