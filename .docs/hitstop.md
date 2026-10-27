# Freeze frame / hit pause / hitstop

Cubu Dash will use **hitstop** (also called freeze frame or hit pause): on a meaningful impact, game time freezes for a **fraction of a second** so the hit reads as heavy. Stronger impacts may also trigger **camera rumble / shake**.

Not implemented in code yet — this is the design contract for when impacts exist (land weight, breakables, walls, bosses).

## Feel goal

Make contact *land* — the same bounce language as UI dock and jump squash, but for collisions: a brief hold, then motion continues. Pair with [reactive sounds](reactive-sounds.md) at the same peak (`TODO: AUDIO` until AudioManager exists).

## Intensity tiers

| Tier | When (examples) | Hitstop | Camera |
| --- | --- | --- | --- |
| **Light** | Cubu lands after a **double jump** (heavier than a normal soft land) | Very short freeze | None or tiny nudge |
| **Medium** | **Dash** breaks a destructible object | Clear freeze | Light rumble |
| **Strong** | Defeat a **boss**, or smash into a **hard wall** / immovable obstacle | Longer freeze | Full shake / rumble |

Normal single-jump soft lands should stay subtle (squash only, or no hitstop) so Light remains reserved for the double-jump land punch.

## Suggested starting tunables (not final)

Tune in playtests; do not treat as locked.

| Tier | Freeze window (guide) | Shake (guide) |
| --- | --- | --- |
| Light | ~2–4 frames (~30–70 ms) | Optional 1–2 px |
| Medium | ~4–8 frames (~70–130 ms) | Short rumble |
| Strong | ~8–14 frames (~130–230 ms) | Stronger shake, short decay |

Prefer **frame-based** or fixed ms via a single Hitstop service so feel stays stable if FPS dips. Do not stack unbounded freezes: a new stronger hit may replace a weaker one; same-or-weaker while already frozen is ignored or slightly extended (decide at implementation time — **ask** if unclear).

## IDTCS sketch

```text
INPUT:         impact resolved (land edge, dash-break, wall smash, boss kill, …)
DETECTION:     impact kind → tier Light | Medium | Strong
TRANSFORM:     freeze gameplay time briefly; optional camera offset/shake
COMMUNICATION: EventBus (e.g. impact_hitstop) → Hitstop + Camera; TODO: AUDIO — impact <tier>
STATE/LOGIC:   tier rules; no freeze during pause menu; resume scroll/physics after window
```

Pause menu (`get_tree().paused`) is separate from hitstop. Hitstop must not leave the tree paused or break UI.

## Architecture notes (when implementing)

- Own freeze + shake in a small run-scoped service or Game helper; gameplay emits **tier** (or impact id), not raw `Engine.time_scale` from every collider.
- Camera shake lives with the run camera (`scenes/run/`), driven by the same event.
- Keep adapters thin; hitstop is domain feel, not an input/save concern.

## Related

- [gameplay.md](gameplay.md) — run scroll / jump / dash
- [reactive-sounds.md](reactive-sounds.md) — SFX on the same impact peaks
- [sdd/LD-00-dev-lab.spec.md](sdd/LD-00-dev-lab.spec.md) — Dev Lab is the primary place to tune hitstop tiers when implemented
- Root [`RULES.md`](../RULES.md) — run feel summary
