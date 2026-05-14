# skeleton_keys_polygon

Tools for creating the polygon files needed for the [skeleton_keys](https://github.com/AllenInstitute/skeleton_keys) package. These polygon files define cortical layer boundaries relative to individual neuron soma positions, enabling layer-aligned morphological analysis.

## Overview

This project generates polygon dictionaries that describe cortical layer boundaries around a given neuron's soma location. It accounts for the curvature of cortical layers by using local "up vectors" that vary across the XZ plane of the dataset (MICrONS minnie65).

The pipeline:
1. Determines the local up vector for a soma's XZ position
2. Intersects layer boundary meshes with a plane through the soma (oriented by the up vector)
3. Builds polygon paths for each layer (Layer 1 through Layer 6b)
4. Outputs a polygon dictionary consumable by skeleton_keys

## Project Structure

```
.
├── polygon_creation.py        # Main module: mesh loading, up vector lookup, polygon generation
├── tools.py                   # Utilities: mesh vertex extension and face triangulation
├── cut_polygons_1.ipynb       # Development notebook for prototyping the polygon workflow
├── environment.yml            # Conda environment specification for reproducible setup
├── up_vecs_ranges.json        # Pre-computed up vectors and XZ bin ranges for minnie65
├── layer_meshes/              # Precomputed Neuroglancer meshes for cortical layer boundaries
│   ├── l23, l4, l5, l6a, l6b, wm
├── extended_layer_meshes/     # Layer meshes with vertices extended to cover full dataset XZ extent
└── README.md
```

## Setup

Create and activate the conda environment:

```bash
conda env create -f environment.yml
conda activate skeleton_keys_polygon
```

This installs all required dependencies (Python 3.11, trimesh, meshparty, neuroglancer-scripts, nglui, caveclient, NumPy, SciPy, pandas, matplotlib, and more-itertools).

## Usage

```python
from polygon_creation import make_poly_file, mesh_dict

# soma position in nm (x, y, z)
soma_pos = [800000, 200000, 900000]
specimen_id = 12345

# Generate the polygon dictionary
poly_dict = make_poly_file(mesh_dict, soma_pos, specimen_id)
```

The returned `poly_dict` contains:
- `layer_polygons` — list of layer boundary polygons (Layer1 through Layer6b)
- `pia_path` — pia surface mesh line
- `wm_path` — white matter surface mesh line
- `soma_path` — circular polygon around the soma

## Key Functions

### `polygon_creation.py`

| Function | Description |
|----------|-------------|
| `load_meshes(mesh_folder_path)` | Loads layer boundary meshes from a directory of precomputed mesh files |
| `find_up_vec(x_pos, z_pos)` | Returns the local up vector for a given XZ position |
| `make_poly_file(mesh_dict, soma_pos, specimen_id)` | Main entry point — builds the full polygon dictionary |
| `get_mesh_line(mesh, soma_pos, up_vec)` | Intersects a mesh with a plane and returns ordered vertices |
| `build_layer_dict(...)` | Constructs a single layer polygon dictionary |
| `calculate_soma_poly(soma_loc, n, radius)` | Generates a circular polygon around the soma |

### `tools.py`

| Function | Description |
|----------|-------------|
| `extend_vertices(vertices, ...)` | Extends grid-like mesh vertices to cover the full dataset XZ extent |
| `create_faces(vertices)` | Triangulates extended grid vertices into mesh faces |
| `triangulate_grid_rows(row_1, row_2)` | Creates triangle faces between two rows of equal-length indices |
| `verts_to_df(vertices)` | Converts vertex array to a sorted DataFrame |
