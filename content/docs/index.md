# GenerativeDynamics

**Generative dynamics on manifold.**

For robotics: **Planning. Control. Learning.**

![MDOC, MD-COAS, 2GO, MGA, and real robot demonstrations](/media/showcase-poster.png)

Train reusable policy priors, learn trajectory diffusion models, and perform
model-based generative inference with dynamics, manifold geometry and
constraints. Connect the resulting plans to closed-loop control through shared task, solver and execution
interfaces.

[Watch the full-quality video](https://github.com/hhhhzl/genedynamics/blob/main/docs/assets/showcase.mp4) ·
[Explore the task recipes](/docs/recipes)

## 🧩 Build with reusable components

| Feature | What you can use |
| --- | --- |
| **🧠 Robot learning** | PPO/SAC training and checkpoint reuse; PPO horizon priors for MGA. |
| **✨ Generative models** | Learned trajectory diffusion, model-based generative inference and composable reverse transports. |
| **Planner plugins** | MDOC, MD-COAS, 2GO, MGA and comparison algorithms through shared task and solver interfaces. |
| **Batched constraints** | Candidate evaluation, CBF/CFS correction and manifold feedback inside generative inference; see the [batch workflow](/docs/batched-constraints). |
| **Geometry and constraints** | Convex primitives, meshes, signed-distance geometry, collision constraints, state/action limits and constraint schedules. |
| **Robot task composition** | Semantic robot profiles, task objectives, dynamics and scene geometry kept separate from planner implementations. |
| **Simulation adapters** | MuJoCo, MJX, Brax and D3IL integrations with recipe-specific dependencies. |
| **Execution components** | Receding-horizon planning, followers, controllers, governors, safety filters and robot I/O. |
| **Experiment tooling** | Seeded configurations, resumable task matrices, metrics, trajectories, reports and observer plugins. |
| **Compute** | JAX-first planning with CPU setup and CUDA installation paths; device support is qualified per planner. |

## 🧪 Explore our algorithms

| Algorithm | Start building |
| --- | --- |
| **MDOC** | Run the planar planning component with `configs/single_2d/mdoc.yaml`; see the [original multi-robot project](https://github.com/hhhhzl/mdoc) for its CBS application. |
| **MD-COAS** | Explore constrained diffusion and adaptive scheduling with `configs/single_2d/mdcoas.yaml`, then D3IL arm avoidance with `configs/d3il_avoiding/mdcoas.yaml`. |
| **2GO** | Plan quadruped stepping stones and humanoid corridor motion; begin with the [humanoid corridor recipe](/docs/humanoid-corridor). |
| **MGA** | Reuse learned policy priors for generative motion–impedance control in [peg insertion and surface scanning](/docs/recipes#mga). |

The [algorithm catalogue](/docs/planners) links the papers, explains each
planner and maps published names to configuration keys.

## 🌍 Environments and constraints

Browse six [environment families](/docs/environments) and the
[constraint catalogue](/docs/constraints): CBF/CFS filters, adaptive
scheduling, and closed-form, JAXopt OSQP, OSQP and CVXOPT numerical solvers.

## 🚀 Start here

| Goal | Guide |
| --- | --- |
| Install the package and optional simulators | [Installation](/docs/installation) |
| Use an environment and planner in your own Python loop | [Direct Python example](/docs/python-api) |
| Run JAX planners on Linux/NVIDIA | [GPU workflows](/docs/gpu/) |
| Use isolated CPU or NVIDIA GPU environments | [Docker](/docs/docker/) |
| Initialize pinned robot descriptions and meshes | [MuJoCo Menagerie](/docs/mujoco-menagerie/) |
| Save trajectories, metrics and multi-seed experiments | [First experiment](/docs/quickstart) |
| Train models and reuse learned priors | [Learning workflows](/docs/learning-and-priors) |
| Choose a paper algorithm and task | [Recipes](/docs/recipes) |
| Understand or extend the interfaces | [Architecture](/docs/architecture) · [Plugin development](/docs/adding-a-plugin) |
| Check support or upcoming integrations | [Compatibility](/docs/compatibility) · [Roadmap](/docs/roadmap) |

![Complete task, learning, planning, simulation, deployment and evaluation workflow](/media/architecture-overview.png)

## 📍 Release status

V1 is JAX-first and includes six task families. Dedicated Torch integrations
support DPCC and SafeDiffuser; broader Torch support and Rust/C++ backends are
planned. MGA GPU qualification and humanoid walking/push are follow-up
priorities. See the [roadmap](/docs/roadmap), [compatibility matrix](/docs/compatibility) and
[V1 scope](/docs/v1-scope) for the current release boundaries.
