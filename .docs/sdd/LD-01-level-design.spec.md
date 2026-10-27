# Spec LD-01 — Level design SDD (Cubu Dash)

**Status:** Draft (brainstorm → lock requirements later)  
**Owner domain:** Run content (`stages/`, `scenes/run/`)  
**Depends on:** existing run loop (scroll, jump, dash), Level Select world/phase ids, [hitstop.md](../hitstop.md), [reactive-sounds.md](../reactive-sounds.md), IDTCS

## Purpose

Define how Cubu Dash **authors, stores, and ships stages** with Spec-Driven Development: each phase of content starts as a brainstorm, becomes a locked spec with acceptance, then implementation lands in `stages/` + run systems — without inventing mechanics the human has not approved.

## What we have today

| Layer | Reality |
| --- | --- |
| Run verbs | Jump (double), dash, pause; player X locked; world scrolls |
| Select | 4 worlds × 4 phases; only W0/P0 unlocked; ids stored on `GameManager` |
| Stage load | **None** — Game shows flat ground; world/phase unused |
| Feedback | Squash/stretch; EventBus jump/dash; `TODO: AUDIO`; hitstop **planned** |
| Folders | Empty `stages/`, empty `audio/` |

## What must come in sequence (pipeline)

Do not skip gates. Later phases may refine, not rewrite the contract without a new spec revision.

```mermaid
flowchart LR
  brainstorm[Brainstorm] --> vocab[LD vocabulary locked]
  vocab --> runtime[Stage runtime load]
  runtime --> teach[Teach phase W0-P0]
  teach --> verbs[Hazard and pickup set]
  verbs --> feel[Hitstop plus SFX on impacts]
  feel --> progress[Unlock and clear rules]
  progress --> expand[Worlds 1-3 content]
```

| Sequence | Deliverable | Unlocks |
| --- | --- | --- |
| **S0** | This spec + Phase brainstorm | Shared language for chunks / beats |
| **Tooling** | [LD-00 Dev Lab](LD-00-dev-lab.spec.md) + Tuning Panel (parallel, `IS_DEV_BUILD`) | Fine-tune all mechanics without using W0-P0 teach strip |
| **S1** | Stage data format + loader into Game | `current_world` / `current_phase` actually change the run |
| **S2** | First playable phase (W0-P0) | Teach jump → double jump → dash |
| **S3** | Hazard / breakable / wall / coin contracts | Medium/Strong hitstop targets |
| **S4** | Clear / fail / restart | `player_died`, win sting, records hook |
| **S5** | Unlock rules + save progress | Locked slots become progression, not hardcode |
| **S6** | Remaining phases / worlds | Content factory, not new engine per level |

Phase 00 Dev Lab is **tooling**, not a substitute for S2. See [phases/phase-00-dev-lab/brainstorm.md](phases/phase-00-dev-lab/brainstorm.md).

## SDD card shape (per phase or system)

Each work unit under `.docs/sdd/phases/` uses:

1. **`brainstorm.md`** — open ideas, references (GD / RUA), beat sketches (this file’s sibling pattern).
2. **`SPEC.md`** (when locked) — requirements `LD-xx`, acceptance bullets, out of scope, DoD.
3. **Implementation** — code + stage assets under `stages/`; IDTCS filled for every new interaction; `# TODO: AUDIO` until audio exists.

Agents: if any IDTCS axis or LD requirement is unclear → **ask**; do not invent boss/wall timings as final.

## Level-design vocabulary (working)

| Term | Meaning for Cubu |
| --- | --- |
| **Beat** | One readable challenge cycle (see → react → recover) at scroll speed |
| **Chunk** | Reusable strip of ground/hazards (TileMap / scenes) placed along X |
| **Gate** | Soft teach moment: only one new verb or hazard type |
| **Spike** | Difficulty peak; must be preceded by a Gate for that pattern |
| **Recover** | Safe scroll gap so the player resets rhythm |
| **Read distance** | How far ahead the camera look-ahead must show the threat |
| **Fail** | Contact that ends the run (or damages — decide in S3/S4) |
| **Clear** | Reach end marker / survive distance — decide in S4 |

## Player verbs → design pressure

| Verb | Design use |
| --- | --- |
| Single jump | Gaps, low blocks, rhythm hops |
| Double jump | High gaps, “air correction”, Light hitstop land |
| Dash | Timing windows, breakables (Medium hitstop), speed-read hazards |
| No move on X | All difficulty is **in the world scrolling at the player** |

## Stage data (target direction for S1)

Prefer data the select screen already knows:

- Key: `(world_id, phase_id)` → stage descriptor in `stages/`
- Descriptor lists ordered **chunks** and/or beat timeline (scroll distance or time)
- Theme color / name can mirror Level Select procedural names until art exists
- Game loads descriptor in `_ready` after `start_run` using `GameManager.current_world` / `current_phase`

Exact schema is **out of this draft** — lock in Phase 01 SPEC after brainstorm.

## Feel hooks every content beat should plan

- Reactive SFX at peaks ([reactive-sounds.md](../reactive-sounds.md))
- Hitstop tier if impact ([hitstop.md](../hitstop.md))
- Squash/stretch already on jump/land — content should not fight it

## Out of scope for LD-01 draft

- Final art, music stems, exact ms for every spike
- Full 4×4 authored content
- Online leaderboards

## Acceptance (spec draft itself)

- [x] Pipeline S0–S6 written
- [x] Vocabulary defined
- [ ] Phase 01 brainstorm exists and references this spec
- [ ] Phase 01 SPEC locked (separate step after human review of brainstorm)

## Related

- Phase 01 brainstorm: [phases/phase-01-first-run/brainstorm.md](phases/phase-01-first-run/brainstorm.md)
- [scene-flow.md](../scene-flow.md), [gameplay.md](../gameplay.md), [current-status.md](../current-status.md)
