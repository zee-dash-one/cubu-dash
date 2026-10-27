# Phase 01 — First run (W0 / P0) — brainstorm

**Spec parent:** [LD-01-level-design.spec.md](../../LD-01-level-design.spec.md)  
**Target slot:** World 0 **NEON CUBES** · Phase 0 (only unlocked phase today)  
**Goal of this phase:** First playable **authored** strip that teaches Cubu’s verbs and proves stage load from select → Game.  
**Status:** Brainstorm only — nothing locked; promote ideas into `SPEC.md` after review.

---

## 1. Context (do not forget)

### Already shipping
- Auto-runner: scroll world, lock X, jump / double jump / dash / pause
- Select ring with locks; dive into Game with `current_world` / `current_phase`
- Feel language: bounce squash, pending input on select, reactive `# TODO: AUDIO`
- Planned: hitstop tiers (Light / Medium / Strong)

### Missing (this phase should force the minimum)
- Stage loader + data under `stages/`
- Any hazard, coin, death, clear
- Breakables / walls (Medium–Strong hitstop need *something* to hit)
- Progress unlock beyond hardcoded `locked: index > 0`

### Fantasy references
- **Geometry Dash:** binary fail, rhythm read, teach-test-spike, camera foresight
- **Robot Unicorn Attack:** glam speed, dash as punctuation, big impact moments

Cubu hybrid: GD clarity + RUA juice (hitstop + SFX), still one-button-family verbs.

---

## 2. Design pillars for W0-P0

1. **One verb at a time** — Gate before Spike.
2. **Read, don’t snipe** — threats enter look-ahead before commit.
3. **Fail is fair** — if we add death, it must be readable 1 beat early.
4. **Juice on contact** — land / break / wall use hitstop + audio markers.
5. **Short** — first phase is a tutorial runway, not a marathon (aim ~45–90s clear for a learning player).

---

## 3. Beat map sketch (scroll left →)

Distances are relative “screens” of 1280px feel, not final units.

| # | Beat | Teach | Player action | Recover | Feel hook |
| --- | --- | --- | --- | --- | --- |
| 0 | Flat runway | Trust scroll | None | — | Soft land optional |
| 1 | Low gap | Single jump | Jump | Flat | Soft land |
| 2 | Twin low gaps | Jump rhythm | Jump · Jump | Flat | — |
| 3 | Raised block | Jump onto / over | Jump | Flat | Soft land |
| 4 | High gap | Double jump | Jump → Jump | Flat | **Light hitstop** on double-jump land |
| 5 | Dash lane | Empty dash | Dash through open air / stripes | Flat | Dash whoosh SFX |
| 6 | Break crate | Dash destroys | Dash into prop | Flat | **Medium hitstop** + break SFX |
| 7 | Combo | Jump then dash | Jump → Dash | Flat | — |
| 8 | Fake wall? | Optional: soft wall that kills or bounces | Dash too late = fail **or** Strong hitstop stub | Flat | Only if fail rules exist |
| 9 | End gate | Clear | Reach marker | — | Clear sting SFX |

**MVP cut:** beats 0–5 + end gate (no breakable/fail) if S3 not ready.  
**Juice cut:** beats 0–7 + end (breakable + Light/Medium hitstop).  
**Full Phase 01:** all beats including a single fair fail and restart.

---

## 4. Chunk ideas (reusable strips)

| Chunk id | Contents | Reuse later |
| --- | --- | --- |
| `flat_safe` | Long ground | Everywhere |
| `gap_s` | Small gap (1 jump) | W0 early |
| `gap_m` | Medium gap (often double) | W0 mid |
| `block_low` | Knee-high block | Pattern intro |
| `dash_open` | Visual stripes / no collision | Teach dash |
| `crate_1` | One breakable in dash height | Medium hitstop |
| `end_portal` | Clear trigger | All phases |

Authoring options to decide in SPEC:
- **A)** Godot packed scenes as chunks, placed by distance list  
- **B)** TileMap layers + marker nodes (spawn hazards)  
- **C)** Pure data (JSON) + factory spawns primitives  

Lean **A** for Cubu now: matches scene workflow, easy to art-swap.

---

## 5. Hazard / object brainstorm (S3 fodder)

| Object | Verb interaction | Fail? | Hitstop |
| --- | --- | --- | --- |
| Spike / saw | Jump over | Yes on touch | None or Light |
| Pit | Jump / double | Yes if fall | — |
| Crate | Dash breaks | No if dashed; Yes if body-block? | Medium on break |
| Hard wall | Dash / run into | Yes **or** stun | Strong |
| Coin / cube pickup | Pass through | No | None (sparkle SFX) |
| Boss (later worlds) | Pattern | Yes on hit | Strong on kill |

W0-P0 should introduce **at most**: pit/gap, one breakable, optional one spike. Boss is **not** Phase 01.

---

## 6. Camera & read

- Existing `camera_look_ahead` (~420) is the budget for “see threat”
- Chunk spacing must respect: threat enters view ≥ ~0.4–0.6s before contact at base scroll
- During dash (2.2× scroll), reads shorten — early dashes only in open lanes (beat 5)

---

## 7. Fail / clear (open questions for SPEC)

Record answers before coding:

1. Clear = end marker distance, or survive N seconds?
2. Fail = instant restart phase, or return to select?
3. Checkpoint mid-phase for W0-P0? (Suggest **no** for first teach phase.)
4. Does touching a crate without dash kill, bounce, or block?

---

## 8. SDD backlog derived from this brainstorm

Promote to Phase 01 `SPEC.md` requirement IDs when locking:

| Idea id | Statement (draft) |
| --- | --- |
| LD-P01-01 | Game loads stage for `(0,0)` from `stages/` instead of only flat ground |
| LD-P01-02 | Phase includes Gate beats for jump, double jump, dash in that order |
| LD-P01-03 | Double-jump land can request Light hitstop (even if service is stubbed behind EventBus) |
| LD-P01-04 | Optional crate beat requests Medium hitstop on break |
| LD-P01-05 | Clear emits a single end event; fail rules only if human locks them |
| LD-P01-06 | All new peaks have `# TODO: AUDIO — …` markers |
| LD-P01-07 | Completing W0-P0 does not yet unlock W0-P1 unless save progress SPEC exists |

---

## 9. Wild / later ideas (do not block Phase 01)

- Scroll speed eases up within a phase (RUA panic)
- Moving platforms that scroll with world
- Dash-through neon rings for score
- Ghost replay of best run
- Dual lane (up/down) — probably too much for Cubu’s single Y jump space
- Rhythm-synced obstacles to BGM BPM (needs audio system first)

---

## 10. Next human decision

Pick for the Phase 01 SPEC lock:

1. **MVP / Juice / Full** beat cut from §3  
2. Chunk authoring **A / B / C** from §4  
3. Answers to §7 fail/clear questions  

Then create `phases/phase-01-first-run/SPEC.md` with `LD-P01-xx` acceptance and implement S1 loader + content.
