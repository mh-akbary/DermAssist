# DermAssist Preprocessing

A simple Computer Vision project for skin image preprocessing using the PH2 dataset.

## Dataset

The dataset is the PH2-dataset from Zenodo:

https://zenodo.org/records/17498821

The dataset archive contains `ph2_dataset/trainx` and `ph2_dataset/trainy`, and also the corresponding `trainx` and `trainy` folders used by this project.

## Input Folders

The notebook uses only these two image folders:

```text
PH2/trainx/
PH2/ph2_dataset/trainx/
```

The `trainy` folders are not used as input images.

The lesion masks in `trainy` may be used only as a protection mask when the image and lesion-mask filenames match correctly.

## Processing Pipeline

```text
Two trainx folders
        |
        v
Dataset Check
        |
        v
Show Original Images
        |
        v
Denoising Test
        |
        +-- Mean
        +-- Gaussian
        +-- Median
        +-- Bilateral
        +-- Fourier
        |
        v
Denoising Comparison
        |
        v
Select Denoising Method
        |
        v
Denoise All Images
        |
        v
Dark + Bright Hair Detection
        |
        v
Hair Mask
        |
        v
Hair Removal
        |
        v
Telea / Navier-Stokes
        |
        v
Select Inpainting Method
        |
        v
Final Images
```

## Denoising Methods

The notebook tests:

- Mean 3x3
- Mean 5x5
- Gaussian 3x3
- Gaussian 5x5
- Median 3x3
- Median 5x5
- Bilateral
- Fourier low-pass filtering

The denoising methods are compared using the available image results and edge preservation.

## Denoising Evaluation

MSE, RMSE, PSNR and SSIM need a valid clean reference image.

The PH2 `trainy` files are lesion masks, not clean versions of the `trainx` images. Therefore, the notebook does not use them as fake Ground Truth.

When no clean reference is available, the notebook uses edge preservation and visual comparison.

## Hair Detection

The notebook tests dark and bright hair.

Dark hair uses grayscale and Black-Hat processing.

Bright hair uses a simple LAB and Top-Hat approach.

The two masks are combined into one hair mask.

## Hair Segmentation

A binary hair mask is created and cleaned with simple morphological operations.

## Hair Removal

The hair mask is used before inpainting.

The lesion area is protected when a matching lesion mask can be found.

## Morphological Methods

The notebook uses:

- Black-Hat
- Top-Hat
- Morphological Opening
- Morphological Closing
- Dilation

## Inpainting

Two OpenCV methods are compared:

- Telea
- Navier-Stokes

The methods are tested on real images, and the final method can be selected based on the obtained results.

## Output Folders

```text
outputs/
├── denoised/
├── hair_masks/
├── hair_removed/
└── final/
```

The original PH2 images are not overwritten.

## How to Run

This repository contains two notebooks that are meant to be run in order:

1. `notebooks/DermAssist_Preprocessing.ipynb` — removes hair and prepares clean skin images.
2. `DermAssist_Lesion_Features_Classification.ipynb` — uses the hair-removed images to segment the lesion and analyze its features.

### Step 1 — Run the Preprocessing Notebook

1. Put the project files in the project folder.
2. Keep the PH2 dataset in the expected folder structure.
3. Open:

```text
notebooks/DermAssist_Preprocessing.ipynb
```

4. Run the cells from top to bottom.
5. Check the denoising comparison and selected method.
6. Check the inpainting comparison and selected method.
7. Run the remaining cells.
8. Check the generated images in the output folders.

### Step 2 — Run the Lesion Analysis Notebook

The second notebook uses the hair-removed images that the first notebook produced (the Telea outputs). Make sure the preprocessing notebook has already finished and the `outputs/Telea/` folder exists before continuing.

1. Keep the PH2 dataset in the expected folder structure.
2. Open:

```text
DermAssist_Lesion_Features_Classification.ipynb
```

3. Run the cells from top to bottom.
4. Review the segmentation comparison for a few samples.
5. Review the segmentation metrics table.
6. Review the PCA and KMeans plots.
7. Check the generated masks and overlays in the output folders.

Both notebooks use relative paths, so the project can be moved to another computer without changing an absolute dataset path.

## Requirements

Install the packages with:

```bash
pip install -r requirements.txt
```

## GitHub Structure

```text
DermAssist/
├── PH2/
│   ├── trainx/
│   ├── trainy/
│   └── ph2_dataset/
│       ├── trainx/
│       └── trainy/
├── notebooks/
│   └── DermAssist_Preprocessing.ipynb
├── DermAssist_Lesion_Features_Classification.ipynb
├── outputs/
│   ├── denoised/
│   ├── hair_masks/
│   ├── hair_removed/
│   ├── final/
│   ├── Telea/
│   └── lesion_segmentation/
│       ├── predicted_masks/
│       └── overlays/
├── README.md
├── requirements.txt
└── .gitignore
```

Do not upload the full dataset or all generated images to GitHub.

Keep large dataset and output files local unless you have a suitable storage method.

## Notes

The notebook removes exact duplicate input files by file hash.

It does not overwrite the original dataset.

Progress is printed during large processing loops.

---

# DermAssist — Lesion Segmentation and Feature Analysis

A follow-up notebook to the DermAssist preprocessing stage. It takes the hair-removed (Telea) images and performs lesion segmentation and feature analysis using the PH2 reference masks.

## Overview

