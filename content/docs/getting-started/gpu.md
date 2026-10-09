# Run with an NVIDIA GPU

Use the JAX CUDA installation path on Linux to run the framework's JAX planners
on an available NVIDIA GPU. The task configuration and experiment interface stay
the same: select `--device gpu` for configured runs, or `device="gpu"` in Python.

The current release provides a CUDA installation path; qualification varies by
task. These examples show how to configure and inspect a GPU run. They are not
GPU benchmark results or a claim of numerical parity with the CPU release suite.

For a container setup, follow the [Docker guide](/docs/docker/).

## Install the CUDA environment

Use Python 3.10–3.12 on a Linux host with a compatible NVIDIA driver. The
repository's GPU requirement file selects `jax[cuda12]>=0.6.2,<0.7` and also
includes the simulation dependencies.

```bash
git clone https://github.com/hhhhzl/genedynamics.git
cd genedynamics
python -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install -e ".[optimization]"
python -m pip install -r requirements/gpu-jax.txt
python -m pip check
```

If you already have a checkout, begin with its environment setup instead of
cloning again. Keep the repository's JAX version range when installing CUDA
support. Consult the [official JAX installation guide](https://docs.jax.dev/en/latest/installation.html)
for driver and CUDA compatibility; the latest upstream installation command may
target a newer JAX version than this release.

The V1 CUDA path is not available on macOS. Use the CPU example locally or run
these commands on a suitable Linux/NVIDIA machine.

## Confirm the actual device

First check that JAX can initialize a GPU, then verify that the framework places
an array there:

```python
import jax
from genedynamics.core.backends.runtime import RuntimeBackendManager

print("JAX version:", jax.__version__)
print("GPU devices:", jax.devices("gpu"))

RuntimeBackendManager.set_backend("jax", device="gpu")
backend = RuntimeBackendManager.get_backend()
assert backend.get_device() == "gpu", "The runtime fell back to CPU"

probe = backend.tensor([0.0, 1.0])
probe.block_until_ready()
print("Probe placement:", probe.devices())
assert all(device.platform == "gpu" for device in probe.devices())
```

Run this in the same Python environment as your experiment. If GPU initialization
fails, resolve the installation or driver problem before recording GPU results.
The framework can fall back to CPU with a warning, so successful process exit
alone does not establish that a GPU was used.

Use `gpu` for the device setting in both configured runs and the direct Python API.

## Run the included GPU examples

The current examples accept a device and candidate count. They check the selected
device and fail clearly if a GPU is unavailable:

```bash
python examples/plan_to_goal.py --device gpu --samples 1024
python examples/batched_constraints.py --device gpu --samples 4096
```

The first example runs the 2GO plan–execute loop. The second projects an action
batch against fixed halfspaces; it demonstrates array composition rather than a
complete robot task. Use [Docker](/docs/docker/) to run these entrypoints in the
GPU container.

## Run a planar experiment on GPU

Start with a single seed and obstacle level. This MBD task does not require a
robot simulator or learned checkpoint.

```bash
genedynamics-run configs/single_2d/mbd.yaml \
  --device gpu --seed 0 --level 0 --dry-run

genedynamics-run configs/single_2d/mbd.yaml \
  --device gpu --seed 0 --level 0 \
  --development-root results/_development/gpu-planar
```

`--dry-run` checks the configuration; it does not initialize a planner or verify
GPU execution. During the actual run, inspect the device message as well as the
completed/failed summary. The runner prints the resolved output directory and
saves configuration, metrics, trajectories, and provenance there.

## Run the 2GO corridor planner on GPU

The Zone A configuration uses 2GO for constrained humanoid body-motion planning.
The selected level below is the configuration's default level, with one seed.

```bash
genedynamics-run \
  configs/humanoid/corridor_2d/main/twogo_zone_a.yaml \
  --device gpu --seed 0 --level 1 --dry-run

genedynamics-run \
  configs/humanoid/corridor_2d/main/twogo_zone_a.yaml \
  --device gpu --seed 0 --level 1 \
  --development-root results/_development/gpu-corridor
```

This runs the planning configuration. It does not launch a humanoid walking
controller or establish a hardware deployment setup. Inspect the selected
trajectory, corridor metrics, and run status before expanding the comparison.
See the [corridor recipe](/docs/humanoid-corridor/) for task context.

After inspecting one completed case, expand the same output tree:

```bash
genedynamics-run \
  configs/humanoid/corridor_2d/main/twogo_zone_a.yaml \
  --device gpu --seeds 0 1 2 --level 1 \
  --development-root results/_development/gpu-corridor --resume
```

The runner reuses complete entries with matching resolved configurations.
Preserve aborted execution evidence and use a new output root for a new attempt.

## Use 2GO directly from Python

The [Python walkthrough](/docs/python-api/) constructs a task and planner before
entering a receding-horizon loop. For GPU use, select the device before creating
the planner. This compact example generates one plan:

```python
import jax
import numpy as np

from genedynamics.core.backends.runtime import RuntimeBackendManager
from genedynamics.envs import make_env, make_energy
from genedynamics.solvers import TwoGOSolver

assert jax.devices("gpu"), "No JAX GPU is available"
RuntimeBackendManager.set_backend("jax", device="gpu")
backend = RuntimeBackendManager.get_backend()
assert backend.get_device() == "gpu"

device = jax.devices("gpu")[0]
print("Selected JAX device:", device)
with jax.default_device(device):
    env_name = "single_integrator_box_2d"
    env = make_env(env_name)
    planner = TwoGOSolver(
        dynamics=env,
        energy=make_energy(env_name),
        backend=backend,
        dt=env.dt,
        horizon=20,
        Nsample=128,
        Ndiffuse=16,
    )
    state = backend.tensor([0.8, 0.8], dtype=np.float32)
    print("Input placement:", state.devices())
    plan = planner.solve(state, horizon=20, rng_key=jax.random.PRNGKey(0))
    assert np.isfinite(np.asarray(plan.states)).all()
    assert np.isfinite(np.asarray(plan.actions)).all()
    next_state = env.transition(state, plan.actions[0])

# The public trajectory interface returns host-side NumPy data.
print("First action:", plan.actions[0])
```

The JAX device context covers task construction and planning, including arrays
created inside compiled kernels. This example selects one visible GPU; it does
not distribute candidates across multiple GPUs.

To execute a full task, repeat the plan–execute loop from the walkthrough with a
fresh deterministic random key at each step. The current checked-in
`examples/plan_to_goal.py` supports `--device gpu --samples 1024` and includes
the complete loop. Use the configurable runner above to save experiment artifacts.

## Support boundaries

| Capability | Current status |
| --- | --- |
| Core JAX CPU | Supported portable path; CPU release evidence is documented. |
| JAX CUDA on Linux/NVIDIA | Installable through `requirements/gpu-jax.txt`; task qualification varies. |
| 2GO GPU configuration | Uses the JAX runtime and device override shown above; these commands do not imply a published GPU parity or speed benchmark. |
| MGA on GPU | Qualification is deferred. Use its qualified CPU path for the release recipes. |
| DPCC and SafeDiffuser | Specialized optional Torch integrations. Their CPU/CUDA behavior depends on the Torch installation, task assets, and checkpoints; JAX CUDA installation does not configure them. |
| MuJoCo/MJX and Brax | Simulation dependencies and adapters are separate from solver selection. A planner's GPU flag does not automatically move every simulator or controller to GPU. |
| Humanoid walking/push and Isaac Lab | Walking/push readiness remains deferred; Isaac Lab is an experimental deployment adapter. |
| macOS CUDA | Not available in the V1 support matrix. |

## Interpret performance carefully

The first planning call includes JAX tracing and compilation. Record compilation
and cold latency separately from warm latency, and include synchronization and
host/device transfers in the stated timing policy. Log hardware, JAX/jaxlib
versions, commit, configuration, seeds, horizon, candidate count, and diffusion
steps. Small examples are useful integration checks, not evidence of a GPU
speedup. See [Metrics and benchmarks](/docs/metrics/) for the reporting contract.

For a CPU comparison, run the same task with `--device cpu` and a separate
development output root. Different platforms can select different valid modes
when floating-point reductions break a symmetric tie; judge both task metrics
and constraint behavior rather than requiring identical trajectory coordinates.

## Sources

- [GPU dependency contract](https://github.com/hhhhzl/genedynamics/blob/main/requirements/gpu-jax.txt)
- [Compatibility matrix](https://github.com/hhhhzl/genedynamics/blob/main/docs/reference/compatibility.md)
- [Machine-readable support status](https://github.com/hhhhzl/genedynamics/blob/main/release/v1_compatibility.json)
- [CLI device override](https://github.com/hhhhzl/genedynamics/blob/main/genedynamics/experiments/runner.py)
- [JAX runtime device handling](https://github.com/hhhhzl/genedynamics/blob/main/genedynamics/core/backends/runtime/jax_backend.py)
- [2GO corridor configuration](https://github.com/hhhhzl/genedynamics/blob/main/configs/humanoid/corridor_2d/main/twogo_zone_a.yaml)
- [Direct Python example](https://github.com/hhhhzl/genedynamics/blob/main/examples/plan_to_goal.py)
