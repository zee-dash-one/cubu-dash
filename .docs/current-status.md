# Current status

Snapshot of what is implemented vs stubbed vs unused. Prefer this when planning new work.

## Source of truth

| Concern | Prefer |
| --- | --- |
| What the game does at runtime | Code |
| Intended ring / transition / bounce feel | [`RULES.md`](../RULES.md) |
| Documented vs coded mismatches | This file |

Known mismatch: Level Select travel duration — RULES **800 ms** vs code `TRANSITION_MS = 0.35` (see [level-select.md](level-select.md)).

## Implemented

- Port / adapter / factory for **input** and **save**
- Exclusive schemes + Control Settings remap + touch overlay
- Persist input scheme/bindings to `user://save.json`
- Main Menu → Level Select (clock ring) → Game
- Auto-run scroll, lock X, jump / double jump, dash scroll boost, pause
- IDTCS on jump/dash: `player_jumped`, `player_dashed`, `player_dash_ended`; Player owns dash clock
- Light player FSM (`RUNNING` / `JUMPING` / `DASHING`); land on floor edge only
- Single `start_run` in `game._ready`; `end_run` from pause → Main Menu
- `InputManager.sync_input_map(play_exclusive)` — no read of `GameManager.is_running`
- Jump/land squash-stretch and ring settle / dock bounce language
- Under-construction routing for unfinished menu entries
- `IS_DEV_BUILD = true` (Debugger menu entry + DEV BUILD label)
- Level Select stores `current_world` / `current_phase` for future stage load (Game does not load content yet)
- Level Select: 4 worlds × 4 phases; only first world/phase unlocked; locked slots gray + denied anim on confirm
- Reactive SFX pending: `# TODO: AUDIO — …` markers at interaction peaks (search repo to wire AudioManager later)

## Stubs / placeholders

| Area | Status |
| --- | --- |
| `AudioManager` | Empty Godot template; no AudioPort |
| `PlayerStats` | Empty Godot template |
| Main Menu Sound / Graphic / Debugger | → `UnderConstruction.tscn` |
| Player art | `ColorRect` stand-in; unused `Sprite2D` / `AnimationPlayer` |
| `audio/` / `stages/` | Empty placeholders for future AudioPort and stage content |

## Reserved / incomplete APIs

| API | Notes |
| --- | --- |
| `EventBus.player_died` | **Reserved** — no death flow yet |
| `EventBus.coin_collected` / `GameManager.add_coins` | **Reserved** — emitter ready; no collectors / UI listeners |
| `GameManager.records` | Unused dictionary |
| Progress save | World/phase/coins/records not written to SaveLoad |
| Stage content | Flat ground only; selection ids stored but unused by Game |
| Hitstop / camera rumble | **Planned** — see [hitstop.md](hitstop.md); no service yet |
| Phase 00 Dev Lab + Tuning Panel | **MVP** — Debugger → lab; panel edits Player/Game live ([LD-00](sdd/LD-00-dev-lab.spec.md)) |

## Suggested next integration points

1. Expand Dev Lab zones (break/wall/fail) and Hitstop fields on Tuning Panel.
2. Lock Phase 01 SPEC from [sdd/phases/phase-01-first-run/brainstorm.md](sdd/phases/phase-01-first-run/brainstorm.md) (beat cut + fail/clear rules).
3. Stage data from World/Phase into Game (hazards, coins, win/lose) — SDD pipeline [LD-01](sdd/LD-01-level-design.spec.md).
4. Listeners for `player_jumped` / `player_died` / coins (VFX, audio, HUD).
5. Persist progress via `SaveLoad` keys (keep SavePort stack).
6. AudioPort + adapters when replacing `AudioManager` stub.
7. Hitstop service + camera shake on impact tiers ([hitstop.md](hitstop.md)); tune in Dev Lab.
8. Align Level Select `transition_duration` with RULES (800 ms) if feel target is adopted.
