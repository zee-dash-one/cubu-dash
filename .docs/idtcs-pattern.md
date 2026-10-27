# IDTCS — atomic interaction pattern

Standard way to design and implement every gameplay / UI interaction in Cubu Dash.

Aligned with Sense → Think → Act, plus explicit **communication** and **state**. Works together with Port → Adapter → Factory: adapters translate devices/backends; IDTCS describes what the **domain object** does.

## The five axes

Answer all five before writing code. If an axis does not apply, write **N/A** and why — do not invent behavior.

| Axis | Question |
| --- | --- |
| **I — INPUT** | What causes this action? Button press? Timer? Area entered? Event received? |
| **D — DETECTION** | Does the object need to query the world? Collision, floor check, raycast, inventory count, look-ahead? |
| **T — TRANSFORM** | What changes on the object or world? Position, velocity, animation, mesh, inventory contents? |
| **C — COMMUNICATION** | Does it emit or listen? EventBus signal, call into another system, save write? **Reactive SFX belong here** (bus → AudioManager). |
| **S — STATE AND LOGIC** | Which states flip? Which rules / guards / counts apply? What are the object’s directives? |

Reactive audio: every meaningful action needs a response sound unless the human marks SFX as N/A. See [reactive-sounds.md](reactive-sounds.md) and root [`RULES.md`](../RULES.md).

## Mandatory rule for agents

If any of the five axes is unclear, **stop and ask the human** before implementing. Do not guess INPUT bindings, DETECTION conditions, TRANSFORM feel, COMMUNICATION targets, or STATE names.

## Spec template

Use this block (in the PR/plan/chat) for each new interaction:

```text
Interaction: <name>

INPUT:        ...
DETECTION:    ... or N/A because ...
TRANSFORM:    ...
COMMUNICATION: ... or N/A because ...
STATE/LOGIC:  ...
```

## Examples

### Jump (player)

```text
INPUT:         btn_a just pressed (InputActions, via active scheme)
DETECTION:     jump_count < max_jumps (floor edge resets count)
TRANSFORM:     velocity.y = jump_force; jump squash/stretch tween
COMMUNICATION: EventBus.player_jumped
STATE/LOGIC:   jump_count += 1; state = JUMPING; on land → jump_count = 0, land squash, RUNNING
```

### Dash (player → world)

```text
INPUT:         btn_b just pressed while not already dashing
DETECTION:     is_dashing == false
TRANSFORM:     player dash timer; Game multiplies world scroll while dashing
COMMUNICATION: EventBus.player_dashed (start); EventBus.player_dash_ended (end)
STATE/LOGIC:   Player owns duration; Game boosts until ended (no second timer)
```

### Open chest / pick item (illustrative, other games)

```text
INPUT:         interact pressed while overlapping chest
DETECTION:     area overlap; chest closed; inventory has free slot
TRANSFORM:     chest open anim; item removed from chest / added to inventory UI
COMMUNICATION: inventory.add(item); maybe EventBus.item_collected
STATE/LOGIC:   chest.state = OPEN; inventory slots updated
```

## With Port + Adapter

| IDTCS piece | Prefer |
| --- | --- |
| INPUT | `InputActions` + `InputPort` / scheme adapters — never raw keys in gameplay |
| DETECTION / TRANSFORM / STATE | Domain scripts (Player, Game, UI controllers) |
| COMMUNICATION | `EventBus` or a Port — not hard references to concrete backends |
| Persistence side effects | `SaveLoad` → `SavePort` adapters |

Adapters stay thin. Business rules for the five axes stay outside adapters.

## Related docs

- [architecture.md](architecture.md) — Port / Adapter / Factory, EventBus
- [gameplay.md](gameplay.md) — current jump / dash behavior
- Root [`RULES.md`](../RULES.md) — feel rules and architecture summary
