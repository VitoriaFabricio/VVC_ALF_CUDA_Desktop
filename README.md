# CUDA VVC Adaptive Loop Filter (ALF)

This repository contains a GPU-accelerated (CUDA) implementation of the **Adaptive Loop Filter (ALF)** luma filter, as defined in the **VVC (Versatile Video Coding / H.266)** video coding standard.

The code performs block classification and pixel filtering, supporting **10-bit** color depth, ensuring high precision and performance through shared memory optimizations.

## 🚀 Features

* **Block Classification:** Implementation of gradient-based classification (Laplacian) on 4x4 blocks, utilizing an 8x8 **overlapping window**, adhering to the VTM specification.
* **Geometric Transformation:** Calculation and application of geometric transformations (rotation, diagonal, and vertical mirroring) to align the texture direction with the filter coefficients.
* **Luma Filtering:** Application of the Wiener filter with fixed coefficients based on the determined class.
* **10-bit Support:** Uses `unsigned short` data types to correctly process HDR or high bit-depth video (values 0-1023), preventing *clipping* artifacts and integer overflow.
* **Power Monitoring:** Integration with the NVML library to measure GPU power consumption during execution.

## 🛠️ Prerequisites

* NVIDIA GPU (CUDA-compatible architecture).
* CUDA Toolkit installed (`nvcc`).
* NVIDIA Management Library (NVML) for power monitoring.

## ⚙️ Configuration (Resolution and Bit Depth)

To change the image resolution or bit depth, you must edit the constants at the beginning of the `main` function and the global definitions in the `.cu` file:

1.  **Image Resolution:**
    Go to the `main()` function (approximately line 430) and modify:
    ```c
    const int width = 1920;  // Image width
    const int height = 1080; // Image height
    ```

2.  **Bit Depth:**
    At the top of the file, under definitions:
    ```c
    #define BIT_DEPTH 10 // Change to 8 or 12 if necessary
    ```

## 📦 Compilation

To compile the code, use `nvcc`. You must link the `nvidia-ml` (for power monitoring) and `pthread` libraries.

```bash
nvcc main.cu -o alf_filter -lnvidia-ml -lpthread
