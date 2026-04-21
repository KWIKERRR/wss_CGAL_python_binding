# Weighted Straight Skeleton Wrapper

A Python/C++ wrapper around [CGAL](https://www.cgal.org/)'s weighted straight skeleton and skeleton extrusion (`CGAL::extrude_skeleton`) for generating 3D roof-like meshes from 2D polygon footprints with per-edge angle control.

---

## Overview

This library exposes C++ functions via a shared library (`libweighted_straight_skeleton.so`):

- **`compute_straight_skeleton_and_save`** — computes the weighted straight skeleton of a simple polygon and saves the resulting 3D mesh.

The Python wrapper (`weighted_straight_skeleton.py`) handles all ctypes bridging and exposes a single clean function: `compute_straight_skeleton`.

Output is saved as an `.off` mesh file (Object File Format), readable by most 3D tools.

---

## Requirements

| Dependency | Version |
|---|---|
| CGAL | 5.6.2 |
| Boost | 1.71.0 |
| GMP | 6.3.0 |
| MPFR | 4.2.1 |
| CMake | ≥ 3.10 |
| Python | ≥ 3.x |

> All dependencies are built locally under `dependencies/`. No system-wide installation is required.

---

## Building

### 1. Set up the environment variable

The build system relies on `WSS_PROJECT_ROOT` to locate dependencies the compiled shared library at runtime.

```bash
export WSS_PROJECT_ROOT=$(pwd)
```

### 2. Download and build dependencies

A convenience script is provided to download and compile all dependencies from source:

```bash
bash install.sh
```

This will download and build GMP, MPFR, Boost, CGAL into `dependencies/` and compiled the library. The compiled library will be placed in `build/libweighted_straight_skeleton.so` (Linux).

### 3. Set up the python environment

```bash
pip install -r requirements.txt
```
The core library only requires `numpy`. `plotly` is an optional dependency used for mesh visualization.

---

## Usage

### 1. WSS computation
```bash
python demos/1_base.py
```

```python
from weighted_straight_skeleton import compute_straight_skeleton
from utils import ensure_counterclockwise, load_off_file, visualize_mesh

# Footprint should be in the counterclockwise order
footprint = ensure_counterclockwise([(-10, 10), (-10, -10), (10, -10), (10, 10)])

# angles[0] is associated to edge footprint[0] -> footprint[1]
base_angles = [90, 45, 90, 45]

# The results is saved in a .off file
success = compute_straight_skeleton(footprint, 
                                    angles=base_angles, 
                                    output_file_path="roof.off")

vertices, faces = load_off_file(output_file_path)
visualize_mesh(vertices, faces)
```
![Output demos/1_base.py](./images/roof.png)

### 2. WSS computation with one hole

```bash
python demos/2_one_hole.py
```

```python
from weighted_straight_skeleton import compute_straight_skeleton
from utils import ensure_clockwise, ensure_counterclockwise, load_off_file, visualize_mesh

footprint = ensure_counterclockwise([(-10, 10), (-10, -10), (10, -10), (10, 10)])
base_angles = [90, 45, 90, 45]

# Hole points should be in the clockwise order
hole = ensure_clockwise([(-5, 5), (-5, -5), (5, -5), (5, 5)])
hole_angles = [45, 90, 90, 45]

# Base angles and hole angles should be concatenate in one list
roof_with_one_hole_angles = base_angles + hole_angles

compute_straight_skeleton(footprint, 
                          angles=roof_with_one_hole_angles, 
                          holes_list=[hole], 
                          output_file_path='roof_with_one_hole.off')

vertices, faces = load_off_file('roof_with_one_hole.off')
visualize_mesh(vertices, faces)
```
![Output demos/2_one_hole.py](./images/one_hole.png)

### 3. WSS computation with several holes

```bash
python demos/3_several_holes.py
```

```python
from weighted_straight_skeleton import compute_straight_skeleton
from utils import ensure_clockwise, ensure_counterclockwise, load_off_file, visualize_mesh

footprint = ensure_counterclockwise([(-10, 10), (-10, -10), (10, -10), (10, 10)])
base_angles = [90, 45, 90, 45]

hole1 = ensure_clockwise([(2, 8), (8, 8), (8, 2), (2, 2)])
hole1_angles = [90, 45, 90, 45]

hole2 = ensure_clockwise([(-2, -8), (-8, -8), (-8, -2), (-2, -2)])
hole2_angles = [45, 90, 90, 45]

hole3 = ensure_clockwise([(-2, 8), (-8, 8), (-8, 2), (-2, 2)])
hole3_angles = [45, 90, 45, 90]

hole4 = ensure_clockwise([(2, -8), (8, -8), (8, -2), (2, -2)])
hole4_angles = [90, 90, 45, 45]

holes = [hole1, hole2, hole3, hole4]
roof_with_holes_angle = base_angles + hole1_angles + hole2_angles + hole3_angles + hole4_angles


compute_straight_skeleton(footprint, 
                          angles=roof_with_holes_angle,
                          holes_list=holes, 
                          output_file_path='roof_with_holes.off')

vertices, faces = load_off_file('roof_with_holes.off)
visualize_mesh(vertices, faces)
```
![Output demos/3_several_holes.py](./images/several_holes.png)

### Parameters

| Parameter | Type | Description |
|---|---|---|
| `arr` | `list` / `np.ndarray` | Outer contour as `[[x, y], ...]`, counterclockwise |
| `angles` | `list` / `np.ndarray` | Roof slope angle per edge (degrees). Covers outer contour + all holes in order |
| `output_file_path` | `str` | Path to the output `.off` file |
| `holes_list` | `list` (optional) | List of hole contours `[[[x,y], ...], ...]` |
| `verbose` | `bool` (optional) | Print polygon and angle details from the C++ side |

---


## Project Structure

```
.
├── weighted_straight_skeleton.cpp   # C++ CGAL wrapper
├── weighted_straight_skeleton.py    # Python ctypes bindings
├── CMakeLists.txt                   # Build configuration
├── install.sh                       # Dependency download & build script
├── utils.py                         # Helper python functions
└── build/
    └── libweighted_straight_skeleton.so 
```

---
