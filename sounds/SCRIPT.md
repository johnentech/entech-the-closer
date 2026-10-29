# DNS: The Reckoning — Voice Recording Script

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

**Where files go:** `sounds/voice/` next to `index.html`, e.g. `sounds/voice/ammo_1.mp3`.
The game tries `.mp3` first, then `.wav`. It skips missing files without errors, so lines can be added a few at a time. Refresh the page to pick up new files.

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

### Ammo pickup (tickets or closures), 50% chance, at most every 9 s
| File | Line |
|---|---|
| `ammo_1` | "Tickets. Great. More tickets." |
| `ammo_2` | "Queue's full again." |
| `ammo_3` | "Reloading the backlog." |

### Health pickup (coffee), 60% chance, at most every 9 s
| File | Line |
|---|---|
| `health_1` | "Ahh, coffee. SLA restored." |
| `health_2` | "Caffeine. The real uptime." |
| `health_3` | "That's the good stuff." |

### Closing Cannon pickup
| File | Line |
|---|---|
| `cannon_1` | "Closing Cannon. Now we're resolving tickets." |

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

### Boss encounter
Plays when the arena door seals behind you and the DNS appears. This is the big one. Give it some weight.

| File | Line |
|---|---|
| `boss_intro_1` | "Of course. It's always DNS." |

### Boss phase changes
| File | Line |
|---|---|
| `boss_phase2_1` | "Shadow IT? Who approved this?!" |
| `boss_phase3_1` | "TTL's hitting zero. Now or never." |

### Taking damage, 35% chance, at most every 1.5 s
Short grunts and "oof"s, no words. These play often, so record **4 different ones** and keep them under half a second.

| File | Line |
|---|---|
| `hurt_1` … `hurt_4` | *(grunt / pained exhale / sharp "agh" / "oof")* |

### Low health (SLA drops to 25% or below), at most every 20 s
| File | Line |
|---|---|
| `low_health_1` | "I need coffee. Now." |
| `low_health_2` | "SLA's about to breach..." |

### Out of ammo (you fire on empty), at most every 5 s
| File | Line |
|---|---|
| `empty_1` | "Out of tickets!" |
| `empty_2` | "Queue's empty!" |

### Enemy killed, 15% chance, at most every 10 s (optional)
| File | Line |
|---|---|
| `kill_1` | "Ticket closed." |
| `kill_2` | "Resolved." |
| `kill_3` | "Marked as duplicate." |

### Death
A death cry or groan, no words needed, about 1–2 s.

| File | Line |
|---|---|
| `death_1` | *(death cry)* |
| `death_2` | *(defeated groan, fading out)* |

### Victory
Plays when "CONNECTION ESTABLISHED." appears after the boss dies.

| File | Line |
|---|---|
| `victory_1` | "Resolved. It was DNS. ...It's always DNS." |

---

## Adding more variations
To add another take, put the file in `sounds/voice/` (e.g. `ammo_4.mp3`). Then add it to that event's `lines` in the `VOICE` table in `index.html`:

```js
ammo: { chance: 0.5, cooldown: 9000, priority: 1, lines: [['ammo_1', 'Tickets. Great. More tickets.'], /* … */ ['ammo_4', 'Your new subtitle.']] },
```

Leave the subtitle as `''` for lines without words (grunts). In the same table you can tune `chance`, `cooldown` (ms) and `priority`; a higher-priority line interrupts a lower one.
