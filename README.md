# Halation

**A person and an agent, making a scene together.**

![A volcanic massif at dusk with a smoke plume rising from the summit, rendered in Cycles](docs/hero.png)

Halation is an agent-native procedural 3D tool. You describe an asset or a scene, an AI agent builds it as a typed node graph, and you bake the result to engine-ready meshes, materials, terrain, volumes and renders. The agent works the way a person would: by reading and writing the document's own DSL, not by calling a hidden side API. Every step it takes is text you can read, diff, edit and re-run.

> **Project showcase.** This page is a case study of what Halation is and how it is built. The source is private. A code walkthrough is available on request.

## How it works

Most "AI + 3D" tools put a chat box in front of a fixed pipeline, and the model's moves disappear once the output lands. Halation makes the agent a first-class author of the document instead.

- **The document is the contract.** A scene is a typed node graph with typed ports (image, mesh, material, heightfield). The UI, the agent and the executor all read and write the same DSL, so anything the agent can do, you can see and change.
- **A director and its jobs.** A director agent turns an ask into jobs. Each job works in its own copy of the scene and merges back when it lands.
- **The board.** Runs appear on a board as they happen. When the agent is unsure, it asks: a short ballot of options, answered in a click. When a job has something to show, it presents a render and waits for a judgement. You steer at checkpoints, not by wiring nodes.

![The Halation app: a job transcript with render checkpoints above the board, where each card is a step of the run](docs/board.png)

The hero image above came out of one of these runs. The first render had no visible plume. After a redirect ("I don't see the smoke plume"), the director found the real defect on its own: the smoke domain was 1 metre across on a 3 kilometre scene. It sent the job a scale correction, and the next render is the one at the top of this page.

## What it makes

| | | |
|---|---|---|
| ![A server rack rendered in Cycles](docs/rack.png) | ![An interior door, half open](docs/door.png) | ![An eroded mountain massif in a clay pass](docs/massif-clay.png) |
| A server rack, from a short ask | An interior door, from a short ask | Eroded terrain, clay pass |

## Evaluation

The agent is only as good as the measurement behind it, so the eval suite is built like a product of its own.

- **198 agent cases** across terrain, fluids, scenes, modeling, materials, animation and simulation.
- **Two instruments, kept apart.** A *guard* suite answers "did I break something?": binary per case, every case must pass, deterministic checks only. A *frontier* suite answers "is the agent getting better?": graded per domain against a kept baseline, read only above a measured A/A noise floor, and it gates nothing. Cases graduate from frontier to guard once they pass reliably.
- **Judges earn their place.** Every model-graded score carries a calibration receipt against blind human grading on its own domain. Scoring is comparative (against a seed, graded exemplars, or another run) because every absolute LLM rubric measured here under-performed or inverted.
- **Promote or delete.** An advisory signal gets two sittings to clear a rule written down before scoring. Then it gates, or it is removed.

## Sim-ready assets for robotics

A sim-ready mode certifies articulated OpenUSD assets (cabinets, doors, drawers) for robot simulation. A deterministic battery runs in Isaac Sim over the baked USD:

- **Static:** structural checks plus NVIDIA's SimReady conformance validation.
- **Driven:** every joint driven through its full range and back to rest, an at-rest settle, a drop test, and an explosion detector watching the bounding box throughout.
- **Force:** a force-perturbation protocol measuring how far each joint travels under constant effort.

The battery is itself tested against deliberately broken assets. One example: a gravity-loaded drop-down door with an under-powered motor. It opens under gravity, so a simple range sweep passes it. The return-to-rest check catches that it cannot lift itself closed.

## GPU simulation

Two native Rust + wgpu kernels do the heavy compute, off the Python hot path.

- **Fluids:** smoke and fire on an Eulerian grid with multigrid pressure and a sparse block solver, plus FLIP/APIC liquids. Output is written as OpenVDB for Cycles, Houdini or any DCC, and the live viewport preview runs the same solver the bake uses. Deterministic, and 256³ at about 8 ms per step on an RTX PRO 6000. Every optimization was accepted or rejected by benchmark gates: field difference against a slow high-accuracy reference solve, divergence invariants, and a render-space detail metric.
- **Terrain:** hydraulic and thermal erosion on heightfields.

## Architecture

```
Electron shell
  └─ Frontend (Next.js)        React Flow graph editor · react-three-fiber viewport · the board
        │  REST + WebSocket
  Backend (FastAPI)            typed node graph · DSL · director + job agents · cook/bake
        │                      OpenUSD scene and project persistence · sim-ready certification · MCP server
        ├─ Rust kernels        shared semantic core (Python and wasm) · terrain erosion · fluids · OpenVDB
        ├─ Render worker       Cycles through hdCycles and OpenUSD Hydra (C++)
        └─ Generation service  image and mesh generation
```

## Stack

Python · FastAPI · Pydantic AI · Rust · wgpu / WGSL · OpenUSD · OpenVDB · Cycles · Isaac Sim · TypeScript · Next.js · React Flow · react-three-fiber · Electron

## Status

Active development (2026). Source private; walkthrough on request.

## License

© 2026 Nat Bishop. All rights reserved. This page is published for portfolio and evaluation purposes only.
