# Change Summary

This document summarizes the main features and bug fixes added on top of the upstream SolidWorks URDF exporter.

## New Features

- Added `DAE via scikit-robot` as a new assembly mesh export format.
- Added an automatic post-processing pipeline that exports intermediate `3dxml` meshes and converts them to `dae`/`stl` with `skrobot.apps.convert_urdf_mesh`. A user-facing setup and troubleshooting documentation for the Python/scikit-robot export pipeline in [SCIKIT_ROBOT_MESH_EXPORT.md](./SCIKIT_ROBOT_MESH_EXPORT.md).
- Added support for the `SW2URDF_PYTHON` environment variable so the exporter can use a specific Python interpreter.
- Added better support for empty links used as helper frames in robot structures.

## Bug Fixes

- Fixed a 3DXML export side effect where localizing a link for mesh export could corrupt URDF inertial data.
- Preserved correct 3DXML visual/collision compensation so meshes exported in assembly coordinates still appear in the right place in URDF.
- Prevented links with zero mass from emitting invalid or misleading `<inertial>` elements.

## Behavioral Notes

- Empty links which will be marked as fixed frames are exported as structural frames only. They do not generate mesh, also invalid `<visual>` & `<collision>` elements, or `<inertial>` data will not be generated since no geometry is present. Importantly, they **should not rely on automatic joint inference** like `automatically detected` or `automatically generated`.
- Links connected by `fixed joint` are still treated as normal rigid links unless they are Empty links.
- The new DAE workflow depends on an external Python environment. Standard STL and 3DXML export paths remain available without Python.

## End-User Guide

### Installation

- Install the exporter using the visual studio. You can create a installer by [Inno Setup](https://jrsoftware.org/isinfo.php) for easier deployment of the upgraded exporter.
- If you plan to use `DAE via scikit-robot`, install a supported Python environment and configure `SW2URDF_PYTHON` if needed.
- Restart SolidWorks after installation or after changing Python-related environment variables.

### Basic Export Workflow

1. Open the target assembly in SolidWorks.
2. Launch the SW2URDF exporter.
3. Define links and joints as usual.
4. Choose a mesh export format:
   - `STL` for the standard built-in workflow.
   - `3dxml` for direct 3DXML mesh export.
   - `DAE via scikit-robot` for zero-origin converted meshes.
5. Run the export and inspect the generated package in the selected output directory.

### When to Use Each Mesh Format

- Use `STL` when you want the simplest dependency-free export path.
- Use `3dxml` when you want to preserve the existing 3DXML-based workflow.
- Use `DAE via scikit-robot` when you need mesh geometry with origin baked into the mesh and cleaner URDF visual/collision origins.
