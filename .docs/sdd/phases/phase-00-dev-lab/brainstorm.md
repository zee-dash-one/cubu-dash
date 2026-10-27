# Phase 00 — Dev Lab — brainstorm

**Spec parent:** [LD-00-dev-lab.spec.md](../../LD-00-dev-lab.spec.md)  
**Slot:** Dev-only (`stages/dev/phase_00_lab`) — **not** Level Select W0-P0  
**Goal:** One catalog runway with everything Cubu can do, plus a live Tuning Panel for fine-tuning  
**Status:** Brainstorm + parent spec; **MVP lab + panel implemented** (catalog stubs; expand zones as mechanics land)

---

## 1. Why this exists

Phase 01 (W0-P0) must stay a clean **teach** strip. Fine-tuning jump height, dash length, hitstop frames, and hazard sizes is painful if you only have the tutorial layout.

Dev Lab = **museum of mechanics**: walk (scroll) past every exhibit, pause, tweak, resume, repeat.

---

## 2. Catalog strip (zones along scroll)

Order is for reading left→right in design notes; implementation can rearrange. Distances relative.

| Zone | Label (dev sign) | What to place | Feel to tune |
| --- | --- | --- | --- |
| A | JUMP | Small gaps, low block | `jump_force`, gravity, land squash |
| B | DOUBLE | High gap forcing air jump | Double-jump timing; **Light hitstop** on that land |
| C | DASH | Open lane / stripes | `dash_duration`, scroll multiplier, whoosh |
| D | BREAK | One crate at dash height | Break rules; **Medium hitstop** + rumble |
| E | WALL | Hard wall segment | Fail vs stun; **Strong hitstop** + shake |
| F | COIN | Pickup cubes | Collection FX/SFX; no fail |
| G | FAIL | One fair spike or pit | Death / restart hook |
| H | CLEAR | End portal | Clear event; optional sting |

**Stub policy:** If break/wall/coin/fail/clear/hitstop are not coded yet, use colored markers + debug print / EventBus stub so the **panel and layout** still exist.

Safe pads between zones so tweaking one exhibit does not instantly kill the run.

---

## 3. Dev Tuning Panel — UX sketch

```
+------------------------------------------+
|  [DEV]  <--- collapsed tab (always)      |
+------------------------------------------+

Expanded (corner, e.g. top-right):
+---------------------------+
| DEV TUNING          [x]   |
| Target: [Player    v]     |
| --------------------------+
| jump_force      [ -600 ]  |
| gravity         [ 1200 ]  |
| dash_duration   [ 0.20 ]  |
| max_jumps       [    2 ]  |
| ...                       |
+---------------------------+
```

- **Collapsed:** small tab labeled `DEV`; mouse click toggles expand  
- **Expanded:** scrollable list of fields for current target  
- **While paused:** tab and fields still receive mouse (panel `PROCESS_MODE_ALWAYS`)  
- **Gameplay keys:** unchanged; panel does not steal `btn_a` / dash unless focus is in a text field (implement carefully)

### First property groups to expose

| Target | Fields (from today’s `@export` / planned) |
| --- | --- |
| Player | `jump_force`, `gravity`, `dash_duration`, `max_jumps` |
| Game | `base_scroll_speed`, `dash_scroll_multiplier`, `camera_look_ahead` |
| Camera | look-ahead / shake amplitude (when hitstop camera exists) |
| Hitstop | Light / Medium / Strong duration + shake (when service exists) |

Edits write straight to the live node; optional “Reset defaults” later.

### Selection ideas

1. Dropdown of known targets (fastest to ship)  
2. Click mesh/body in world to select (best for many props)  
3. Both — dropdown + click (recommended long-term)

Lock combination at implement time; ask if still ambiguous.

---

## 4. Entry / flow

1. Main Menu → **Debugger** (visible only if `IS_DEV_BUILD`)  
2. Loads Game with Dev Lab stage + Tuning Panel autoloaded or scene child  
3. Pause with Start still works; open panel with mouse anytime  
4. Back to menu via pause Main Menu (same as normal run)

---

## 5. Open questions (non-blocking)

| # | Question | Default if unanswered |
| --- | --- | --- |
| 1 | Hotkey to toggle panel / reopen lab? | Debugger only for now |
| 2 | Persist tuned values to disk between runs? | No — session only until asked |
| 3 | Wall zone: fail or Strong hitstop bounce? | Ask when implementing Strong tier |
| 4 | Panel corner L/R? | Top-right |

---

## 6. Draft requirement IDs (for future SPEC lock / impl)

| ID | Draft |
| --- | --- |
| LD-P00-01 | Dev Lab stage descriptor under `stages/dev/` |
| LD-P00-02 | Debugger routes to lab when `IS_DEV_BUILD` |
| LD-P00-03 | Catalog zones A–H present (stub OK) |
| LD-P00-04 | Tuning panel DTP-01–08 from parent spec |
| LD-P00-05 | Player + Game tunables editable live |
| LD-P00-06 | Lab absent from release builds |

---

## 7. Next step after docs

Human approves this brainstorm → implementation pass: wire Debugger, minimal lab ground + panel MVP (dropdown targets first), then grow zones as mechanics land.
