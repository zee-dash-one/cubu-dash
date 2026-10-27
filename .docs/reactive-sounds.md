# Reactive sounds

Cubu Dash must answer every meaningful action with short **reactive SFX**. Music (BGM) is separate; reactive cues must stay readable over it.

## Until audio is implemented — `TODO: AUDIO` markers

`AudioManager` is still a stub and Sound settings are under construction. At every interaction peak, leave an English marker:

```gdscript
# TODO: AUDIO — <what just happened>
```

| Do | Don't |
| --- | --- |
| Add `TODO: AUDIO — …` at the fire point | Leave the peak silent with no marker |
| Keep descriptions short and specific | Vague notes like “sound here” |
| Replace markers later via EventBus → AudioManager | Scatter finished-looking play calls before the Port exists |
| Ask when final timbre is unknown | Invent a final sonic brand without human input |

Find all pending cues with a project search for `TODO: AUDIO`.

## When audio exists

- Route via COMMUNICATION (`EventBus` → `AudioManager` / AudioPort).
- Port → Adapter → Factory like input/save.
- One cue per interaction peak — not every physics tick.

## IDTCS slot

```text
COMMUNICATION: EventBus.<event> → AudioManager.play(<sfx_id>)
# Until then: # TODO: AUDIO — <event description>
```

## Suggested event → SFX map (current / near-term)

| Event | Reactive cue role | Marker example |
| --- | --- | --- |
| UI confirm | Bright accept click | `TODO: AUDIO — UI confirm start game` |
| UI deny / locked | Short buzz | `TODO: AUDIO — locked slot denied` |
| Pause / unpause | Soft whoosh | `TODO: AUDIO — run paused` |
| Jump / air jump | Sharp pluck | `TODO: AUDIO — player jumped` |
| Land | Soft thump | `TODO: AUDIO — player landed` |
| Dash start / end | Whoosh | `TODO: AUDIO — player dashed` |
| Coin (future) | Sparkle | `TODO: AUDIO — coin collected` |
| Death (future) | Crush | `TODO: AUDIO — player died` |

## Genre direction (Geometry Dash × Robot Unicorn Attack)

**A — Chiptune / 8-bit** · **B — Synth whooshes** · **C — Percussive body**  
Recommended start: **A + light C**; optional **B** on dash only. One-shots ~200–400 ms.
