# Topic 9 — Watermarking in Frequency Domain

## Course Project: Introduction to Information Security

This repository contains the final demonstration for **Topic 9: Watermarking in Frequency Domain**.  
The project implements a robust image watermarking system using **block-wise Discrete Cosine Transform (DCT)** and compares it with a spatial-domain **Least Significant Bit (LSB)** baseline.

The goal is to embed a copyright logo into a high-resolution image and demonstrate that the logo can still be extracted after common image attacks such as **JPEG compression** and **cropping**.

---

## 1. Project Overview

Digital watermarking is used to embed ownership or copyright information into digital media.  
In this project, a binary copyright logo is embedded into a cover image using frequency-domain watermarking.

The final demo includes:

- DCT-based frequency-domain watermark embedding.
- Watermark extraction after JPEG compression.
- Watermark extraction after cropping.
- LSB spatial-domain baseline for comparison.
- Wrong secret key test to demonstrate the role of key-based block selection.
- Evaluation using PSNR, NCC, and BER.

---

## 2. Main Idea

### Spatial Domain: LSB Watermarking

LSB watermarking hides watermark bits directly inside the least significant bits of image pixels.  
It produces very high image quality because only small pixel-level changes are made.

However, it is fragile. JPEG compression modifies pixel values and destroys the least significant bits, making the extracted watermark unreadable.

### Frequency Domain: DCT Watermarking

DCT watermarking first transforms image blocks from the spatial domain into the frequency domain.  
Instead of modifying raw pixels, the watermark is embedded into selected DCT coefficients.

This project embeds watermark bits into **mid-frequency DCT coefficients** because:

- Low-frequency coefficients are visually important and may cause visible distortion.
- High-frequency coefficients are aggressively removed by JPEG compression.
- Mid-frequency coefficients provide a good balance between invisibility and robustness.

---

## 3. Demo Scenario

The demo simulates a copyright protection scenario:

1. A copyright owner embeds a hidden logo into a high-resolution image.
2. An attacker tries to damage or remove the watermark using:
   - JPEG compression.
   - Cropping.
3. The verification system attempts to extract the watermark from the attacked image.
4. The extracted watermark is compared with the original logo using quantitative metrics.

---

## 4. Features

- Embed a binary copyright logo into a high-resolution image.
- Use block-wise 8×8 DCT, similar to JPEG compression.
- Embed watermark bits into mid-frequency DCT coefficients.
- Use secret-key pseudo-random block selection.
- Use redundant embedding and majority voting to improve cropping robustness.
- Test robustness under JPEG compression at Q=90, Q=50, Q=30, and Q=10.
- Test robustness under cropping at 10%, 25%, and 40%.
- Compare DCT watermarking with LSB watermarking.
- Test extraction using a wrong secret key.
- Export results to `output/results.csv`.
- Display visual comparison of extracted watermarks.

---

## 5. Repository Structure

Recommended repository structure:

```text
.
├── DCT.ipynb
├── original.jpg
├── logo.jpg
├── requirements.txt
├── Final_Report_Topic9_Watermarking_DCT_WITH_WRONG_KEY.docx
└── output/
    ├── results.csv
    ├── watermarked_dct.png
    ├── watermarked_lsb.png
    ├── dct_jpeg_q90.jpg
    ├── dct_jpeg_q50.jpg
    ├── dct_jpeg_q30.jpg
    ├── dct_jpeg_q10.jpg
    ├── dct_crop_10.png
    ├── dct_crop_25.png
    ├── dct_crop_40.png
    ├── extracted_dct_no_attack.png
    ├── extracted_dct_jpeg_q90.png
    ├── extracted_dct_jpeg_q50.png
    ├── extracted_dct_jpeg_q30.png
    ├── extracted_dct_jpeg_q10.png
    ├── extracted_dct_crop_10.png
    ├── extracted_dct_crop_25.png
    ├── extracted_dct_crop_40.png
    ├── extracted_dct_wrong_key.png
    ├── extracted_lsb_no_attack.png
    ├── extracted_lsb_jpeg_q90.png
    ├── extracted_lsb_jpeg_q50.png
    ├── extracted_lsb_jpeg_q30.png
    └── extracted_lsb_jpeg_q10.png
```

