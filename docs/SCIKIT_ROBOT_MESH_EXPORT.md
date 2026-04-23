# DAE export with scikit-robot

The `DAE via scikit-robot` mesh format exports SolidWorks meshes as an
intermediate `.3dxml` package, then calls scikit-robot to convert the URDF and
meshes to local-origin `.dae` visual meshes and `.stl` collision meshes.

This matches scikit-robot's `--force-zero-origin` workflow: the visual/collision
origin is baked into the mesh vertices, and the generated URDF origins are set
back to zero.

## Install Python

Install Python 3.10 or 3.11 from:

https://www.python.org/downloads/windows/

During installation, enable `Add python.exe to PATH`.

Avoid Python 3.14 for now. Some geometry dependencies used by scikit-robot do
not yet provide Windows wheels for Python 3.14, so pip may try to compile them
from source.

Check the installation from PowerShell:

```powershell
python --version
```

## Create an environment

Using a virtual environment is recommended:

```powershell
py -3 -m venv C:\sw2urdf-skrobot
C:\sw2urdf-skrobot\Scripts\python.exe -m pip install --upgrade pip
```

Install scikit-robot and only the mesh conversion dependencies needed by this
exporter:

```powershell
C:\sw2urdf-skrobot\Scripts\python.exe -m pip install --upgrade numpy scipy networkx pillow six ordered-set packaging lxml trimesh pycollada
C:\sw2urdf-skrobot\Scripts\python.exe -m pip install --upgrade scikit-robot --no-deps
```

Optional, for GLB or advanced mesh workflows:

```powershell
C:\sw2urdf-skrobot\Scripts\python.exe -m pip install --upgrade pygltflib
```

## Point the exporter to Python

If `python` on PATH already points to the environment above, no extra setup is
needed.

Otherwise set `SW2URDF_PYTHON` to the full Python executable path:

```powershell
setx SW2URDF_PYTHON "C:\sw2urdf-skrobot\Scripts\python.exe"
```

Restart SolidWorks after setting the environment variable.

## Verify scikit-robot

Run:

```powershell
C:\sw2urdf-skrobot\Scripts\python.exe -m skrobot.apps.convert_urdf_mesh --help
```

If this prints command help, the exporter should be able to call scikit-robot.

## Use from SolidWorks

1. Open the Assembly Export form.
2. In `Mesh Format`, select `DAE via scikit-robot`.
3. Export normally.

The exporter writes:

- `urdf/<robot>.urdf`: final URDF with visual/collision origins baked into mesh files.
- `meshes/*.dae`: visual meshes.
- `meshes/*.stl`: collision meshes.
- `urdf/<robot>_3dxml_intermediate.urdf`: intermediate URDF used for conversion.
- `meshes/*.3dxml`: intermediate source meshes.

The intermediate files are kept so failures can be debugged and conversion can
be rerun manually if needed.

Manual rerun example:

```powershell
C:\sw2urdf-skrobot\Scripts\python.exe -m skrobot.apps.convert_urdf_mesh `
  "C:\path\to\package\urdf\robot_3dxml_intermediate.urdf" `
  --output "C:\path\to\package\urdf\robot.urdf" `
  --format dae `
  --collision-mesh-format stl `
  --force-zero-origin `
  --overwrite-mesh
```

## Troubleshooting

If export fails, check `export.log`.

Common fixes:

- `python` not found: set `SW2URDF_PYTHON` and restart SolidWorks.
- `No module named skrobot`: install scikit-robot into the Python environment used by `SW2URDF_PYTHON`.
- `Failed building wheel for pysdfgen`: use Python 3.10 or 3.11, then install
  scikit-robot with `--no-deps` after installing the minimal dependencies
  listed above. `pysdfgen` is not needed for this exporter path.
- DAE export missing colors: upgrade trimesh with `python -m pip install --upgrade trimesh`.
- Package paths cannot be resolved: keep the generated package directory layout unchanged while converting.
