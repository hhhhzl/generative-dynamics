# MuJoCo Menagerie

GenerativeDynamics uses MuJoCo Menagerie robot descriptions and meshes for
selected simulation, execution, and rendering paths, including Unitree Go2 and
G1. The integration supplies robot assets; the framework supplies the task,
planner, controller, and evaluation interfaces. Adding the asset catalogue does
not make every Menagerie robot a supported framework recipe.

## Initialize the pinned assets

From the GenerativeDynamics repository root:

```bash
bash scripts/setup/setup_mujoco_menagerie.sh
```

For source-based simulation outside Docker, install the simulation dependencies
in your Python environment as well:

```bash
python -m pip install -e ".[simulation]"
```

The setup script initializes `third_party/mujoco_menagerie` when needed and
applies the project's Go2 MJX adaptation. The submodule points to the official
upstream repository at commit:

```text
a03e87bf13502b0b48ebbf2808928fd96ebf9cf3
```

Use the setup script instead of replacing the checkout with the latest upstream
revision. It checks the revision and patch state before changing files.

## What the Go2 patch changes

`third_party/patches/mujoco_menagerie-go2-mjx.patch` changes 13 collision
geometries in `unitree_go2/go2_mjx.xml` to explicitly use spheres with one
radius. It preserves the earlier local Go2 MJX adaptation; it does not modify
robot meshes, inertial/dynamics parameters, or the G1 model.

The setup command is repeatable. An already applied patch is retained; a
conflicting patch or unexpected revision causes the script to stop without
resetting or cleaning the checkout. On a fresh setup, Git may report the Go2
XML as a local submodule modification while `HEAD` remains at the official pin.
The former local commit `e5146679f3cfcb327cf759fc9706f5bb0236bd5e` is also
recognized and preserved, but is not the revision to fetch from upstream.

## Paths and an existing checkout

The standard checkout lives at `third_party/mujoco_menagerie`. Relevant robot
paths are:

| Asset | Path below the Menagerie root |
| --- | --- |
| Go2 robot | `unitree_go2/go2.xml` |
| Go2 MJX variant patched by the framework | `unitree_go2/go2_mjx.xml` |
| G1 robot candidates | `unitree_g1/g1.xml`, `unitree_g1/g1_mjx.xml` |
| G1 scene candidates | `unitree_g1/scene.xml`, `unitree_g1/scene_mjx.xml` |

Resolvers choose available candidates according to the requested model/scene and
MJX preference. Keep each robot's mesh and scene directories together.

To use an independently initialized checkout at the expected revision:

```bash
bash scripts/setup/setup_mujoco_menagerie.sh /absolute/path/to/mujoco_menagerie
export MUJOCO_MENAGERIE_PATH=/absolute/path/to/mujoco_menagerie
```

`MUJOCO_MENAGERIE_PATH` points to the catalogue root, not the `unitree_go2` or
`unitree_g1` subdirectory. Passing a directory to the setup script checks and
patches it; it does not configure the runtime environment variable for you.

Inspect the paths the framework actually resolves:

```python
from genedynamics.robots import get_robot_registry

registry = get_robot_registry()
print("Go2:", registry.get_model_path("quadruped", "go2"))
print("G1:", registry.get_model_path("humanoid", "g1"))
```

G1 resolution prefers an explicit override, then the environment variable,
an installed Menagerie package, and the project tree. The current Go2 registry
checks an installed Menagerie package before the environment variable. If a
different installation is being selected, use the chosen component's explicit
model-path option and inspect the resolved path. A found file establishes asset
location, not successful simulation or task performance.

## CPU, GPU, and rendering

Asset setup is independent of compute-device selection. MuJoCo and MJX consume
robot model files; the planner, physics adapter, and renderer determine where
computation runs. Selecting `--device gpu` for a planner does not automatically
move its controller or renderer onto the GPU.

MJX paths prefer compatible model/scene variants where available. The Go2 patch
addresses that model's collision geometry, not general CUDA installation. Follow
[GPU workflows](/docs/gpu/) and [compatibility](/docs/compatibility/) for device
setup and per-task support.

## Use the assets with Docker

See [Docker setup](/docs/docker/) for building and selecting the CPU or GPU
service.

The supplied Compose services mount the repository at `/workspace`. Both images
set `MUJOCO_MENAGERIE_PATH=/workspace/third_party/mujoco_menagerie`, so the
initialized host checkout is visible at the expected container path.

Initialize through the mounted checkout after building the CPU image:

```bash
docker compose -f docker/compose.cpu.yml build
docker compose -f docker/compose.cpu.yml run --rm genedynamics-dev-cpu \
  bash scripts/setup/setup_mujoco_menagerie.sh
```

The change persists in the host checkout. The GPU service uses the same asset
mount. CPU Compose selects OSMesa rendering (`MUJOCO_GL=osmesa`); GPU Compose
selects EGL (`MUJOCO_GL=egl`). These are headless artifact-rendering paths.

The Docker build copies packaged framework robot assets, but does not copy the
whole Menagerie submodule into the image. Likewise, the optional submodule is
not included in the Python wheel. Keep the checkout mounted for recipes that
need it.

For an external checkout, initialize and patch it on the host, then mount it
read-only and set its container-visible path:

```bash
docker compose -f docker/compose.cpu.yml run --rm \
  -v /absolute/path/to/mujoco_menagerie:/opt/menagerie:ro \
  -e MUJOCO_MENAGERIE_PATH=/opt/menagerie \
  genedynamics-dev-cpu \
  python -c "from genedynamics.robots import get_robot_registry; print(get_robot_registry().get_model_path('humanoid', 'g1'))"
```

Do not run the patching setup command against that read-only mount.

## Sources and model licenses

The framework's MIT license does not replace the individual model licenses.
Retain the corresponding model notices when redistributing descriptions or
meshes. The repository records the integration in:

- [Submodule definition](https://github.com/hhhhzl/genedynamics/blob/main/.gitmodules)
- [Asset setup script](https://github.com/hhhhzl/genedynamics/blob/main/scripts/setup/setup_mujoco_menagerie.sh)
- [Pinned revision and Go2 patch notes](https://github.com/hhhhzl/genedynamics/blob/main/third_party/patches/README.md)
- [Robot registry](https://github.com/hhhhzl/genedynamics/blob/main/genedynamics/robots/registry.py)
- [G1 asset resolver](https://github.com/hhhhzl/genedynamics/blob/main/genedynamics/robots/g1/assets.py)
- [Docker setup](https://github.com/hhhhzl/genedynamics/blob/main/docker/README.md)
- [Third-party notices](https://github.com/hhhhzl/genedynamics/blob/main/THIRD_PARTY_NOTICES.md)