This notebook, `DermAssist_Lesion_Features_Classification.ipynb`, is the second stage of the DermAssist pipeline. It uses the final hair-removed images produced by `DermAssist_Preprocessing.ipynb` and builds a complete lesion analysis pipeline:

1. Detect lesion regions with a simple color-based Otsu thresholding baseline
2. Compare predicted masks with PH2 reference masks
3. Extract features from the detected lesion regions
4. Reduce dimensionality and cluster the features with PCA and KMeans

A supervised classifier was not trained in this notebook because the PH2 reference masks do not carry class labels for the lesions. The notebook focuses on segmentation quality and on how the extracted features group together.

## Input Folders

The notebook uses:

```text
outputs/Telea/                # hair-removed images from the preprocessing stage
PH2/trainy/                   # PH2 lesion reference masks
PH2/ph2_dataset/trainy/       # PH2 lesion reference masks (alternate path)
```

The images and masks are matched by normalized filename keys, so differences such as `_mask`, `-mask`, `_lesion`, `-lesion` and `_segmentation` are handled automatically.

## Lesion Segmentation Pipeline

```text
Hair-removed image
        |
        v
Grayscale + Gaussian Blur
        |
        v
Otsu Threshold (Inverse Binary)
        |
        v
Morphological Closing + Opening
        |
        v
Largest Connected Component
        |
        v
Predicted Lesion Mask
        |
        v
Save mask + overlay
        |
        v
Compare with PH2 reference mask
```

### Segmentation Method

A simple color-based baseline is used:

- Convert the image to grayscale
- Apply Gaussian blur (5×5)
- Apply **Otsu thresholding** (inverse binary)
- Morphological closing and opening (7×7 elliptical kernel)
- Keep the largest connected component as the lesion

The baseline is intentionally simple. It works well on high-contrast lesions and motivates future deep-learning based segmentation.

## Segmentation Evaluation

Predicted masks are compared with the PH2 reference masks using three metrics:

| Metric | Description | Desired Direction |
|--------|-------------|-------------------|
| Dice Coefficient | Overlap similarity between predicted and reference masks | Higher |
| IoU (Jaccard Index) | Intersection over Union | Higher |
| Pixel Disagreement (%) | Fraction of pixels where prediction and reference differ | Lower |

### Sample Results

| Image | Mask | Dice | IoU | Pixel Disagreement (%) |
|-------|------|------|-----|------------------------|
| IMD002.bmp | IMD002_lesion.bmp | 0.927 | 0.863 | 3.79 |
| IMD003.bmp | IMD003_lesion.bmp | 0.942 | 0.891 | 1.30 |
| IMD004.bmp | IMD004_lesion.bmp | 0.956 | 0.916 | 1.73 |
| IMD006.bmp | IMD006_lesion.bmp | 0.706 | 0.546 | 6.78 |
| IMD008.bmp | IMD008_lesion.bmp | 0.499 | 0.333 | 7.81 |

The Otsu-based baseline performs well on high-contrast lesions but degrades on low-contrast or irregular lesions, which is expected for a classical segmentation method.

## Visual Analysis

For each sample, the notebook displays:

- Input dermoscopy image (hair-removed)
- Predicted lesion mask overlaid on the image
- PH2 reference mask

The overlay uses a red highlight over the detected lesion region and is saved to disk for further inspection.

## Feature Extraction

Features are extracted from the segmented lesion regions. These features describe the lesion region in terms of color, shape, and texture, and they form the input for the unsupervised analysis.

## Unsupervised Analysis

Since the PH2 reference masks do not include class labels, the notebook does not train a supervised classifier. Instead, it performs:

- **StandardScaler** for feature normalization
- **PCA** for dimensionality reduction and 2D visualization of the feature space
- **KMeans** for unsupervised clustering of lesion features

These steps help to see natural groupings in the data and to explore whether the extracted features carry useful structure.

## Output Folders

```text
outputs/lesion_segmentation/
├── predicted_masks/     # predicted lesion masks (PNG)
└── overlays/            # colored overlays (PNG)
```

The original PH2 images and reference masks are never overwritten.

## Key Findings

- The Otsu-based baseline achieves Dice > 0.9 on high-contrast lesions.
- Dice drops below 0.5 on low-contrast or irregular lesions, showing the limits of a classical threshold-based method.
- Pixel disagreement grows quickly when segmentation fails, and is a useful sanity check.
- PCA and KMeans reveal natural groupings in the feature space, useful for further investigation.
- Without lesion class labels in PH2, a supervised classifier is not trained in this stage.

## Citation

Mendonça, T., Ferreira, P. M., Marques, J. S., Marcal, A. R. S., & Rozeira, J. (2013).
PH2 - A dermoscopic image database for research and benchmarking.
*2013 35th Annual International Conference of the IEEE Engineering in Medicine and Biology Society (EMBC)*, 5437–5440.

## Conclusion

This notebook extends the DermAssist preprocessing stage with a complete lesion segmentation, evaluation, and feature analysis pipeline.

The combination of a simple baseline segmentation method and quantitative metrics (Dice, IoU, Pixel Disagreement) gives a solid foundation for the DermAssist system.

Since the PH2 reference masks do not carry lesion class labels, this notebook does not train a supervised classifier. The work here stays focused on segmentation quality and on extracting features from the detected lesion regions.

## Author

**Mohadese Akbary**

GitHub: @mh-akbary
