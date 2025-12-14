# Advanced Computational Photography & Image Blending

![Python](https://img.shields.io/badge/python-3.8%2B-blue?logo=python&logoColor=white)
![NumPy](https://img.shields.io/badge/numpy-1.24%2B-013243?logo=numpy&logoColor=white)
![SciPy](https://img.shields.io/badge/SciPy-Sparse_Solvers-8C86C0?logo=scipy&logoColor=white)
![OpenCV](https://img.shields.io/badge/OpenCV-Image_Processing-red?logo=opencv&logoColor=white)

## Project Overview

This repository contains the solution for **Series 5 of the Principles of Image Processing** course. It addresses three complex computer vision challenges focusing on image manipulation and blending. The implementations are derived from specific mathematical models required by the assignment:

1.  **Face Morphing (Question 1):** Geometric transformation and color interpolation to morph one face into another over a time sequence.
2.  **Poisson Blending (Question 2):** Gradient-domain image editing to seamlessly clone an object into a target image by solving Poisson partial differential equations.
3.  **Multiresolution Blending (Question 3):** Frequency-domain blending using Laplacian stacks to merge images with different "feathering" intensities across frequency bands.

## Features & Mathematical Basis

### 1. Face Morphing (`q1.py`)

[Project screenshot](./results/result1.png)

* **Goal:** Generate a 45-frame video sequence morphing Face A to Face B.
* **Algorithm:**
    * **Correspondence:** Uses 81 manual landmarks (eyes, nose, mouth) to define facial structure.
    * **Triangulation:** Applies Delaunay Triangulation (`scipy.spatial.Delaunay`) to create a consistent mesh across both images.
    * **Affine Warping:** Calculates a unique Affine Transformation matrix for every triangle in every frame to map pixels from the source/target to the intermediate shape.
    * **Cross-Dissolve:** Linearly interpolates pixel intensity (color) simultaneously with the geometric warp.

### 2. Poisson Blending (`q2.py`)
* **Goal:** Seamlessly paste a source region into a target background such that the edges are invisible.
* **Algorithm:**
    * **Gradient Preservation:** Instead of copying pixel values directly, we copy the *gradients* (changes in intensity) of the source image.
    * **Laplacian Operator:** The problem is formulated as finding a function whose Laplacian matches the source's Laplacian, subject to boundary constraints defined by the target image.
    * **Sparse Solver:** Constructs a massive sparse matrix $A$ (representing the neighbor relationships/Laplacian operator) and solves the linear system $Ax = b$ using `scipy.sparse.linalg.spsolve`.

### 3. Multiresolution Blending (`q3.py`)
* **Goal:** Blend two images without "ghosting" artifacts by treating high and low frequencies differently.
* **Algorithm:**
    * **Laplacian Stack:** Decomposes images into a "stack" of frequency bands (similar to a pyramid but without downsampling).
    * **Variable Feathering:** Applies a wide Gaussian blur to the mask for low-frequency layers (strong blending) and a narrow blur for high-frequency layers (preserving details).
    * **Reconstruction:** Sums the blended layers to produce the final `res10.jpg`.

## Installation

1.  **Clone the repository:**
    ```bash
    git clone [https://github.com/nikelroid/image-processing-5.git](https://github.com/nikelroid/image-processing-5.git)
    cd image-processing-5
    ```

2.  **Install Dependencies:**
    The project relies heavily on `scipy.sparse` for the linear algebra solver and `dlib` for landmark detection.
    ```bash
    pip install numpy opencv-python scipy dlib imutils
    ```

## Code Description & Usage

### Question 1: Face Morphing

* **`Results/q1-ext/landmarker.py`**:
    * *Purpose:* A helper tool to generate `.dat` files containing (x,y) coordinates of facial features.
    * *Logic:* It uses `dlib`'s pre-trained face detector to find initial points, then allows the user to manually refine them to ensure perfect correspondence between the two faces.
    * *Usage:* Run this to generate `landmarks1.dat` and `landmarks2.dat`.

* **`Results/q1.py`**:
    * *Purpose:* The main morphing engine.
    * *Process:*
        1.  Loads images `res01.jpg` and `res02.jpg`.
        2.  Reads landmarks and computes Delaunay triangles.
        3.  Iterates 45 times (for 45 frames).
        4.  For each frame, calculates the intermediate shape (weighted average of points) and warps both images to this shape using Affine transforms.
        5.  Blends the warped images based on the frame progress (`fr` variable).
    * *Output:* Produces `morph.mp4` (video) and specific frames `res03.jpg` and `res04.jpg` as required by the assignment.
    * *Run:* `python Results/q1.py`

### Question 2: Poisson Blending

* **`Results/q2-ext/Segment.py`**:
    * *Purpose:* A semi-automatic tool to create a binary mask for the object to be cloned.
    * *Logic:* Implements the **GraphCut** algorithm via `cv2.grabCut`. The user draws a rectangle, and the algorithm separates foreground from background.
    * *Usage:* Run to generate `mask.jpg`.

* **`Results/q2.py`**:
    * *Purpose:* Solves the Poisson equation for seamless cloning.
    * *Process:*
        1.  Loads Source (`res05.jpg`), Target (`res06.jpg`), and Mask.
        2.  Constructs the Laplacian matrix (matrix with 4 on diagonals, -1 on neighbors) which represents the derivative of the image.
        3.  Constructs the divergence vector $b$ from the source image gradients.
        4.  Solves the system for R, G, and B channels separately to reconstruct the pixel values.
    * *Output:* `res07.jpg`.
    * *Run:* `python Results/q2.py`

### Question 3: Multiresolution Blending

* **`Results/q3.py`**:
    * *Purpose:* Performs blending using Laplacian Stacks.
    * *Process:*
        1.  Loads two images (`res08.jpg`, `res09.jpg`) and a split mask.
        2.  Creates a Gaussian stack for the mask, increasing the blur radius (`sigma`) at each level.
        3.  Creates Laplacian stacks for both images (Image - Gaussian(Image)).
        4.  Blends each level independently: $L_{blend} = L_A \cdot M + L_B \cdot (1-M)$.
        5.  Collapses the stack to form the final image.
    * *Output:* `res10.jpg`.
    * *Run:* `python Results/q3.py`

## Contributing

1.  Fork the project.
2.  Create your feature branch (`git checkout -b feature/NewAlgorithm`).
3.  Commit your changes (`git commit -m 'Add new blending mode'`).
4.  Push to the branch (`git push origin feature/NewAlgorithm`).
5.  Open a Pull Request.

## License

This project is open-source. Please attribute the original author when using or modifying this code.
