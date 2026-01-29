# Targetless LiDAR–Camera Calibration (Notebook)

This repository contains the Jupyter notebook **Targetless__Lidar_Camera_calibration.ipynb**.
It performs targetless LiDAR–camera calibration by:
- segmenting boats in the camera image using YOLOv8 instance segmentation,
- clustering LiDAR points,
- projecting LiDAR points into the image,
- scoring how well projected clusters align with the image masks, and optimizing the extrinsics.

The notebook is written for **Google Colab**, but it can also run locally (setup below).

---

## Data requirements 

### 1) Folder structure

`DATA_ROOT` must contain **exactly** these two subfolders (same names, including the space):

```
DATA_ROOT/
  image_2/        # camera images
  lidar pcd/      # LiDAR point clouds
```

### 2) Camera images

- Location: `DATA_ROOT/image_2/`
- Format: **PNG** files (`*.png`)
- All images should be from the same camera and have a consistent resolution.
- The scenes should contain **boats** (or the obejct class must be changed the code).

### 3) LiDAR point clouds

- Location: `DATA_ROOT/lidar pcd/`
- Format: **KITTI-style binary** files (`*.bin`)
- Data type: `float32`
- Shape: `N x 4` per file, in this order:

  `x, y, z`

So each `.bin` file must have a byte size that is a multiple of **16 bytes** (4 floats × 4 bytes).


### 4) Pairing between image and LiDAR

The notebook loads files using:

- `sorted(image_2/*.png)`
- `sorted(lidar pcd/*.bin)`

and then pairs them by index (`imgs[i]` with `bins[i]`).

To avoid mismatches:
- the filenames match by frame id (recommended), e.g. `000123.png` and `000123.bin`, or
- both folders have the **same number of files** and sorting produces the same order.

### 5) Coordinate and unit assumptions

- LiDAR points are expected in **meters**.
- The notebook uses a fixed “LiDAR → camera” axis mapping inside `make_A_lidar_to_cam0()`.
  The docstring describes the assumed LiDAR axes as:

  - LiDAR: **X left/right**, **-Y forward**, **Z up**
  - Camera: **X right**, **Y down**, **Z forward**

If a LiDAR uses a different convention, mapping must be adjusted in `make_A_lidar_to_cam0()` (and/or the `mirror_x` option), otherwise projections will be wrong.

### 6) Camera intrinsics (required for projection)

The notebook requires:
- camera matrix `K` (3×3),
- distortion coefficients `dist` (OpenCV format, currently uses a 14×1 vector).

These intrinsics must correspond to the **same image resolution** as `image_2/*.png`.


## Setup

### Option 1 — Google Colab (tested)

1. Upload the notebook to Colab.
2. Mount Google Drive (already present in the first cell).
3. Put dataset on Drive and set:

```python
DATA_ROOT = "/content/drive/MyDrive/<your_dataset_folder>"
```

4. Run the install cell, for example:

```python
%pip install -U ultralytics open3d plotly opencv-python scipy matplotlib pandas
```

### Option B — Local run

1. Create a virtual environment (recommended).
2. Install dependencies:

```bash
pip install -U ultralytics open3d plotly opencv-python scipy matplotlib pandas
```

3. Set `DATA_ROOT` to your local path (and remove/ignore `drive.mount(...)`):

```python
DATA_ROOT = "./data"
```

---
## Running the notebook

1. Open the notebook and run cells from top to bottom.
2. If you want to process fewer frames, adjust the `frames` list/range used in the notebook.
3. Replace `K` and `dist` with your camera intrinsics before evaluating projections.

## Notes

This notebook is a research-style pipeline and assumes a dataset where:
- boats are visible in the camera images,
- boats generate separable LiDAR clusters,
- camera intrinsics are known (or approximated),
- correct LiDAR–camera axis mapping is provided.

