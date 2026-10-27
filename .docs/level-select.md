# Level select (clock ring)

Implementation: [`scenes/select/level_select.gd`](../scenes/select/level_select.gd) + `LevelSelect.tscn`, slots via `CarouselItem`. Intended feel rules: root [`RULES.md`](../RULES.md).

## Flow

World stage → Phase stage → Game.

- North on the ring is the selected slot.
- Confirm World (`btn_a`): `dive_in`, then Phase with `zoom_in`.
- Confirm Phase: `dive_in` into Game (`GameManager` world/phase set, then change scene).
- Back from Phase (`btn_b`): `zoom_out`, then World with `dive_out`.
- Back from World: `zoom_out` toward Main Menu.

Menu → World uses `zoom_in` when entering from the main menu path.

## Ring gestures

| Gesture | Behavior |
| --- | --- |
| Short tap | Move exactly 1 / 2 / 3 slots (tap count), with acceleration |
| Hold | Continuous chase; look-ahead ramps 1 → 2 → 3 at 0 / 0.8s / 1.8s (`HOLD_LOOK_2`, `HOLD_LOOK_3`) |
| Reverse while spinning | Short reverse bounce, then one step the other way |
| Release after hold | Brief coast, then settle |

Settle **always** parks with a bounce: overshoot past the slot, then snap back. Keep the unwrapped angle (no TAU wrap snap). Rapid left/right must feel like holding, not stuck taps.

**Pending actions:** Confirm / Back / Left / Right pressed during TAP/HOLD are queued until `_park_on_slot` (latest wins). Same-direction HOLD keeps chasing without re-queue.

Code constants include `MAX_LOOK = 3`, `HOLD_MODE_AFTER = 0.22` (tap upgrades to hold), plus exports for speeds, settle bounce, reverse bounce, tap window.

## Circle transitions

Named animations; travel uses cubic ease-in.

| Name | Meaning |
| --- | --- |
| `zoom_in` | Arrive from far away into selection framing |
| `zoom_out` | Leave the current circle (pull back) |
| `dive_in` | Plunge into the north item |
| `dive_out` | Come from inside the camera out to selection framing |

### Dock bounce

Same idea as spin settle: advance a little past the rest pose, then return.

- Bounce **only** when about to park on stationary selection framing.
- Bounce **never** at the start of a transition.
- **Do** bounce at the end of `zoom_in` and `dive_out`.
- **Do not** bounce on `zoom_out` or `dive_in`.

Implementation: finish the travel tween first, then start a **new** tween for the dock bounce. Do not `chain()` bounce onto a `set_parallel(true)` travel tween.

Tune with exports: `transition_bounce_zoom`, `transition_bounce_pixels`, `transition_bounce_duration`. Related: `dive_zoom`, `zoom_out_scale`, `zoom_out_pull`, `zoom_in_start_scale`.

### Duration: RULES vs code

| Source | Travel duration |
| --- | --- |
| [`RULES.md`](../RULES.md) | **800 ms** |
| Code `TRANSITION_MS` / `transition_duration` | **0.35** seconds (350 ms); comment nearby has said “400ms” |

Runtime follows the code export. Treat RULES as the intended feel target when tuning.

## Procedural world data

`_build_world_data()` builds **4** named worlds (NEON CUBES … CANDY CORE), each with **4** phases. Only world/phase index `0` is unlocked (`locked: false`); the rest are locked. Navigation around the ring stays free; confirm on a locked slot plays a denied press anim and does not enter. Colors from `WORLD_COLORS`. Not loaded from disk; not tied to save progress yet.

### Confirm locked (IDTCS)

```text
INPUT:         BTN_A while IDLE on north item
DETECTION:     north item.locked == true
TRANSFORM:     play_denied() — quick scale peak then rest (150 ms)
COMMUNICATION: N/A (local UI)
STATE/LOGIC:   stay on select ring; no dive / no scene change
```
