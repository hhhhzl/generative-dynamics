# Choose a method

Begin with a task and the behavior you need. The four research methods share
the framework, but they address different stages of constrained generative
planning and control.

| Your goal | Start with | Practical entry |
| --- | --- | --- |
| Embed a complete planning loop in Python | 2GO | [Direct Python example](/docs/python-api), CPU with the optimization extra. |
| Learn configured runs and inspect artifacts | MBD | [Planar CPU example](/docs/planar-cpu), core package only. |
| Put safety projections inside model-based diffusion rollouts | MDOC | `configs/single_2d/mdoc.yaml`. The original project's multi-robot CBS application is a separate entry. |
| Combine feasibility scoring, CFS projection, and adaptive scheduling | MD-COAS | `configs/single_2d/mdcoas.yaml`; then D3IL avoidance with its additional setup. |
| Shape updates and exploration using active constraint geometry | 2GO | [Humanoid corridor](/docs/humanoid-corridor) or quadruped stepping stones. |
| Refine policy proposals for contact-rich motion–impedance control | MGA | Peg insertion or surface scanning; install simulation dependencies and obtain the recipe's frozen learned assets. |

## Keep paper names and implementation names distinct

- **MDOC** uses `mdoc`.
- **MD-COAS** uses the historical `cfsmbd` / `cfsmbd_full` implementation family.
  The main `mdcoas.yaml` selects adaptive scheduling. MD-COAS-F and MD-COAS-A
  are configuration variants, not additional framework packages.
- **2GO** uses the method key `2go`; configuration filenames also use `twogo`.
- **MGA** uses `mga`. Learned priors and reliability settings are task-specific;
  read the selected configuration and asset contract.

## Switch a planner without changing the question

Use configurations from the same task family. Keep the task, condition, seeds,
metric definitions, and compute protocol fixed while comparing methods. Do not
assume changing the `method` field alone is sufficient: method-specific
parameters, constraint filters, or learned checkpoints may differ. The
[planner catalogue](/docs/planners) links the supported configurations
and comparison methods.

## Check the supported path

V1 is JAX-first. CPU is the portable entry; CUDA qualification varies by task.
DPCC and SafeDiffuser use specialized optional Torch integrations. The broader
research gallery includes hardware and humanoid contact demonstrations beyond
the qualified V1 recipe set. Consult [compatibility](/docs/compatibility)
and [release scope](/docs/v1-scope) before selecting a device or robot.

## Sources

Adapted from the repository [README](https://github.com/hhhhzl/genedynamics/blob/main/README.md),
[planner catalogue](/docs/planners), [recipes](/docs/recipes),
and [release scope](/docs/v1-scope).
