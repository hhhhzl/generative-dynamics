# Docker

Build a local image, mount your checkout, and run planning or headless simulation without installing the framework into your host Python environment. Run every command below from the repository root.

| Target | Host | Use |
| --- | --- | --- |
| `dev-cpu` / `sim-cpu` | Native amd64 or arm64, including Apple Silicon | CPU planning and headless simulation with OSMesa |
| `train-gpu` | Linux amd64 with NVIDIA | CUDA JAX planning and optional Torch workloads, with EGL |

These are existing build targets and local image tags. No published container-registry image is required.

## CPU: build and run

```bash
docker compose -f docker/compose.cpu.yml build

docker compose -f docker/compose.cpu.yml run --rm genedynamics-dev-cpu \
  python examples/plan_to_goal.py --device cpu

docker compose -f docker/compose.cpu.yml run --rm genedynamics-dev-cpu \
  python examples/batched_constraints.py --device cpu
```

CPU Compose uses your host architecture. No custom environment file is needed. On Linux, set ownership before running Compose so generated files belong to your account:

```bash
export LOCAL_UID="$(id -u)" LOCAL_GID="$(id -g)"
```

Run MD-COAS with persistent experiment artifacts:

```bash
docker compose -f docker/compose.cpu.yml run --rm genedynamics-dev-cpu \
  genedynamics-run configs/single_2d/mdcoas.yaml \
  --device cpu --seed 0 --level 6 \
  --development-root results/_development/docker-mdcoas
```

Results appear under `results/_development/docker-mdcoas` on the host. Add `--dry-run` to validate the configuration first. Omit the final command to open an interactive shell.

## NVIDIA GPU: build, check, run

Use Linux x86-64 with an NVIDIA driver and the [NVIDIA Container Toolkit configured for Docker](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html). The Compose file's `gpus: all` setting requires [Compose 2.30.0 or later](https://docs.docker.com/reference/compose-file/services/#gpus). This is not an Apple GPU or Jetson image.

```bash
docker compose -f docker/compose.gpu.yml build

docker compose -f docker/compose.gpu.yml run --rm genedynamics-train-gpu \
  nvidia-smi

docker compose -f docker/compose.gpu.yml run --rm genedynamics-train-gpu \
  python -c "import jax; print(jax.devices('gpu'))"

docker compose -f docker/compose.gpu.yml run --rm genedynamics-train-gpu \
  python examples/plan_to_goal.py --device gpu --samples 1024

docker compose -f docker/compose.gpu.yml run --rm genedynamics-train-gpu \
  python examples/batched_constraints.py --device gpu --samples 4096
```

The service sets `JAX_PLATFORMS=cuda`, so unavailable CUDA fails during initialization rather than silently falling back to CPU. Use `--device gpu` for the framework's JAX paths. To restrict visibility, add `-e CUDA_VISIBLE_DEVICES=0` after `run --rm`.

A device check confirms access, not task success or numerical parity. MGA GPU qualification remains deferred; use its CPU release path.

## Direct Docker build and mount

Compose already mounts the repository at `/workspace`. For an explicit CPU build and interactive run:

```bash
docker build -f docker/Dockerfile \
  --target dev-cpu -t genedynamics/dev-cpu:local .

docker run --rm -it \
  --user "$(id -u):$(id -g)" \
  -v "$PWD:/workspace" -w /workspace \
  genedynamics/dev-cpu:local
```

For the GPU equivalent, build with `--platform linux/amd64 --target train-gpu`, use the tag `genedynamics/train-gpu:local`, and add `--gpus all` to `docker run`. Always specify the build target: the Dockerfile's final default stage is `train-gpu`.

Source edits are immediately visible through the mount, and output files survive container removal. Dependency changes require rebuilding the image. Tests and external simulator assets require the mounted checkout; examples, configurations, and packaged robot assets are included in the image.

## Headless simulation and assets

CPU rendering uses `MUJOCO_GL=osmesa`; GPU rendering uses `MUJOCO_GL=egl`. These defaults are already configured. The workflows save videos and figures offscreen and do not forward a desktop viewer.

Prepare the pinned Menagerie assets for recipes that need them:

```bash
docker compose -f docker/compose.cpu.yml run --rm genedynamics-dev-cpu \
  bash scripts/setup/setup_mujoco_menagerie.sh
```

They are read from `/workspace/third_party/mujoco_menagerie`. Third-party source assets are not baked into these images. D3IL still needs its integration setup, including Pinocchio and task data; GPU access alone does not provide them.

## Versions and scope

Both installers include the package's simulation, optimization, reports, and test extras, with JAX 0.6.2, Brax 0.14.1, and MuJoCo/MJX 3.6.0 pinned. CPU includes Torch CPU; GPU pins Torch 2.10.0 and torchvision 0.25.0 from the cu126 index. Both installers run `pip check`.

The CPU base defaults to Python 3.10. The GPU base defaults to CUDA 12.6.3 / Ubuntu 22.04. Available build arguments are `PYTHON_VERSION` for CPU, and `CUDA_VERSION` / `UBUNTU_VERSION` for GPU. Changing the CUDA base does not update the install script's Torch wheel index. Other dependencies follow package constraints, so these are development images rather than fully locked production environments.

The repository records CPU Docker validation; current CI does not build these images or run NVIDIA acceptance tests. The commands here were checked against the current source, not executed during this documentation review. The separate Isaac Lab Dockerfile remains experimental and outside V1 container qualification.


## Sources

- [Docker build targets](https://github.com/hhhhzl/genedynamics/blob/main/docker/Dockerfile)
- [CPU Compose service](https://github.com/hhhhzl/genedynamics/blob/main/docker/compose.cpu.yml)
- [GPU Compose service](https://github.com/hhhhzl/genedynamics/blob/main/docker/compose.gpu.yml)
- [Device and task support](/docs/compatibility/)
- [GPU workflows](/docs/gpu/)