---

## 6. Requirements

The project requires Python 3 and the following libraries:

```text
numpy
opencv-python
matplotlib
```

These libraries are listed in `requirements.txt`.

Install dependencies using:

```bash
pip install -r requirements.txt
```

If using Jupyter Notebook or VS Code Jupyter, make sure the selected Python kernel is the environment where these packages are installed.

---

## 7. How to Run

### Step 1: Clone the repository

```bash
git clone <your-repository-link>
cd <repository-folder>
```

### Step 2: Install dependencies

```bash
pip install -r requirements.txt
```

### Step 3: Open the notebook

Open:

```text
DCT.ipynb
```

using Jupyter Notebook, JupyterLab, or VS Code.

### Step 4: Check input files

Make sure these files exist in the same folder as the notebook:

```text
original.jpg
logo.jpg
```

- `original.jpg`: high-resolution cover image.
- `logo.jpg`: copyright logo watermark.

### Step 5: Run all cells

In Jupyter:

```text
Kernel → Restart Kernel → Run All Cells
```

In VS Code:

```text
Restart Kernel → Run All
```

### Step 6: Check results

After running the notebook, check:

```text
output/results.csv
```

and the generated extracted watermark images inside the `output/` folder.

---

## 8. Important Parameters

The main parameters are defined in the notebook:

```python
WM_SIZE = (32, 32)
SECRET_KEY = 2026
DELTA = 55
REPEAT = 9
```

| Parameter | Meaning |
|---|---|
| `WM_SIZE` | Size of the binary watermark. The logo is resized to 32×32. |
| `SECRET_KEY` | Key used for pseudo-random block selection. The same key is required during extraction. |
| `DELTA` | Embedding strength. Higher values improve robustness but may reduce image quality. |
| `REPEAT` | Number of times each watermark bit is embedded. Higher values improve cropping robustness. |

---

## 9. DCT Embedding Method

The cover image is converted to YCrCb color space, and the watermark is embedded in the Y channel.

The image is divided into 8×8 blocks. For each selected block:

1. Apply DCT.
2. Select two mid-frequency coefficients:
   - `c1 = (4, 1)`
   - `c2 = (3, 2)`
3. Embed the bit using coefficient comparison.

Embedding rule:

```text
If bit = 1: force c1 > c2 + DELTA
If bit = 0: force c2 > c1 + DELTA
```

This creates a robust relationship between two DCT coefficients.

---

## 10. DCT Extraction Method

During extraction, the decoder uses the same secret key to locate the selected blocks.

For each selected block:

1. Apply DCT.
2. Compare the two coefficients.
3. Recover the bit:
   - If `c1 > c2`, extracted bit = 1.
   - Otherwise, extracted bit = 0.

Since each bit is embedded multiple times, the final bit is decided using majority voting.

---

## 11. Attack Tests

### JPEG Compression Attack

The watermarked DCT image is compressed at different JPEG quality levels:

| JPEG Quality | Attack Strength |
|---|---|
| Q=90 | Light compression |
| Q=50 | Medium compression |
| Q=30 | Strong compression |
| Q=10 | Very heavy compression |

### Cropping Attack

The image is cropped by:

```text
10%
25%
40%
```

The cropped image is padded back to the original size so that the decoder can process the same 8×8 block grid.

### Wrong Secret Key Test

A wrong key is used during extraction:

```python
WRONG_KEY = 9999
```

This test demonstrates that the correct secret key is required to locate the embedded watermark blocks.

---

## 12. Evaluation Metrics

### PSNR — Peak Signal-to-Noise Ratio

PSNR measures the visual quality of the watermarked or attacked image compared with the original image.

Higher PSNR means better image quality.

### NCC — Normalized Cross-Correlation

NCC measures similarity between the original watermark and the extracted watermark.

- NCC close to 1: very similar.
- NCC close to 0: not similar.
- Negative NCC: poor or incorrect extraction.

### BER — Bit Error Rate

BER measures the percentage of incorrect watermark bits.

