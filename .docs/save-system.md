# Save system

Port / adapter / factory for persistence. Game code talks to the `SaveLoad` autoload; I/O stays in the adapter.

## Pieces

| Piece | Path | Role |
| --- | --- | --- |
| `SavePort` | [`save/save_port.gd`](../save/save_port.gd) | Contract: `save_data(Dictionary)`, `load_data() -> Dictionary` |
| `FileSaveAdapter` | [`save/file_save_adapter.gd`](../save/file_save_adapter.gd) | JSON at `user://save.json` |
| `SaveFactory` | [`save/save_factory.gd`](../save/save_factory.gd) | `make_default() -> FileSaveAdapter` |
| Facade | [`autoload/SaveLoad.gd`](../autoload/SaveLoad.gd) | In-memory `_data`; every `set_value` rewrites the whole file |

## SaveLoad API

```gdscript
SaveLoad.get_value(key: String, default_value = null) -> Variant
SaveLoad.set_value(key: String, value: Variant) -> void  # updates dict + save_data
```

On `_ready`: `_port = SaveFactory.make_default()` then `_data = _port.load_data()`.

Missing or invalid file → empty dictionary. Write failures push a warning and skip.

## File format

Pretty-printed JSON (`JSON.stringify(data, "\t")`) at `user://save.json`.

Top-level keys are string identifiers chosen by callers (not a fixed schema beyond what features store).

## What is persisted today

| Key | Writer | Contents |
| --- | --- | --- |
| `"input"` | `InputManager` | `{ "scheme": "...", "bindings": { ... } }` — see [input-system.md](input-system.md) |

## What is not persisted yet

| Data | Where it lives | Notes |
| --- | --- | --- |
| `coins` | `GameManager.coins` | Session only; `add_coins` exists but no collectors |
| `records` | `GameManager.records` | Empty dictionary; unused |
| `current_world` / `current_phase` | `GameManager` | Set when entering a run from Level Select; not saved |
| Audio / graphic settings | — | Screens are under construction |

## Extending

1. Keep using `SaveLoad.get_value` / `set_value` from domain code.
2. For a new backend (Steam Cloud, etc.): implement `SavePort`, return it from `SaveFactory.make_default()` (or a chooser), leave `SaveLoad` API unchanged.
