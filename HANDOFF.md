# ENTECH: The Closer — Session Handoff

## What this is
IT-themed Doom clone built in a single HTML file using canvas raycasting (no libraries).
Title: **ENTECH: The Closer** (renamed from "DNS: The Reckoning"; the repo folder still has the old name). Title screen: "E N T E C H :" in bright blue `#2aa4ff` over "THE CLOSER". Player is an IT tech who fights office equipment and ultimately battles the DNS boss.

**File:** `C:\Users\JohnMiller\Documents\DNS The Reckoning\index.html` (own git repo, separate from the Solitaire project)

---

## What's done

### Chunk 1 — Raycasting Engine ✅
- DDA raycasting with perspective-correct wall rendering
- Two-tone wall shading (EW face lighter, NS face darker)
- Dark navy ceiling, dark brown carpet floor
- Player movement: W/S move, A/D strafe, ←/→ turn, mouse look via pointer lock
- Slide-based collision with margin
- Mini-map overlay (toggle M)

### Chunk 2 — Combat Foundation ✅
- Keyboard weapon sprite drawn with canvas rects, kick animation + muzzle flash glow
- HUD: SLA health bar (green→yellow→red), ammo counter, weapon name label
- Fire: left-click (after pointer lock) or Space

### Chunk 3 — Enemies ✅
- **Turning fix:** ←/→ and mouse look were inverted (left turned right). Fixed in `update()`.
- `castRay(ox, oy, dx, dy)` — shared DDA (render, hitscan, LOS). Sets globals `rayHit`, `raySide`, `rayWallX`.
- `hasLOS(ax, ay, bx, by)`, `tryMove(obj, mx, my, r)` — shared slide collision (player + enemies)
- **Sprite renderer** `drawSprites()` — billboards sorted far→near, per-column clip against `zBuf[]`. Sprite entry: `{x, y, tex, scale (fraction of wall height), z (bottom height above floor), alpha?}`
- **Procedural textures** in `TEX` — drawn once into offscreen canvases (`makeTex`, `tintTex`, `radialTex`). Each enemy has `walk0 / walk1 / attack / pain / dead` frames.
- **Enemies** (`ENEMY_TYPES`, `spawnEnemy(type, x, y)`):

| Type | HP | Speed | Attack |
|---|---|---|---|
| `printer` | 110 | 0.022 | Ink blob projectile (16 dmg, speed 0.07) at ≤9 tiles; paper-cut melee 20 at ≤1.2 |
| `router` | 30 | 0.014 | 3-pellet hitscan burst, 4–7 each (`BURST_DMG`), accuracy falls off with range |
| `shadow` | 300 | 0.065 | Melee 22 at ≤1.15, fast cooldown. **Not placed in the map — boss spawns it via `spawnEnemy('shadow', x, y)`** |

(Values above are after play-test round 1 — see below.)

- **AI states:** `idle → chase → windup → (attack) → chase`, plus `pain` (interrupts windup) and `dead`. Wake on sight (LOS within `sight` range), on being shot, or on hearing gunfire/explosions (`alertEnemies`: <5 tiles, or <10 with LOS).
- **Pathing:** BFS flow field from the player's cell (`updateFlow`, rebuilt when the player changes cell or when `flowCell = -1` is set after a map change; 8-way with no corner cutting). Enemies with LOS walk straight at you; otherwise they step downhill on the field.
- **Hitscan:** keyboard casts 7 rotated rays; each hits the nearest enemy (ray vs circle `def.radius`) before the wall. Per-pellet damage via `pelletDamage(dist)` (see play-test round 1). Damage is summed per enemy per shot so pain is rolled once. `damageEnemy()` returns true on a kill.
- **Player damage:** `hurtPlayer(dmg)` + red flash. At 0 HP: `game.over`, placeholder "YOUR TICKET HAS BEEN CLOSED / Press R" overlay (real death screen is Chunk 6).
- **Pulled forward from Chunk 6:** `game.kills` + "TICKETS CLOSED" counter (top right).

### Chunk 4 — Level ✅
- **Map is now defined as strings** (`LEVEL[]`, 40×30) and parsed into `MAP[][]` + `LEVEL_SPAWNS`. Edit the level by editing the strings:

| Char | Meaning | Char | Meaning |
|---|---|---|---|
| `#` | concrete (1) | `P` | player start (faces east) |
| `=` | cubicle divider (2) | `p` / `r` | Printer / Rogue Router |
| `S` | server rack (3) | `t` | tickets (+8 keyboard ammo) |
| `D` | boss door (4) | `c` | coffee (+20 SLA) |
| `K` | security door (7) | | |
| `X` | override switch (8) | | |
| `%` | IT closet shelving (5) | `C` | Closing Cannon |
| `@` | arena wall (6, DNS purple) | `o` | closures (+3 cannon ammo) |
| | | `F` | Flush DNS Cache charge (Chunk 5) |
| | | `B` | DNS boss spawn → `BOSS_SPAWN` |