- BER close to 0: successful extraction.
- BER close to 0.5: almost random extraction.

---

## 13. Final Results

| Method | Attack | Parameter | PSNR | NCC | BER |
|---|---|---:|---:|---:|---:|
| DCT | None | - | 34.6749 | 1.0000 | 0.0000 |
| DCT | JPEG Compression | Q=90 | 34.2301 | 1.0000 | 0.0000 |
| DCT | JPEG Compression | Q=50 | 27.6166 | 1.0000 | 0.0000 |
| DCT | JPEG Compression | Q=30 | 25.7393 | 1.0000 | 0.0000 |
| DCT | JPEG Compression | Q=10 | 22.5987 | 0.9783 | 0.0107 |
| DCT | Crop | 10% | - | 1.0000 | 0.0000 |
| DCT | Crop | 25% | - | 0.9980 | 0.0010 |
| DCT | Crop | 40% | - | 0.9824 | 0.0088 |
| DCT | Wrong Secret Key | Key=9999 | - | 0.0728 | 0.4639 |
| LSB | None | - | 88.5660 | 1.0000 | 0.0000 |
| LSB | JPEG Compression | Q=90 | 43.7468 | -0.0202 | 0.5254 |
| LSB | JPEG Compression | Q=50 | 28.6308 | -0.0486 | 0.5352 |
| LSB | JPEG Compression | Q=30 | 26.3844 | 0.0513 | 0.5264 |
| LSB | JPEG Compression | Q=10 | 23.1035 | -0.0069 | 0.4971 |

---

## 14. Result Interpretation

The DCT watermarking method successfully survives both JPEG compression and cropping.

At **JPEG Q=10**, the DCT watermark still achieves:

```text
NCC = 0.9783
BER = 0.0107
```

At **40% cropping**, the DCT watermark still achieves:

```text
NCC = 0.9824
BER = 0.0088
```

With the wrong secret key, extraction fails:

```text
NCC = 0.0728
BER = 0.4639
```

This proves that the correct key is required to locate the embedded watermark blocks.

The LSB baseline fails under JPEG compression. Its BER values are close to 0.5, meaning the extracted watermark is almost random.

---

## 15. Conclusion

This project demonstrates that frequency-domain watermarking using block-wise DCT is more robust than spatial-domain LSB watermarking.

The DCT watermark survives:

- JPEG compression up to Q=10.
- Cropping up to 40%.
- Verification with correct secret-key extraction.

In contrast, LSB watermarking fails after JPEG compression because JPEG modifies pixel values and destroys least significant bits.

Overall, the final demo satisfies the requirement of embedding a copyright logo into a high-resolution image and extracting it after compression and cropping.

---

## 16. Limitations

This implementation is a classroom proof-of-concept and focuses on the required attacks: JPEG compression and cropping.

Limitations:

- The cropping test pads the image back to the original size to preserve the 8×8 block grid.
- Rotation, scaling, affine transformation, strong blur, and combined attacks are not tested.
- The watermark payload is not encrypted; only the block positions are protected by the secret key.
- The watermark size is fixed at 32×32 in this demo.
- The implementation is not intended as production-level copyright protection.

---

## 17. Future Improvements

Possible future improvements:

- Add rotation and scaling attack tests.
- Add watermark scrambling or encryption before embedding.
- Add automatic tuning for `DELTA`.
- Test the algorithm on multiple cover images.
- Build a command-line version for easier reproduction.
- Add blind detection with stronger synchronization against geometric attacks.

---

## 18. References

1. Cox, I., Miller, M., & Bloom, J. *Digital Watermarking*. Morgan Kaufmann.
2. Gonzalez, R. C., & Woods, R. E. *Digital Image Processing*. Pearson.
3. Katzenbeisser, S., & Petitcolas, F. *Information Hiding Techniques for Steganography and Digital Watermarking*.
4. OpenCV Documentation.
5. NumPy Documentation.
6. Matplotlib Documentation.

---

## 19. Authors

- Nguyen Tran Ngoc Quy — 523H0087
- Pham Huynh Trinh Nam — 523H0058
