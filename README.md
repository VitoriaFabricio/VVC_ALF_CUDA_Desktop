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
```

*Note: If the compiler cannot find `nvidia-ml`, ensure that GPU drivers are correctly installed or adjust the library path.*

## ▶️ How to Use

1.  **Prepare the Input File:**
    The code expects a file named **`original_0.csv`** in the same directory.
    * **Format:** Integer pixel values (Luma) separated by commas or spaces.
    * **Size:** Must contain exactly `width * height` values.

2.  **Run the Program:**
    ```bash
     ./alf_filter
    ```
    *(On some Linux systems, running with `sudo` might be necessary to access NVML power metrics).*

## 📂 Output Files

After execution, the program will generate three CSV files for analysis:

1.  **`imagem_final_filtrada.csv`**
    * Contains the pixel values of the image after applying the ALF filter. Values will be in the range [0, 1023] (for 10-bit).

2.  **`mapa_de_classes_final.csv`**
    * A matrix the size of the image where each pixel contains the Class ID (0 to 24) assigned to its 4x4 block. This visualizes how the algorithm classified the texture of that region.

3.  **`mapa_de_transformacao_final.csv`**
    * A matrix containing the Geometric Transformation ID (0 to 3) applied to each block.
    * `0`: No transformation.
    * `1`: Diagonal Mirroring.
    * `2`: Vertical Mirroring.
    * `3`: Rotation.

## 📝 Technical Details

* **Classification Kernel:** Uses a *Gather* strategy in Shared Memory to access the necessary neighbors for the 8x8 window without repeated Global Memory accesses.
* **Filtering Kernel:** Applies the 7x7 diamond-shaped filter using coordinates transformed by the `transform_coords_filter` function based on the classification result.
