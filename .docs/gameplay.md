# Gameplay

Auto-runner feel inspired by Geometry Dash / Robot Unicorn Attack (`game.gd` comment): player X locked, world scrolls left, only jump and dash are controlled.

## Responsibilities

| Piece | File | Job |
| --- | --- | --- |
| Game | [`scenes/run/game.gd`](../scenes/run/game.gd) | Scroll `World`, lock player X, camera look-ahead, dash scroll boost |
| Player | [`scenes/run/player.gd`](../scenes/run/player.gd) | Gravity, jump / double jump, dash emit, squash/stretch visuals |
| Pause | [`scenes/run/pause_menu.gd`](../scenes/run/pause_menu.gd) | `btn_start` toggle |

## Scroll model

- On ready: `GameManager.start_run()`, store spawn X, set `_scroll_speed = base_scroll_speed`.
- Each physics frame: `world.position.x -= _scroll_speed * delta`, then re-lock player X to spawn.
- Player `velocity.x` is always `0` — horizontal progress is world motion only.

### Tunables (`game.gd`)

| Export | Default | Meaning |
| --- | --- | --- |
| `base_scroll_speed` | `350.0` | Normal world scroll |
| `dash_scroll_multiplier` | `2.2` | Scroll speed during dash |
| `camera_look_ahead` | `420.0` | Camera X offset ahead of player |

## Dash

1. Player presses `btn_b` → `start_dash()` → `EventBus.player_dashed.emit()`.
2. Game multiplies scroll by `dash_scroll_multiplier`.
3. When Player dash timer ends → `EventBus.player_dash_ended` → Game restores base speed.

Player owns the dash duration clock; Game does not use a separate timer.

## Jump

- Action: `InputActions.BTN_A` via `_try_jump()`.
- Up to `max_jumps` (default `2`); `jump_force` default `-600`; `gravity` default `1200`.
- Emits `EventBus.player_jumped`.
- Floor **edge** land resets `jump_count` and plays land squash (`RUNNING` / `JUMPING` / `DASHING` FSM).

## Visual feel

Player uses a `ColorRect` stand-in (`$ColorRect`). Jump/land tweens overshoot then rest — same bounce language as UI dock (see [`RULES.md`](../RULES.md) “Run feel”).

### Hitstop (planned)

Impacts will use freeze frame + optional camera rumble. Tiers: Light (double-jump land), Medium (dash breaks object), Strong (boss / hard wall). Full contract: [hitstop.md](hitstop.md). Not wired in code yet.

Unused scene nodes today: empty `Sprite2D`, `AnimationPlayer` on Player.

## World content (current)

One long green ground `StaticBody2D` under `World`. No coins, hazards, death, or win conditions yet. World/phase selection does not load distinct stage data into Game.

## Pause

`PauseMenu` listens for `btn_start` (toggle), uses `PROCESS_MODE_ALWAYS`. Resume button continues the run; Main Menu ends the run and navigates home. No global `BTN_A` resume.
