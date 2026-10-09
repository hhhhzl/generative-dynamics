# Installation

Choose the [portable CPU setup](#supported-base-environment), a
[Linux/NVIDIA GPU environment](/docs/gpu/), or the framework's
[Docker images](/docs/docker/). GPU selection and simulator selection are
separate; check the requirements of your task.

## Supported base environment

- Python 3.10, 3.11, or 3.12
- Linux or macOS for the core package
- JAX CPU for the portable default

Clone the repository, then create an isolated environment. Repository
access is required while the open-source release is being prepared.
The optimization extra supports the CPU 2GO walkthrough:

```bash
git clone https://github.com/hhhhzl/genedynamics.git
cd genedynamics
python -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install -e ".[optimization]"
python -c "import genedynamics; print(genedynamics.__version__)"
```

For core-only integrations that do not use QP-backed constraint paths,
`python -m pip install -e .` is sufficient. Add extras for the capabilities
your task requires.

## Capability extras

| Extra | Adds | Use it for |
| --- | --- | --- |
| `optimization` | OSQP, CVXOPT, JAXopt, QPAX | QP-backed constraint paths and comparisons |
| `simulation` | MuJoCo/MJX, Brax, Gymnasium, PyBullet, trimesh | Robot simulation tasks |
| `manipulator` | Pinocchio, JAX kinematics and mesh proximity support | Serial-arm tools |
| `d3il` | D3IL Python/build dependencies | D3IL avoiding recipes |
| `torch` | Torch, Diffusers, QPTH | DPCC and SafeDiffuser integrations |
| `reports` | Jinja, pandas, seaborn | HTML and aggregate reports |
| `deploy` | FlatBuffers, WebSockets, OSQP | Runtime and AR transport paths |
| `docs` | MkDocs Material | Local documentation site |
| `dev` | Test, build, formatting, and release tools | Contributors |

Example:

```bash
python -m pip install -e ".[simulation,optimization,reports]"
```

## GPU JAX

GPU installation depends on the host driver and platform, so it is not a core
wheel dependency. On a compatible Linux/NVIDIA host:

```bash
python -m pip install -r requirements/gpu-jax.txt
python -c "import jax; print(jax.default_backend(), jax.devices())"
```

The first V1 release does not qualify MGA GPU behavior. A visible GPU device
only proves installation; it does not prove numerical parity or performance.
See [GPU workflows](/docs/gpu/) for device checks, Python use, and configured
MBD/2GO examples. See [Docker](/docs/docker/) to run the same tasks in a container.

## MuJoCo Menagerie assets

G1 and Go2 integrations can use the pinned Menagerie submodule. Initialize the
assets and preserve the project's Go2 MJX adaptation with:

```bash
bash scripts/setup/setup_mujoco_menagerie.sh
```

The setup is safe to repeat and refuses conflicting edits. See the
[MuJoCo Menagerie integration](/docs/mujoco-menagerie/) for the pinned revision,
Go2 patch, model paths, and Docker mounts.

## D3IL

D3IL is kept under `third_party/environments/d3il`. Clone with submodules and
run the integration setup:

```bash
bash scripts/setup/setup_mujoco_menagerie.sh
./scripts/setup/setup_d3il.sh
python -m pip install -e ".[d3il,torch]"
```

See [the D3IL integration guide](/docs/d3il) for dataset and
controller details.

## Verify a wheel

```bash
python -m pip install build twine
python -m build
python -m twine check dist/*
python -m pip install --force-reinstall dist/genedynamics-*.whl
python -c "import genedynamics; print(genedynamics.__version__)"
```

The release CI also inspects wheel contents to ensure required robot assets are
present and deferred V1 systems are absent.