- **Route:** Office/cubicle farm (top-left, 3 Printers) → east hallway (Router) → **IT Closet** (top-right, shelving aisles, Printer + Router guarding the **Closing Cannon**) → hallway down (Printer) → **Server Room** (bottom-left, rack clusters, 4 Routers + Printer) → **Boss Door** (east wall of server room, `BOSS_DOOR = {x:25, y:22}`) → **Arena** (bottom-right, 13×17 open room with 4 pillar pairs, coffee + closures by the entrance). Office also connects straight down to the server room — but both server-room entrances are security doors that only unlock once you have the cannon (play-test round 1), so the closet is mandatory.
- 12 enemies placed (14 after play-test round 1); 14 pickups (4 tickets, 5 coffee, 4 closures, 1 cannon) + 60% ticket drops (+4) from kills.
- **Weapons:** `WEAPONS` table (`label`, `ammoLabel`, ammo/max keys on `game`, `COOL_MS`, `KICK`). `game.weapon` = `'keyboard' | 'cannon'`. Switch with **1 / 2 / Q / mouse wheel**; `switchWeapon(name)` lowers then raises the weapon (`weapon.state` `lowering → raising`, `SWITCH_MS = 160` each). A switch requested mid-shot starts when cooldown ends; switching back mid-swap cancels it.
- **Closing Cannon** `[ CLOSING CANNON ]`, ammo label **CLOSURES** (max 10, pickup gives 5 and auto-switches): rocket projectile `ROCKET_SPEED = 0.16` (sub-stepped ×4 so it can't tunnel), explodes on wall or enemy contact. `explode()` deals `SPLASH_DMG` (now 140) with linear falloff over `SPLASH_RADIUS = 2.2`, blocked by walls (LOS). **Self-damage** at `SELF_DMG_MULT` (now 0.4) (≈48 if you fire into a wall in your face). Cooldown 950 ms. Explosion = growing/fading `TEX.explosion` sprite for 350 ms.
- **Pickups** generalized: `pickups[] = {kind, x, y, amount?}`; `PICKUPS[kind] = {tex, scale, apply}` — `apply` returns a message string, or `false` to leave it on the floor (full health / full ammo).
- **Boss door:** red door with frame, seams and lock light (`drawDoorColumn`, uses `rayWallX`). `[ E ] OPEN` prompt when facing it within ~1.8 tiles (`doorAhead()`); **E** opens it (cell → 0). Once the player is >1.3 tiles past it, `checkBossTrigger()` re-seals it, nudges any enemy out of the doorway, and calls **`startBossFight()`** (Chunk 5 — spawns the DNS). Door can't be reopened after that.
- HUD: weapon label + `[1] keyboard 2 cannon` slot hint once you own the cannon; ammo label/count follow the current weapon. Top-center message line (`showMessage(text, ms)`) for pickups and events.
- Minimap shrunk to 5px cells and colored by wall type (door red, arena purple); rockets shown in orange.

### Play-test round 1 ✅ (user feedback → changes)
- **"Enemies should hit harder / have more HP":** Printer HP 60 → 110, ink 10 → 16, melee 12 → 20; Router burst 3–5 → 4–7 per pellet (HP stays 30 — "fine as weaker enemies"); Shadow IT HP 150 → 300, melee 15 → 22. Printer/Router wind-ups slightly longer (500 / 600 ms) so the bigger hits are telegraphed.
- **"Printers: 1–2 hits close, 3–4 far":** keyboard pellets now do random 13–19 each, full damage to 2 tiles then linear falloff to 0.6× at 10 tiles (`PELLET_DMG_MIN/MAX`, `FALLOFF_*`). Measured blasts-to-kill on a Printer (400 trials each): 1.5 tiles 74% one-shot / 26% two; 3 tiles 1–2; 5 tiles 2; 7 tiles mostly 3; 9 tiles mostly 3–4.
- **"Cannon is fine":** kept the feel; `SPLASH_DMG` 110 → 140 so a direct hit still kills the tougher Printer, `SELF_DMG_MULT` 0.5 → 0.4 so self-damage stays ≈ the same.
- **"Lock the server room until the cannon":** new wall type `K` = **security door** (7, steel with yellow/black hazard band) at both server-room entrances (8,16) and (22,16). Locked (red light, "ACCESS DENIED — CLEARANCE IS IN THE IT CLOSET") until `game.hasCannon`; then green light, E to open. `doorLocked(type)` covers both door kinds. Two extra Printers added in the server room (3 Printers + 4 Routers) so the cannon gets a real encounter.
- **"Enemy attacks more noticeable":** pulsing additive glow over an enemy while it winds up (`WINDUP_GLOW_TEX`), muzzle flash on the attacker when it fires (`ATTACK_FLASH_TEX`), bigger glowing ink blobs with a trail and wall splat, Router misses kick up sparks around you, red vignette on hit (stronger), **screen shake** scaled by damage, and **red direction arcs** around the crosshair pointing at whoever hit you (`damageDirs`, tracks as you turn).
- **"Keyboard attack more noticeable":** muzzle flash was effectively one frame (bug) — now fades over `FLASH_MS = 130`, and is bigger/brighter; brief white screen flash; screen shake; bigger kick (44px); **keycaps fly off the keyboard** toward the crosshair (`keycaps`, `drawKeycaps`); impact puffs where pellets land (paper scraps on Printers, sparks on Routers, dust on walls); crosshair with an **X hit marker** (red on a kill).
- **Bug fixed:** boss trigger only checked `player.x`, so walking into the IT Closet started the boss fight and permanently locked the boss door. Now it requires the player to be inside the arena (`ARENA` flood-fill mask).
- Effects plumbing: `effects[]` + `addEffect(x, y, z, tex, scale, ms, rise)` for short-lived billboards; `updateEffects()` prunes effects/explosions/keycaps/damageDirs; sprites support `alpha` and `blend`.
- Environment graphics (wall textures etc.) deliberately deferred — user wants to revisit later.

### Chunk 5 — Boss: The DNS ✅
- **The boss is an `enemies[]` entry** of type `dns` (`ENEMY_TYPES.dns`: **3200 HP**, radius 0.8, sprite scale 1.4), so keyboard hitscan, rocket contact and splash all work unchanged. Its AI is `updateBoss()` (called from `updateEnemies()` instead of the normal state machine). `game.boss` points at it. It floats (`e.z` bobs ~0.12; sprites now honor per-enemy `z`) and isn't pushed by other enemies.
- **Look:** `drawDNS()` — wireframe globe with lat/long grid, one red eye, A/MX/NS/TXT/CNAME labels orbiting on a tilted ring, purple aura. Frames: `idle0/idle1` (labels rotate), `attack` (grid + labels red, eye white-hot), `stun[8]` (eye replaced by a loading spinner), `pain` (white tint), `dead` (dim, cracked, X eye).
- **Spawn:** `startBossFight()` spawns it at `BOSS_SPAWN`, first attack after 2 s.
- **Phases** (`BOSS_PHASES`, by HP fraction, `bossPhase()`):

| Phase | HP | Name | Move speed | Volley | NX speed | Attack every | Shadow IT adds |
|---|---|---|---|---|---|---|---|
| 1 | >50% | RESOLVING | 0.018 | 3 | 0.05 | 2.4 s (650 ms wind-up) | — |
| 2 | >25% | QUERY FLOOD | 0.022 | 4 | 0.055 | 2.0 s (550 ms) | every 9 s, max 2 |
| 3 | ≤25% | TTL 0 | 0.03 | 5 | 0.065 | 1.2 s (380 ms) | every 5 s, max 3 |

- **Movement:** drifts between random arena waypoints 4–8 tiles from the player (`randomArenaCell`, `ARENA_CELLS`), backs off if you get within 3. Stops to wind up (red glow) before each volley.
- **NXDOMAIN** (`projectiles` kind `nx`): billboard reading "NXDOMAIN", fanned `NX_SPREAD` 0.45 rad apart, each curves toward the player at `NX_TURN` 0.012 rad/frame, 18 dmg, dies on walls/pillars or after `NX_LIFE` frames.
- **Shadow IT adds** spawn ≥5 tiles from you with a purple burst, flagged `bossAdd`; they always drop loot (65% +6 tickets, 35% +3 closures). Phase changes announce themselves ("QUERY FLOOD — SHADOW IT INBOUND", "TTL 0 — CACHE POISONED") with a screen shake.
- **TTL 0 flicker** (`drawGlitch`): random dark flashes + horizontal tear bands while phase 3 is active; boss bar flashes red.
- **Flush DNS Cache:** pickup kind `flush` (map `F`): one hidden in the server room's bottom-left corner (1,28), one in the arena's far corner (38,28). It's an **inventory charge** (`game.flushes`, HUD badge "[F] FLUSH DNS CACHE ×n"); **F** → `useFlush()`: stuns the boss for `STUN_MS` 4 s (no moving/attacking/spawning), wipes NXDOMAINs in flight, cyan screen flash, "ipconfig /flushdns" message. Using it with no boss says to save it.
- **Arena supplies** added: tickets ×2, closures ×2, coffee ×2 (+ the original coffee/closures by the door).
- **Boss bar** (`drawBossBar`, top center): "THE DNS", HP with 50%/25% ticks, phase name or "CACHE FLUSHED" while stunned.
- **Death** (`bossDie`): all adds and NXDOMAINs die with it; giant jittering **SERVFAIL** (`drawServfail`) over a 2.6 s chain of explosions as it sinks, then `game.won` → placeholder **"CONNECTION ESTABLISHED."** overlay with tickets closed, R to replay (`drawVictoryOverlay`, to be replaced in Chunk 6).
- **Splash vs the boss** is measured to its surface (center distance − radius) so a rocket into a big target isn't penalized.
- **Scripted fight** (perfect aim, never moving, infinite keyboard ammo, damage tallied not applied): won in 27 s with 39 keyboard blasts; phase 2 at 10 s, phase 3 at 18 s; 3 Shadow IT spawned; standing still costs ≈17 HP/s — a real player must dodge and lean on the cannon (39 blasts is more than a full 25-ticket load).

### Chunk 6 — Polish ✅
- **Title screen** (`drawTitle`, `game.started = false`): the camera slowly pans around the office behind an RGB-split "THE CLOSER" title (originally "THE RECKONING"), a terminal teaser (`nslookup` timing out), the controls, and a blinking "PRESS ANY KEY TO START". Any key or a click starts (`startGame()`); a click also grabs the mouse. **N** toggles sound here too, without starting.
- **Death screen** (`drawDeathOverlay`): the view slumps sideways to the floor, and the weapon drops away. "YOU'VE BEEN CLOSED." appears over a helpdesk ticket whose fields fill in one by one: random `#INC` number, subject, cause (`game.killedBy`, from `hurtPlayer(dmg, x, y, cause)` → `DEATH_CAUSES`), tickets closed, time on shift, and a random joke `RESOLUTIONS` entry. A red **CLOSED** stamp slams down at `STAMP_AT` 1.3 s with a shake and thud. R works after that.
- **Victory screen** (`drawVictoryOverlay`): after SERVFAIL, the screen fades to a terminal. `ipconfig /flushdns`, `nslookup`, `ping` are typed with key clicks and their output resolves (`VICTORY_TERM`/`VICTORY_SCHED`). Then a glowing **CONNECTION ESTABLISHED.**, stats (tickets closed, time to resolve, SLA remaining, rank from `victoryRank()`), "It was DNS. It's always DNS." and R to play again (after `VICTORY_BANNER_AT`).
- **Restart:** R → `restart()` sets a `sessionStorage` flag and reloads, which skips the title (`startGame(true)`). Audio resumes on the next key or click.
- **Sound effects** (`// --- Audio ---`): all synthesized with Web Audio, no files. `tone()` and `noise()` are the building blocks, and the `SFX` table holds every sound (`gap` = minimum ms between repeats). `sfx(name, x, y)` makes a sound positional: it fades with distance, is muffled without line of sight, and pans left/right. A compressor on the master output prevents clipping. Hooked up so far: both weapons, dry-fire click on empty, weapon switch, explosions, per-enemy wake/windup/pain/death, ink/burst/melee attacks, ink splats, the DNS (intro roar, windup, NXDOMAIN chirps, pain, phase alarm, SERVFAIL), player hurt/death, the CLOSED stamp, pickups (ammo/coffee/cannon/flush each have their own sound), doors (open/denied/arena slam), Flush DNS whoosh, title start jingle, victory chord and terminal typing. A UPS-style beep sounds every 1.4 s while SLA ≤ 25 (`LOW_HEALTH`).
- **Voice lines** (player character, recorded by a voice actor): the `VOICE` table maps events → `[file, subtitle]` variants plus `chance`, `cooldown` and `priority`. Files load from `sounds/voice/<file>.mp3`, falling back to `.wav`. Missing ones are skipped silently (they show up as 404s in the console until recorded). `say(event)` plays on a single voice channel: a higher priority interrupts, and the same variant never plays twice in a row. Effects are ducked under the voice, and subtitles are drawn by `drawSubtitles()` (`SUBTITLES` const). Events wired: start, ammo, health, cannon, flush_get, flush_use, locked, locked_core, switch, boss_intro, boss_phase2/3, hurt, low_health, empty, kill, death, victory. **`sounds/SCRIPT.md`** is the recording script for the actor (lines, file names, specs).
- **N** = sound on/off (saved in `localStorage`). `serve.js` now serves audio MIME types and supports HTTP Range requests (so voice clips can restart).
- Verified by stepping frames in the browser: title, gameplay subtitles (with a temporary test clip), the death screen, and the victory screen reached by killing the boss. Every SFX plays without errors, and R skips the title. **Not verified by ear.** The browser pane can't play audio to me, so the sound mix needs a listen in a real browser.

### Play-test round 3 changes ✅
- Death headline is now **"YOU'VE BEEN CLOSED."**
- **Shadow IT slowed** from 0.065 to 0.042 tiles/frame, below the player's 0.055, so you can outrun them.
- **Override switch unlocks the boss door.** New wall `X` (type `SWITCH` = 8) at (9,19): the back wall of the C-shaped rack in the top middle of the Server Room, facing east into the open side. It looks like a server rack. On its east face, `drawSwitchColumn()` draws a steel panel with a blinking red light and a yellow lever up, which turn into a green light and green lever down when flipped. The minimap shows it orange, then green.
  - `doorAhead()` also returns the switch, but only when the player is east of it (inside the C). Prompt: "[ E ] FLIP OVERRIDE SWITCH", then "OVERRIDE ENGAGED".
  - **E** → `flipSwitch()`: sets `game.switchOn`, shows "OVERRIDE ENGAGED — CORE ACCESS UNLOCKED", plays the `lever` SFX (clunk, hum, two beeps) and the `switch` voice line.
  - `doorLocked(DOOR)` is now `!game.switchOn || game.bossTriggered`. Before the switch, the boss door says "LOCKED — FIND THE OVERRIDE SWITCH IN THE SERVER RACKS" with the `locked_core` voice line. Its lock light stays red until the switch is flipped.
  - Order of play: IT Closet (cannon) → security doors → Server Room fight → switch → boss door → arena.

### Freedoom sound effects + Shadow IT loot ✅
- The user wanted Doom's sounds. Original Doom audio is copyrighted (id/ZeniMax), so we use **Freedoom's** BSD-licensed replacements, which are made to sound like Doom. 22 WAVs from github.com/freedoom/freedoom `sounds/` live in **`sounds/sfx/`**, next to `FREEDOOM-LICENSE.adoc` (keep it with the game) and `CREDITS.md` (which sound plays where). The title screen shows a credit line.
- **`SFX_FILES`** maps game sound names to files:
  - Keyboard → shotgun; Closing Cannon → rocket launcher + barrel explosion.
  - Printer, Router and Shadow IT → imp sight/active/pain/death, fireball for ranged attacks, claw for melee.
  - The DNS → cacodemon sight/active/pain/death, fireball volley, with a new `nx_hit` sound on NXDOMAIN impacts.
  - Doors, locked "no way", switch and pickups use their Freedoom sounds too.
  - Unmapped or missing sounds fall back to the synthesized `SFX` versions. The title, stamp, typing, phase alarm, hurt and UPS beep are still synthesized.
- Loading (`loadSfxFiles()`, on the first `ensureAudio()`): over http the files are fetched and decoded into AudioBuffers, so positional volume, stereo pan and a ±6% Doom-style pitch variation all work. When `index.html` is opened as `file://`, fetch is blocked, so it falls back to cloned `<audio>` elements: volume only, no panning. `SFX_FILE_VOL` sets the overall level.
- **Shadow IT** (boss adds) now always drops **coffee (+20 SLA) and closures (+3)**, placed side by side across your line of sight. This replaces the old 65% tickets / 35% closures roll.
- Verified: all 22 files decode and play through `sfx()` with no errors, and drops spawn correctly. **Not checked by ear.**

### Fullscreen + window scaling ✅
- The canvas still renders at 800×500. CSS scales it with `image-rendering: pixelated`, keeping the 8:5 shape. In a normal window it fills the width or height, whichever runs out first, leaving room for the controls line: `width: min(100vw, (100vh − 26px) × 1.6)`.
- **Fullscreen:** **Alt+Enter** (works on the title screen without starting the game) or the "⛶ Fullscreen" button in the controls line → `toggleFullscreen()`, which makes the canvas itself fullscreen. `canvas:fullscreen` uses `object-fit: contain`, so widescreen gets black bars. Esc exits. Mouse capture works the same way inside fullscreen. The title screen lists ALT+ENTER.
- The Claude browser pane refuses fullscreen requests, just as it refuses pointer lock. Verified there: scaling at 789×666 and at 1600×800, and both triggers call `requestFullscreen`. The fullscreen view itself is untested and needs a real browser.

---

## Running / testing
- **Live:** https://johnentech.github.io/entech-the-closer/ (GitHub Pages, deployed from `main` / root of https://github.com/johnentech/entech-the-closer). Every push to `main` redeploys within a minute or two, including new voice files in `sounds/voice/`.
- `node serve.js` → http://localhost:5180 (zero-dependency static server), or just open `index.html` directly.
- In the Claude desktop browser pane, `requestAnimationFrame` runs sporadically; for automated checks, stub it out (`window.requestAnimationFrame = () => 0`) and step the game by calling `update(); render();` from JS.
- If the session was started in the Solitaire folder, `preview_start` reads Solitaire's launch config — run `node serve.js` in the background instead of touching Solitaire files.

---

### Chunk 7 — Ammo UX + KEYCAPS rename ✅
The user cancelled "more ammo before the boss" and "timed boss-fight resupply". Shadow IT's coffee + closures drops stay.
- **Keyboard ammo renamed TICKETS → KEYCAPS**: HUD label, `+8 / +4 KEYCAPS` pickup and drop messages, pickup kind `keycaps` (level char `t`), and `TEX.keycaps`, now drawn as a pile of keycaps with an orange Esc key. **"TICKETS CLOSED" (the kill counter) is unchanged**, and so are `cannon_1` and `kill_1`, which are about closing tickets. Older sections of this file still say "tickets" when they mean keyboard ammo.
- **Voice:** `ammo_1–3` and `empty_1–2` reworded for keycaps. New `empty_cannon_1` ("Out of closures!") and `closures_1` ("Closures. Let's close some tickets."); closures pickups now use the `closures` event, not `ammo`. `sounds/SCRIPT.md` is updated to match.
- **`onEmpty(dryFire)`** runs when the last round leaves the weapon (from `shoot()`) or on a dry-fire. It shows a flashing center alert under the crosshair (`EMPTY_ALERT_MS` 1.8 s: hard blink for 0.6 s, then steady, then fade):
  - other weapon has ammo → **"KEYCAPS EMPTY — SWITCHING TO CANNON"** / **"CLOSURES EMPTY — SWITCHING TO KEYBOARD"**, plus `switchWeapon(other)` (the swap starts once the current shot finishes cooling)
  - both empty (cannon owned) → **"NO AMMO"**; no cannon yet → **"OUT OF KEYCAPS"**
  - Also plays a dry click (dry-fire only) and the weapon's empty voice line (`WEAPONS[].voiceEmpty`). `hasAmmo(name)` is the helper.
- **Ammo counter** (bottom right): label and number blink red while the current weapon is at 0.
- **HUD weapon slots** replace the old weapon label and `[1] keyboard 2 cannon` hint. Two boxes in the center of the bar, each showing a key badge (1 / 2), the name (`WEAPONS[].slot`) and the ammo count:
  - active: green frame and fill (the frame blinks red if it's empty)
  - empty: red name and 0
  - not picked up yet: dark `- - -` / `--`
- `WEAPONS` gained `slot`, `short`, `key` and `voiceEmpty`, and lost `label`.
- Verified in the browser: keycaps sprite, slots, keycaps→cannon auto-switch, closures→keyboard auto-switch, NO AMMO, OUT OF KEYCAPS without the cannon, and no console errors.

**Balance to watch:** two awake enemies take ~60 HP in a few seconds if you stand still. Coffee pickups offset this; tune `INK_DMG`, router burst damage or coffee count after a real play-through.

---

### Chunk 8 — Map textures ✅
All procedural (64×64 canvases drawn once with `makeTex`, seeded by `rng()` + `speckle()` so they look the same on every load). No image files. Lives in the `// --- Map textures` section just above `render()`.
- **Walls** (`MAP_TEX`, `rackTex(frame)`, `WALL_TEX`):
  - concrete = cinder blocks
  - cubicle = woven fabric with a top rail, kick plate and aluminum edges
  - server rack = 1U units with vents, drive bays and green/amber LEDs
  - shelving = boxes and binders on olive shelves
  - arena = purple panels with glowing circuit traces
  - boss door = ribbed red steel
  - security door = brushed steel with a wired-glass window
  - override switch = rack texture with its panel drawn on top
- **How walls draw:**
  - `WALL_TEX[type]` is a list of animation frames, each `[light face, dark face]`; the dark copy is a `tintTex` at `WALL_FACE_DARK`.
  - The renderer `drawImage`s one texture column per screen column (`rayWallX`), stretched to the wall height.
  - Racks have 2 frames, swapped every `RACK_BLINK_MS`, offset per cell (`castRay` now also sets `rayMapX/rayMapY`), so the LEDs blink out of sync.
  - Distance fog is a quantized black overlay per column (`FOG_RATE`, `FOG_MAX`, `FOG_STYLES`), drawn *before* the door / switch overlays so lock lights stay bright.
  - `WALL_COLORS` is removed; the minimap still uses `MINIMAP_COLORS`.
- **Floors** (`FLOOR_SRC` → `FLOOR_TEX` pixel arrays, `FLOOR_MAP` per cell):
  - textures: 0 office carpet tiles, 1 Server Room raised floor with perforated tiles, 2 arena purple grid, 3 IT closet concrete
  - region rules: `ARENA` mask → arena; `y ≥ 16 && x ≤ 25` → Server Room (incl. its doors and the boss door cell); `x ≥ 29 && y ≤ 9` → closet; else carpet
- **Ceiling** (added after Chunk 8, `CEIL_SRC` → `CEIL_TEX`, `CEIL_MAP` per cell):
  - textures: 0 office drop-ceiling tiles (also the closet), 1 the same tiles with a fluorescent light panel, 2 Server Room dark slab with a ladder cable tray of muted network cables (runs along x), 3 arena near-black with a faint purple grid
  - region rules follow `FLOOR_MAP`; office/hallway cells get a light panel when `x % 4 === 2 && y % 3 === 1`
- **`drawFloorAndCeiling()`** does classic floor casting into one full-screen `Uint32Array` / `ImageData` (`flatCanvas`). Each floor row's mirror above the horizon is the same distance, so one pass writes both. It `drawImage`s the result, so shake and the death slump still apply. Gradients (`floorFog`, `ceilFog`) darken both toward the horizon, and the flat `#1a1a2e` / `#282018` fills remain as a backdrop past the screen edges.
- **Performance** (Claude browser pane): full frame ≈ 7.8–9 ms with floor + ceiling, inside the 16.7 ms budget for 60 fps. If slower machines struggle, cast the floor and ceiling at half resolution.
- Verified visually: office, Server Room, IT Closet, arena, boss door, security door, override switch; no console errors.

---

### Printer redesign ✅
The Printer read as a white block with red eyes, so `drawPrinter` / `drawPrinterDead` were redrawn as an office multifunction printer (same 48×44 sprite, same scale and attack look):
- **Top:** paper fanned in the document feeder, the feeder unit, and a dark scanner lid overhanging the body. The lid's edge is the brow over slanted red LED eyes.
- **Front:** a control panel (status screen goes red on windup, plus start/stop buttons), an ENTECH-blue stripe, and an output slot with a printed sheet sticking out. The slot opens into paper teeth around an ink blob when it attacks.
- **Bottom:** two paper drawers with handles and a paper-level window, on casters.
- **Dead:** squashed body under its slumped lid, ink puddle, scattered paper.

---

### Weapon bob ✅
Doom-style walking sway. In `drawWeapon()` the weapon is translated by `cos(a) × BOB_X` sideways and `|sin(a)| × BOB_Y` down, a U-shaped arc that dips mid-swing, with `a` cycling every `BOB_CYCLE_MS`. Defaults: 34 px, 14 px, 1000 ms. Strength is `weapon.bobAmp`, which eases (`BOB_EASE`) toward how far the player actually moved this frame ÷ `MOVE_SPEED`, so it builds in ~0.5 s, settles when you stop, and stays still when walking into a wall. The small idle breathing bob is unchanged.

---

### Printer attack: glowing test page ✅
The Printer's shot (projectile kind `ink`, same speed, damage and hit radius as before) now looks like a **Windows-style printer test page**. `testPageFrames()` draws a 32×42 sheet:
- blue icon square with a tiny printer, top right
- blue title, rule and subtitle line
- three bold section headers with two-column label/value rows and file-path lines
- blank space at the bottom

It returns 8 flutter frames (front → squashed → edge-on → back with faint mirrored print → … ), cycled every `PAGE_FLIP_MS` with a random `p.spin` offset per shot so pages tumble out of sync.
- `drawSprites` draws a pulsing additive red-magenta `TEX.pageGlow` behind each page (`PAGE_GLOW_SCALE`), then the page (`PAGE_SCALE`, `PAGE_Z`).
- Trail: `pageTrail` glow dots, plus a `paperScrap` that drifts down every 10 frames.
- Wall impact: `paperHit` burst + magenta flash.
- The Printer's wind-up glow and muzzle flash (`pageFlash`), the blob in its attack mouth, and the minimap dot all switched from purple to red-magenta to match. The old `ink*` textures are gone; the projectile kind and sounds keep the `ink` name.

---

### Title screen art + music ✅
The user supplied title art: `art/title.webp` (1448×1086), with the character in a suit and a Doom-style "THE CLOSER" logo. It replaces the old text title (the controls list and terminal teaser are gone; the controls stay in the HTML line under the canvas).
- **Stages** (`title.stage`), reworked after play-testing (the old black "press any key" gate made you wait in the dark for the art):
  - `loading`: black until the art *and* its effects are prepared (`titleArt.ready` is set only after `prepTitleFx()`).
  - `art`: the art fades in (`ART_FADE_MS`) with all its animations. "PRESS ANY KEY" appears at `PROMPT_DELAY_MS` (~1.5 s after it's fully visible); keys do nothing before that. That press is also the gesture that lets the browser start sound.
  - `intro`: the logo punches in from 1.7× and slams into place (`SLAM_MS` 260) with shake, flash, `slam` + `explosion` SFX and the riff's first chord; "E N T E C H :" fades in above it. Keys are ignored.
  - `ready` (at `INTRO_READY_MS`, ~2 s after the slam): "CLICK OR PRESS ANY KEY TO START" blinks in the bottom band; a click also grabs the mouse.
  - `advanceTitle()` handles every key/click; `updateTitle()` runs the timeline. R-restart still skips the title (`startGame(true)`).
- **Layout:** the art fills the top 470 px (`ART_SCALE`, centered at `ART_X`); a blurred, darkened copy fills the side bars (`titleArt.sides`). The whole art slowly pushes in 4% and back over 40 s, around `ART_ANCHOR`. `render()` returns right after `drawTitle()` while on the title (no raycasting).
- **Effects** (`drawTitleFx`, in art pixels): `prepTitleFx()` cuts color-masked layers from the art at load, using feature boxes found from its colors (`ART_LIGHTS`, `ART_STRIPE`, `ART_MONITORS`, `ART_RACKS`, `ART_LOGO`):
  - the logo (saturation/navy mask, closed with a radius-2 dilate+erode), used for the punch-in and the glint
  - ceiling lights that hum; light `ART_DYING` stutters off every ~5 s (`lightsOff` dark copies)
  - the red back-wall stripe pulsing via a red-tinted copy (`stripeGlow`)
  - flickering monitor screens
  - 44 rack-LED clusters that randomly go dark or flare
  - a glint band sweeping through the logo's own pixels every `GLINT_EVERY_MS`
  - **file:// fallback:** reading pixels from a file:// image is blocked (tainted canvas), so effects are skipped and the whole art punches in instead of just the logo.
- **Music** (`// --- Music` in the audio section): synthesized, no files, all original compositions (Doom's E1M1 etc. are copyrighted). Each track is a list of bars built with `mk(guitar, kick, snare, hat, crash, lead, lead2, toms)`.
  - **Bars** are 16 sixteenth-notes. Guitar tokens: `x` = palm-muted low-E chug, `~A2` = muted chug on another note, `G2` etc. = power chord, `-` = hold, `.` = rest. `lead`/`lead2` take note names (any octave, parsed by `noteFreq()`). Drum lines use `x` for hits; toms use `h`/`m`/`l`.
  - `title`: the Doom-style riff at 168 BPM. It opens with four slammed E chords (`intro: 2` bars play once), then loops a 4-bar gallop riff.
  - `game` (reworked after play-test: the first version was too slow and mundane): a 16-bar song at 160 BPM, brighter tone. E riff with chromatic stabs → the riff with a lead melody → A gallop with lead → the riff ending on a tom fill.
  - `boss` (reworked: it needed more punch): 16 bars at 176 BPM. Double-kick tremolo riff → chord hits → the riff with a screaming lead → half-time breakdown → snare-roll build → the riff with the lead harmonized a fourth below. It plays while the DNS is alive and cuts to silence when it dies, so SERVFAIL's explosions play alone.
  - Instruments: `guitar()` is detuned saws → waveshaper distortion → lowpass. `leadNote()` is saw + square, lightly driven, with vibrato that fades in on held notes and a send to a shared 0.28 s echo (`music.echo`). `bassNote()`. Punchier drums: the kick has a beater click, the snare a body tone and snap, plus pitched toms.
  - `scheduleMusic()` is a 0.25 s look-ahead scheduler on a 40 ms timer. `syncMusic()` (called from `update()`) picks the track: title / game / boss / silence on death, victory and SERVFAIL.
  - The bus is ducked under voice lines (`music.duck`) and muted by **N**. Volumes are in `MUSIC_VOL` (title 0.5, game 0.2, boss 0.16).
  - Offline-render check (full 16 bars): game peak 0.32 / RMS 0.071, boss peak 0.30 / RMS 0.080, title peak 0.67; no clipping. **Not heard by me**, so the mix needs a real listen.
- `serve.js` serves `.webp` as `image/webp`.

---

### Shadow IT tuning, Flush purge, ammo pickup sound ✅
- **Shadow IT** (`ENEMY_TYPES.shadow`): HP 300 → **270**, so it takes exactly 3 keyboard blasts point-blank out to ~4 tiles (verified 300/300 trials; 5–6 blasts from 7 tiles because of falloff). **New `splashMult: 2.4`** (applied in `explode()` for any type that sets it), so Shadow IT is weak to the Closing Cannon: a direct hit kills it, and two near misses within ~1.3 tiles do. Spawn timing is unchanged (`BOSS_PHASES`: phase 2 every 9 s, max 2; phase 3 every 5 s, max 3; first one 1.2 s after the phase starts).
- **Flush DNS Cache** also purges every living Shadow IT. Each one dies through a normal `damageEnemy` kill (death sound, kill count, coffee + closures drop), with a purple burst; the message reports how many were purged.
- **Ammo pickups** (keycaps and closures) play Freedoom's shotgun-pump clunk, `sounds/sfx/dssgcock.wav` (SFX name `ammo`, with a synthesized clack-clunk fallback). Coffee keeps `dsitemup`.

---

### Weapon art: Doom-style pixel art ✅
Both first-person weapons were redrawn (`// --- First-person weapons: chunky pixel art`, just above `drawKeycaps`). Gameplay is unchanged.
- **Pipeline:** each weapon is drawn at 1/`WPX` (4) resolution and scaled up with nearest-neighbor for Doom-style chunky pixels.
  - **Built once at load:** the bodies, plus gloved hands, white shirt cuffs and dark suit sleeves matching the title art's character. `shadeTube()` writes a cylinder lit from the upper left straight into pixels, and `outlineTex()` adds a 1 px dark outline.
  - **Each frame:** moving parts go into the scratch canvas `wbuf` (`WBUF_W`×`WBUF_H`). `blitWeapon()` tints it by room (`AREA_TINT` by `FLOOR_MAP`, eased: office none, Server Room blue, arena purple, IT closet warm), adds a red wash while `game.hurt`, then scales it ×4 onto the screen.
  - The smooth radial muzzle bloom is still drawn at screen resolution behind each weapon.
- **Closing Cannon** (`CANNON_SPRITE`, 72×44, plus `CANNON_TOP` 24 rows above for the flash; dome tip at screen y `CANNON_TIP_Y` 322). It went through two play-test rounds: the first version read as aiming at the ceiling, and the forward-facing redo with a red bore was too big. The user then asked for something closer to a reference image of Doom's rocket launcher, so this is an **original drawing in that style**:
  - seen from behind: glossy dark gunmetal with a bright streak down the center, a rounded dome at the far end (`CANNON_DOME`) with a grey sight knob, and no visible bore
  - three arched ribs (`CANNON_RIBS`, each with a lit edge), then the red label band (`CANNON_BAND`), then two arched vent slots with lit lips (`CANNON_VENTS`)
  - features arch (∩) like rings around a tube seen from above: each pixel's position "along" the body is `y + CANNON_ARCH·hw·n`, the ranges above are in that unit, and `cannonPeak(a)` gives the row where an arch peaks
  - side flanges, a small upper pair and a bigger lower pair; the gloved hands grip the lower pair, with shirt cuffs and suit sleeves trailing off the bottom corners
  - the band carries **"CLOSE" / "W/O EMAIL"** in two lines (bold 13 px), drawn at full resolution after the blit (outlined) so it stays readable, with each letter set on the band's arch
  - `weapon.fireAt` starts at -1e9, so no flash, smoke or key ripple shows right after load
  - firing: a pixel starburst flash off the dome tip (`CANNON_FLASH`, 2 variants), the existing kick, then 3 smoke puffs curling up for ~0.9 s
- **Keyboard** (`KEYBOARD_BASE` + `KEYBOARD_HANDS`, 96×40):
  - a tapered mechanical-keyboard case with a cable out the back
  - `drawKeys()` draws 3D caps each frame (top face + darker front edge, legend dots), with an orange Esc and an ENTECH-blue Enter
  - firing: keys press down in a wave spreading out from the middle
  - RGB underglow cycles along the far edge and both sides
  - gloved hands grip both ends (finger segments, lit thumbs) with cuffs and sleeves
- **Cost:** under 0.5 ms per frame for either weapon (full frame ≈ 6.3 ms in testing).

---

### One Server Room door, clearer lock prompts ✅
- **Removed the starting office's security door.** Play-testers were confused by two doors into the same area. The door at (8,16) and its 3-tile hallway (cols 8–9, rows 13–15) are solid wall now, and the keycaps pickup that sat in that hallway moved into the office at (8,12), where the hallway used to open. The only Server Room door is the one at the bottom of the hallway from the IT Closet, (22,16), still locked until the Closing Cannon is picked up.
- **Lock prompts under the crosshair** (`drawHUD`), replacing the plain "LOCKED":
  - Server Room door: **"ACQUIRE CLOSING CANNON TO PROCEED"**
  - Boss door: **"RESET THE MAIN SWITCH TO UNLOCK"**, or "LOCKED" once it has re-sealed behind you for the fight
- **The override switch is now the "main switch"** in all on-screen text: "[ E ] RESET MAIN SWITCH" → "MAIN SWITCH RESET", "MAIN SWITCH RESET — CORE ACCESS UNLOCKED", and "LOCKED — FIND THE MAIN SWITCH IN THE SERVER RACKS". Older sections of this file still say "override switch". The voice subtitles `locked_core_1` / `switch_1` keep "override", since they must match what was recorded; change them together with the audio if the lines are re-recorded.

---

## Remaining chunks

### Next session (user's list, 2026-10-29). Not started; discuss before coding
1. **Make pickups more noticeable.** Ideas: a gentle bob/hover, a soft glow or halo under each pickup (color by kind), an occasional sparkle, slightly bigger sprites, maybe dots on the minimap. **Keep the keycaps and coffee sprites as they are.**
2. **Redesign the Closing Cannon pickups to look better.** Both the closures ammo pickup (`TEX.closures`) and the cannon weapon pickup (`TEX.cannon`) should match the new launcher art (dark gunmetal, dome, ribs, red CLOSE W/O EMAIL band).
3. **Weak melee fallback weapon** for when the player has no ammo. To decide: what it is (gloved punch, stapler, coffee mug, mouse on a cord, …), its key (3?), damage and range, whether it auto-equips when both weapons are empty (`onEmpty` currently shows "NO AMMO"), and how it fits the HUD weapon slots.
4. **Secret easter-egg room**, unlocked by the player doing something. To decide: where it is (a hidden wall that slides open like Doom's secrets, behind a poster, in the office or the IT Closet), what unlocks it (pressing E on a suspicious wall, shooting something, typing a code like a cheat, flushing DNS in a certain spot, …), and what's inside (credits for the team / voice actor, bonus supplies, an IT joke), plus a "SECRET FOUND" message and sound.

Still open from before: process the voice actor's clips (ffmpeg pipeline or Audacity; written OK received).

---

## Key code structure in index.html

```
LEVEL[]          — map as strings → MAP[][] (40×30), LEVEL_SPAWNS, PLAYER_START, BOSS_DOOR, BOSS_SPAWN
player{}         — x, y, dirX, dirY, planeX, planeY
game{}           — health, ammo, cannonAmmo, hasCannon, weapon, kills, over, won, bossTriggered, boss, flushes, msg…
WEAPONS{}        — per-weapon slot / short / key / ammo keys / COOL_MS / KICK / voiceEmpty
weapon{}         — anim state: idle | firing | cooling | lowering | raising; next, kickY, flash
zBuf[]           — Float64Array(800), perpWallDist per column

enemies[] (incl. the DNS) / projectiles[] (ink | rocket | nx) / explosions[] / pickups[] / effects[]
ENEMY_TYPES{}    — per-type stats; BOSS_PHASES{}; PICKUPS{} — per-kind tex/scale/apply; TEX{} — sprite textures
MAP_TEX{} / WALL_TEX{} / FLOOR_TEX[] / FLOOR_MAP / CEIL_TEX[] / CEIL_MAP — wall, floor, ceiling textures; drawFloorAndCeiling() — floor/ceiling casting

shoot()          — keyboard: 7 hitscan pellets → damageEnemy(); cannon: spawns rocket
explode()        — splash damage (enemies + player self-damage)
castRay()        — shared DDA; hasLOS(); tryMove() — shared collision
doorAhead() / doorLocked() / useAction() / checkBossTrigger() / startBossFight()
updateBoss() / fireVolley() / bossDie() / useFlush()
switchWeapon() / cycleWeapon()

audio{} / SFX{} / sfx() — synthesized effects (tone(), noise()); VOICE{} / say() — recorded voice lines + subtitles
startGame() / restart() / canRestart() — title → play → death/victory → R

update()         — title pan | death stamp | victory typing → player → weapon anim → updateEnemies() → updateProjectiles() → updatePickups() → checkBossTrigger()
render()         — [shake + death slump] flat backdrop → drawFloorAndCeiling() → textured raycaster (+fog, drawDoorColumn / drawSwitchColumn) → drawSprites() → (title: drawTitle()) → drawWeapon() → drawKeycaps() [/shake] → drawScreenFlash() → drawDeathOverlay() | drawVictoryOverlay() | (drawGlitch() → drawCrosshair() → drawDamageDirs() → drawHUD() → drawBossBar() → drawMinimap() → drawServfail()) → drawSubtitles()
```

---

## Design decisions already locked in

- Single HTML file (no build step, no libraries)
- 800×500 canvas
- FOV plane = 0.66 (~66°)
- MOVE_SPEED = 0.055, ROT_SPEED = 0.038, COLLIDE margin = 0.22
- Keyboard: 7 pellets, SPREAD = 0.09, cooldown 650ms
- Closing Cannon: rocket 0.16 tiles/frame, splash 140 over 2.2 tiles, 40% self-damage, cooldown 950ms
- HUD labels: health = "SLA", keyboard ammo = "KEYCAPS", cannon ammo = "CLOSURES", kills = "TICKETS CLOSED"
- Boss death line: "SERVFAIL"
