# Read your results

A configured experiment saves both its outputs and the information needed to
identify the run. Start with the [first experiment](/docs/quickstart), then inspect
the output directory printed by the runner.

## Locate a run

`--development-root` mirrors the configured project-relative output below the
chosen root. Use the printed **Output directory** rather than assuming all files
are written directly at the root.

Within that resolved directory, a typical matrix contains:

```text
protocol_manifest.json
overall_summary.json
level_<level-or-suite>/
  summary.json
  seed_<seed>/
    results.json
    trajectory/
      trajectory.json
```

Additional figures and folders depend on the selected visualization plugins.
For example, diffusion, candidate trajectory, cost, and adaptive-schedule plots
are recipe-specific. A partially written directory is not proof of success.

## Inspect the evidence

| File or field | What to inspect |
| --- | --- |
| `protocol_manifest.json` | Resolved configuration, Git revision and dirty state, available package versions, learned checkpoint paths and SHA-256 hashes. |
| `results.json` | One seed/condition, metrics, resolved `config_snapshot`, provenance, and any execution status or partial metrics. |
| `trajectory/trajectory.json` | Planned candidate trajectories or an executed trajectory, depending on the method. Inspect its fields before interpreting it. |
| `summary.json` | Aggregated numeric results within one level or condition. |
| `overall_summary.json` | Completed and failed experiment counts, retained failures, and aggregate results. |

Trajectory schemas differ across task families. Candidate planning results can
include `candidate_states`, `candidate_actions`, and `candidate_costs`; direct
executed trajectories can include `states`, `actions`, `infos`, `costs`, task
signals, and execution status. Compare planned and executed trajectories only
when the task actually records both.

## Resume or retry

`--resume` reuses complete entries whose resolved configuration still matches.
Missing or mismatched entries run again. The current runner preserves an aborted
execution attempt and refuses to overwrite it: use a new development output root
for a new attempt. Keep the failed evidence when reporting a comparison.

## Compare results fairly

Record the method, task, seeds, device, horizon, sample count, checkpoints, and
metric definition. Separate cold compilation time from warm execution. The
[metrics contract](/docs/metrics) explains the reporting requirements;
the [configuration guide](/docs/configuration) explains matrices and
formal versus development runs.

## Implementation sources

This guide is derived from `ExperimentRunner` and the documented CLI; it does
not introduce a new artifact schema.

- [Matrix execution and resume](https://github.com/hhhhzl/genedynamics/blob/release/v1-open-source/genedynamics/experiments/framework/experiment.py#L846)
- [Protocol manifest](https://github.com/hhhhzl/genedynamics/blob/release/v1-open-source/genedynamics/experiments/framework/experiment.py#L984)
- [Per-run results and trajectories](https://github.com/hhhhzl/genedynamics/blob/release/v1-open-source/genedynamics/experiments/framework/experiment.py#L3046)
- [Aggregated results](https://github.com/hhhhzl/genedynamics/blob/release/v1-open-source/genedynamics/experiments/framework/experiment.py#L3199)
