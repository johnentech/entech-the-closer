# ENTECH: The Closer — Voice Recording Script

The voice actor plays **the player character**: the last IT tech on shift. They're tired, dry and sarcastic, but competent. Think "seen it all, and it's always DNS."

The wording below is a suggestion. Ad-libs are welcome. If a line changes, update its subtitle text in the `VOICE` table in `index.html` (search for `const VOICE`).

---

## Recording specs

| | |
|---|---|
| **Format** | WAV, 48 kHz, 24-bit, **mono** (masters). MP3 is fine for the game too |
| **Levels** | Peaks around −1 dB, no clipping. Keep the same mic distance for every line |
| **Room** | As dry as possible (closet, blankets). No music or background noise |
| **Trim** | Cut silence at the start and end. Leave about 50 ms before the first sound |
| **Takes** | Record 2–3 takes of each line. Pick the best one, and keep the others as variations |
| **One file per line** | Name each file exactly as shown below |

**Where files go:** `sounds/voice/` next to `index.html`, e.g. `sounds/voice/ammo_1.wav`.
The game tries `.wav` first, then `.mp3`. It skips missing files without errors, so lines can be added a few at a time. Refresh the page to pick up new files.

> Until a file exists, the browser console shows a "404 (Not Found)" for it. That's expected and harmless.

**Permission:** get written permission from the voice actor to use the recordings in the game (a short email is enough).

---

## Lines

**Chance** is how often the line plays when its event happens. Lines with a low chance stay fresh.

### Game start
Plays when you press a key on the title screen.

| File | Line |
|---|---|
| `start_1` | "Another Monday. Let's see what's broken." |
| `start_2` | "Okay. Who touched the DNS?" |

### Keycaps pickup (keyboard ammo), 50% chance, at most every 9 s
| File | Line |
|---|---|
| `ammo_1` | "More keycaps. Nice and clicky." |
| `ammo_2` | "Fresh keycaps." |
| `ammo_3` | "Reloading the keyboard." |

### Closures pickup (Closing Cannon ammo), 60% chance, at most every 9 s
| File | Line |
|---|---|
| `closures_1` | "Closures. Let's close some tickets." |

### Health pickup (coffee), 60% chance, at most every 9 s
| File | Line |
|---|---|
| `health_1` | "Ahh, coffee. SLA restored." |
| `health_2` | "Caffeine. The real uptime." |
| `health_3` | "That's the good stuff." |

### Closing Cannon pickup ✅ *recorded*
| File | Line |
|---|---|
| `cannon_1` | *(recorded: "WEAPON PICKUP")* |

### Flush DNS Cache pickup and use
| File | Line |
|---|---|
| `flush_get_1` | "Flush DNS cache. I'm gonna need that." |
| `flush_use_1` | "Flush THIS." |
| `flush_use_2` | "ipconfig, slash, flush... D-N-S." |

### Locked security door (Server Room), at most every 6 s
| File | Line |
|---|---|
| `locked_1` | "Access denied. Of course." |
| `locked_2` | "I need clearance. IT closet, probably." |

### Locked boss door (before the override switch), at most every 6 s
| File | Line |
|---|---|
| `locked_core_1` | "Core's locked down. There's an override in here somewhere." |

### Override switch flipped
Plays when you flip the switch inside the Server Room rack, which unlocks the boss door.

| File | Line |
|---|---|
| `switch_1` | "Override engaged. Time to talk to the DNS." |

### Boss encounter ✅ *recorded*
Plays when the arena door seals behind you and the DNS appears. This is the big one. Give it some weight.

| File | Line |
|---|---|
| `boss_intro_1` | *(recorded: "BOSS ENCOUNTER")* |

### Boss phase changes
| File | Line |
|---|---|
| `boss_phase2_1` | "Shadow IT? Who approved this?!" |
| `boss_phase3_1` | "TTL's hitting zero. Now or never." |

### Taking damage, every hit ✅ *recorded*
A line plays on every hit, 50/50 between the two. It can cut off a kill quip or pickup line, but never restarts itself mid-line.

| File | Line |
|---|---|
| `hurt_1` | *(recorded: "DAMAGE")* |
| `hurt_2` | *(recorded: "DAMAGE 2")* |

### Low health (SLA drops to 25% or below), at most every 20 s
| File | Line |
|---|---|
| `low_health_1` | "I need coffee. Now." |
| `low_health_2` | "SLA's about to breach..." |

### Out of ammo (the weapon runs dry), at most every 5 s
| File | Line |
|---|---|
| `empty_1` | "Out of keycaps!" *(keyboard)* |
| `empty_2` | "Keyboard's empty!" *(keyboard)* |
| `empty_cannon_1` | "Out of closures!" *(Closing Cannon)* |

### Enemy killed, 50% chance ✅ *recorded*
Half of kills get a quip, picked by weight. Only one line plays at a time, so killing a crowd at once doesn't stack them.

| File | Weight | Line |
|---|---|---|
| `kill_get_rekt` | 40% | "Get rekt!" |
| `kill_haha` | 40% | *(recorded: "HAHA")* |
| `kill_synez` | 20% | *(recorded: "SYNEZ")* |

### Death ✅ *recorded*
| File | Line |
|---|---|
| `death_1` | *(recorded: "DEATH")* |

### Victory ✅ *recorded*
Plays the moment the DNS dies (SERVFAIL).

| File | Line |
|---|---|
| `victory_1` | *(recorded: "VICTORY")* |

---

## Adding more variations
To add another take, put the file in `sounds/voice/` (e.g. `ammo_4.mp3`). Then add it to that event's `lines` in the `VOICE` table in `index.html`:

```js
ammo: { chance: 0.5, cooldown: 9000, priority: 1, lines: [['ammo_1', 'More keycaps. Nice and clicky.'], /* … */ ['ammo_4', 'Your new subtitle.']] },
```

Leave the subtitle as `''` for lines without words (grunts). An optional third value is a weight (see `kill`): weighted variants are picked by weight instead of evenly. In the same table you can tune `chance`, `cooldown` (ms) and `priority`; a higher-priority line interrupts a lower one.
