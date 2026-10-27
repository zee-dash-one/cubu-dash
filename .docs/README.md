# Cubu Dash — technical docs

Local technical specifications for how the project works **today**.  
These files live under `.docs/` (gitignored). Feel / UX rules that the team must keep live in root [`RULES.md`](../RULES.md). Product overview lives in [`README.md`](../README.md).

## Product snapshot

Cubu Dash is a retro auto-runner in **Godot 4.7** (GL Compatibility). The player stays fixed on X; the world scrolls toward them. Jump (`btn_a`, double jump) and dash (`btn_b`) are the run controls. Main scene: `res://scenes/app/MainMenu.tscn`.

## Engine / project config

From [`project.godot`](../project.godot):

| Area | Value |
| --- | --- |
| Features | `4.7`, `GL Compatibility` |
| Main scene | `res://scenes/app/MainMenu.tscn` |
| Display | 1280×720, stretch `canvas_items` |
| Rendering | `gl_compatibility` (mobile too); Windows driver `d3d12` |
| Physics | Jolt listed for 3D; gameplay is 2D |
| Input actions | `left`, `right`, `up`, `down`, `btn_a`, `btn_b`, `btn_start` — empty in the project file; filled at runtime by `InputManager` |
| Autoloads | `EventBus`, `AudioManager`, `GameManager`, `PlayerStats`, `SaveLoad`, `InputManager` |

## Folder map

| Path | Purpose |
| --- | --- |
| `autoload/` | Global singletons |
| `input/` | `InputPort`, scheme adapters, factory, bindings, action names |
| `save/` | `SavePort`, file adapter, factory |
| `audio/` | Placeholder for future AudioPort + SFX/BGM wiring |
| `stages/` | Placeholder for future world/phase stage content |
| `scenes/app/` | MainMenu, ControlSettings, UnderConstruction |
| `scenes/select/` | LevelSelect, CarouselItem |
| `scenes/run/` | Game, Player, PauseMenu |
| `scenes/hud/` | TouchControls overlay |
| `assets/` | Fonts (e.g. Press Start 2P) |

## Doc index

| Doc | Contents |
| --- | --- |
| [idtcs-pattern.md](idtcs-pattern.md) | **Standard** Input → Detection → Transform → Communication → State (ask if unclear) |
| [reactive-sounds.md](reactive-sounds.md) | **Mandatory** reactive SFX on every meaningful action |
| [architecture.md](architecture.md) | Port / adapter / factory, autoloads, EventBus, naming |
| [input-system.md](input-system.md) | Schemes, bindings, InputManager, Control Settings, touch |
| [save-system.md](save-system.md) | SavePort, FileSaveAdapter, what is persisted |
| [scene-flow.md](scene-flow.md) | Menu → Level Select → Game → Pause / placeholders |
| [gameplay.md](gameplay.md) | Scroll world, jump, dash, camera |
| [hitstop.md](hitstop.md) | Freeze frame / hit pause + camera rumble tiers |
| [sdd/LD-00-dev-lab.spec.md](sdd/LD-00-dev-lab.spec.md) | Phase 00 Dev Lab + live Tuning Panel (dev-only) |
| [sdd/phases/phase-00-dev-lab/brainstorm.md](sdd/phases/phase-00-dev-lab/brainstorm.md) | Dev Lab catalog zones + panel UX brainstorm |
| [sdd/LD-01-level-design.spec.md](sdd/LD-01-level-design.spec.md) | Level design SDD pipeline + vocabulary |
| [sdd/phases/phase-01-first-run/brainstorm.md](sdd/phases/phase-01-first-run/brainstorm.md) | Phase 01 (W0-P0) level design brainstorm |
| [level-select.md](level-select.md) | Clock ring, transitions, dock bounce |
| [current-status.md](current-status.md) | Implemented vs stubs vs unused APIs |

## Source of truth

- **Runtime behavior:** the code.
- **Intended feel** (ring spin, transitions, bounce language): [`RULES.md`](../RULES.md). When RULES and code disagree, see [current-status.md](current-status.md).
