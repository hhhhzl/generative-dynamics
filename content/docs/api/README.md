# API surface

The V1 public surface is intentionally small:

- `genedynamics.experiments.framework.ExperimentConfig`
- `genedynamics.experiments.framework.ExperimentRunner`
- `genedynamics.experiments.framework.PluginRegistry`
- `MethodPlugin`, `EnvironmentPlugin`, `MetricsPlugin`,
  `VisualizationPlugin`, and `ObstacleGeneratorPlugin`
- core types and backend interfaces under `genedynamics.core`
- registered environment/robot factories under `genedynamics.envs` and
  `genedynamics.robots`
- deployment registries and configuration under `genedynamics.deploy`

Internal solver modules remain importable for research, but V1 compatibility is
defined at the plugin, configuration, and runner boundaries. Direct imports of
private helpers or simulator implementation classes may change before 1.0.

Use the [plugin guide](/docs/adding-a-plugin) for extension patterns and
the [compatibility matrix](/docs/compatibility) for backend coverage.


## Environment factories

```python
from genedynamics.envs import make_env, make_energy

env = make_env("single_integrator_box_2d")
energy = make_energy("single_integrator_box_2d")
```

`make_env(name: str, **kwargs)` creates a registered environment; an unknown
name raises `ValueError` and lists available registrations.
`make_energy(env_name: str)` creates the registered energy functional for that
environment name. See the [factory implementation](https://github.com/hhhhzl/genedynamics/blob/release/v1-open-source/genedynamics/envs/factories.py#L35).

## Runtime and a direct solver

The current source signatures are:

```text
RuntimeBackendManager.set_backend(name="jax", device="cpu", **kwargs)
RuntimeBackendManager.get_backend(name=None, device=None, **kwargs)

TwoGOSolver(dynamics, energy, backend, *, constraint_filter=None, **kwargs)
```

The first two are class methods. Construct the task and select the runtime
before constructing a solver. `TwoGOSolver` forwards common planning options
through `**kwargs`; the tested walkthrough sets `dt`, `horizon`, `Nsample`, and
`Ndiffuse`. See [Plan from Python](/docs/python-api) for the
complete loop and success criterion, not just a constructor.

The solver interface is `solve(x0, horizon, **kwargs) -> Trajectory`.
The 2GO walkthrough supplies an explicit `rng_key`. Solver-specific keyword
options are not universal across all planners.

`Trajectory` stores `states`, `actions`, and optional `info`. Its constructor
requires `len(states) == len(actions) + 1`; inconsistent lengths raise
`ValueError`.

Sources: [runtime manager](https://github.com/hhhhzl/genedynamics/blob/release/v1-open-source/genedynamics/core/backends/runtime/manager.py#L44),
[2GO constructor](https://github.com/hhhhzl/genedynamics/blob/release/v1-open-source/genedynamics/solvers/single/twogo/twogo.py#L35),
[solver contract](https://github.com/hhhhzl/genedynamics/blob/release/v1-open-source/genedynamics/core/solvers/base.py#L65),
[Trajectory](https://github.com/hhhhzl/genedynamics/blob/release/v1-open-source/genedynamics/core/types.py#L63).

## Configure and run an experiment

```python
from pathlib import Path
from genedynamics.experiments.framework import ExperimentConfig, ExperimentRunner
from genedynamics.experiments.runner import register_all_plugins

config = ExperimentConfig.from_yaml(Path("configs/single_2d/mbd.yaml"))
config.device = "cpu"
config.seeds = [0]
config.obstacle_levels = [0]
config.use_development_output_root(Path("results/_development/python-example"))

runner = ExperimentRunner(config)
register_all_plugins(runner)
results = runner.run_all(resume=True)
if runner.failures:
    raise RuntimeError(runner.failures)
```

This example is derived from the current runner interfaces. For formal research
matrices use the [CLI](/docs/cli), which also applies the configured
formal-readiness and checkpoint-file checks before execution.

| Public operation | Source behavior |
| --- | --- |
| `ExperimentConfig.from_yaml(path: Path)` | Load YAML, resolve inherited configuration, and return an `ExperimentConfig`. |
| `config.validate()` | Return a list of configuration errors. |
| `ExperimentRunner(config)` | Validate configuration and initialize an empty registry, result list, and failure list. |
| `runner.register_plugin(plugin, plugin_type, name=None)` | Register a component by type; defaults to the plugin's own name. |
| `runner.run_all(resume=False)` | Expand configured suites/levels/seeds and return experiment result dictionaries. |
| `runner.failures` | Retained failed entries from the current matrix run. |

Sources: [configuration](https://github.com/hhhhzl/genedynamics/blob/release/v1-open-source/genedynamics/experiments/framework/config.py#L129),
[runner construction](https://github.com/hhhhzl/genedynamics/blob/release/v1-open-source/genedynamics/experiments/framework/experiment.py#L124),
[matrix execution](https://github.com/hhhhzl/genedynamics/blob/release/v1-open-source/genedynamics/experiments/framework/experiment.py#L846),
[built-in registration](https://github.com/hhhhzl/genedynamics/blob/release/v1-open-source/genedynamics/experiments/runner.py#L330).

## Registry and plugin contracts

`PluginRegistry` exposes `register(plugin, plugin_type, name=None)`,
`get_plugin(plugin_type, name)`, `list_plugins(plugin_type=None)`, and
`has_plugin(plugin_type, name)`. Supported types are `method`, `environment`,
`metric`, `visualization`, and `obstacle_generator`.

`list_plugins` returns a name list when given a type; with no type it returns
a mapping from type to names. This documents the actual implementation behavior
(the current source annotation is narrower).

A `MethodPlugin` implements `name`,
`create_planner(env, energy, config)`, and
`plan(planner, initial_state, rng)`. Planning returns a dictionary containing
states, actions, and initial state; additional diagnostics depend on the solver.
Environment plugins own environment creation, energy creation, state dimension,
and position extraction, with optional task-owned reset and execution behavior.

See [Add a plugin](/docs/adding-a-plugin), the
[registry source](https://github.com/hhhhzl/genedynamics/blob/release/v1-open-source/genedynamics/experiments/framework/registry.py#L12),
and [abstract contracts](https://github.com/hhhhzl/genedynamics/blob/release/v1-open-source/genedynamics/experiments/framework/base.py#L19).
