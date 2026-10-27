# Input system

Exclusive device schemes. Gameplay and UI share the same action names; bindings come from the selected scheme, not hardcoded keys in gameplay.

## Contract

[`input/input_port.gd`](../input/input_port.gd) — `class_name InputPort`:

- `get_scheme() -> InputScheme.Id`
- `accepts_event(event) -> bool` — used while remapping in Control Settings
- `get_default_bindings() -> Dictionary` — action → `InputBinding` or `Array[InputBinding]`

## Schemes

[`input/input_scheme.gd`](../input/input_scheme.gd):

```gdscript
enum Id { KEYBOARD, MIX, GAMEPAD, TOUCH }
```

One scheme active at a time. Factory: [`input/input_factory.gd`](../input/input_factory.gd).

- `make(scheme)` → adapter
- `label` / `to_id` / `from_id` / `cycle`
- `default_scheme()` → `TOUCH` on mobile features, else `KEYBOARD`

## Actions

[`input/input_actions.gd`](../input/input_actions.gd):

| Gameplay | UI mirror (when applied) |
| --- | --- |
| `left` / `right` / `up` / `down` | `ui_left` … `ui_down` |
| `btn_a` | `ui_accept` |
| `btn_b` | `ui_cancel` |
| `btn_start` | (no mirror) |

Virtual IDs for touch: `virtual_left` … `virtual_start`. Helpers: `ui_mirror()`, `mirrored_actions()`, `display_label()`.

## Binding model

[`input/input_binding.gd`](../input/input_binding.gd) — kinds: `KEY`, `MOUSE_BUTTON`, `JOY_BUTTON`, `JOY_AXIS`, `VIRTUAL`.

Factories: `from_key`, `from_mouse`, `from_joy_button`, `from_joy_axis`, `from_virtual`, `from_event`, `from_dictionary`.  
Helpers: `to_event()`, `to_dictionary()`, `matches()`, `display_name()`.

## Adapters (defaults)

| Adapter | File | `accepts_event` | Default highlights |
| --- | --- | --- | --- |
| `KeyboardInputAdapter` | `keyboard_input_adapter.gd` | keys | WASD + arrows; J / K / Esc |
| `MixInputAdapter` | `mix_input_adapter.gd` | keys + mouse buttons | Move keys; RMB = A, LMB = B; Esc start |
| `GamepadInputAdapter` | `gamepad_input_adapter.gd` | joy button / axis | D-pad; A / B / Start |
| `TouchInputAdapter` | `touch_input_adapter.gd` | always false | All `VIRTUAL` bindings |

## InputManager

[`autoload/InputManager.gd`](../autoload/InputManager.gd)

- Loads/saves under SaveLoad key `"input"`.
- Signals: `scheme_changed(scheme)`, `bindings_changed`.
- Public API: `get_scheme`, `get_port`, `get_binding(s)`, `get_binding_label`, `set_scheme`, `set_binding`, `reset_current_scheme`, `begin_virtual_remap` / `end_virtual_remap`, `notify_virtual`, `is_touch_scheme`, `sync_input_map`.
- Owns `TouchControls.tscn` as a child overlay; visibility follows touch scheme and scene (hidden on Control Settings unless remapping).

### How bindings apply to InputMap

1. Selected scheme bindings → gameplay actions (and UI mirrors when appropriate).
2. Keyboard stays available for UI even when another scheme is selected — **unless** `GameManager.is_running` (play-exclusive).
3. Outside a run, MIX bindings also feed UI.
4. Mouse buttons are **not** mirrored to `ui_accept` (avoids double-firing menus).
5. Virtual bindings are pressed via `Input.action_press` / `release` from the touch overlay (`notify_virtual`).

`set_binding` swaps conflicting bindings on other actions in the same scheme (one control maps to one action).

## Persistence shape

```json
{
  "scheme": "keyboard",
  "bindings": {
    "keyboard": { "left": [ { "...": "InputBinding dict" } ], "...": "..." },
    "mix": {},
    "gamepad": {},
    "touch": {}
  }
}
```

Scheme ids: `keyboard` | `mix` | `gamepad` | `touch` (`InputFactory.to_id` / `from_id`).

## Control Settings UI

[`scenes/app/control_settings.gd`](../scenes/app/control_settings.gd) + `ControlSettings.tscn`

- Cycle scheme with `InputFactory.cycle` (prev/next or LEFT/RIGHT).
- CHANGE listens; only events accepted by `InputManager.get_port().accepts_event`.
- Touch remap: `begin_virtual_remap` → tap on-screen pad → `end_virtual_remap`.
- Reset: `reset_current_scheme()`; Back → `GameManager.go_main_menu()`.

## Touch overlay

[`scenes/hud/touch_controls.gd`](../scenes/hud/touch_controls.gd) → `InputManager.notify_virtual(virtual_id, pressed)`.
