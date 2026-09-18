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

The notebook uses relative paths, so the project can be moved to another computer without changing an absolute dataset path.

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
├── outputs/
│   ├── denoised/
│   ├── hair_masks/
│   ├── hair_removed/
│   └── final/
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
