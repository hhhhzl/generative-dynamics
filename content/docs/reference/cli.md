# Command-line reference

`genedynamics-run` executes an experiment configuration. Run it from the
repository root when using checked-in configurations and project-relative assets.

```bash
genedynamics-run CONFIG [OPTIONS]
```

The positional `CONFIG` names a YAML file; the implementation also accepts JSON.
It resolves relative paths from the working directory, with a repository-root
fallback when the path does not exist there.

| Argument | Meaning |
| --- | --- |
| `CONFIG` | Configuration file path. |
| `--level N` | Select one integer obstacle level. |
| `--seed N` | Select one seed. Mutually exclusive with `--seeds`. |
| `--seeds N ...` | Select several seeds; duplicates are removed. |
| `--suite NAME` | Select one named entry from `config.suites`. |
| `--suites NAME ...` | Select several named suites. Mutually exclusive with `--suite`. Unknown names are rejected. |
| `--development-root PATH` | Mirror the canonical project-relative output under this root and mark the run as development. |
| `--device cpu\|gpu\|cuda` | Override the configured JAX device. Availability and qualification remain task-specific. |
| `--resume` | Reuse complete results with matching resolved configuration; execute missing or mismatched entries. |
| `--dry-run` | Resolve and validate configuration without allocating an experiment run. |
| `-h`, `--help` | Show the parser's help text. |

## Validate first

```bash
genedynamics-run configs/single_2d/mbd.yaml \
  --device cpu --seed 0 --level 0 --dry-run
```

A successful dry run prints the environment, method, levels or suites, seeds,
resolved output directory, and run class. It does not execute the planner or
prove that every optional dependency and checkpoint required at runtime exists.

## Run a development matrix

```bash
genedynamics-run configs/single_2d/mbd.yaml \
  --device cpu --seeds 0 1 2 --level 0 \
  --development-root results/_development/planar-example --resume
```

The CLI exits unsuccessfully when matrix entries fail and prints their errors.
The runner retains failure evidence; an aborted execution attempt requires a
new output root rather than overwriting its result.

Formal task configurations may require a frozen protocol and learned assets.
Use the documented recipe and [learning guide](/docs/learning-and-priors)
for those dependencies.

## Python module and deployment entry points

The equivalent module invocation is:

```bash
python -m genedynamics.experiments.runner CONFIG [OPTIONS]
```

The package separately exposes `genedynamics-deploy`. Its runtime is composed
from robot I/O, controllers, tasks, safety filters, and observers. Check its
own help and the chosen deployment preset instead of applying experiment-runner
flags to it:

```bash
genedynamics-deploy --help
```

## Sources

- [Actual argument definitions](https://github.com/hhhhzl/genedynamics/blob/release/v1-open-source/genedynamics/experiments/runner.py#L114)
- [Validation, formal readiness, and execution](https://github.com/hhhhzl/genedynamics/blob/release/v1-open-source/genedynamics/experiments/runner.py#L250)
- [Installed entry points](https://github.com/hhhhzl/genedynamics/blob/release/v1-open-source/pyproject.toml#L127)
