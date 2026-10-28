# DNS: The Reckoning — Session Handoff

## What this is
IT-themed Doom clone built in a single HTML file using canvas raycasting (no libraries).
Working title: **DNS: The Reckoning**. Player is an IT tech who fights office equipment and ultimately battles the DNS boss.

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

---

## Running / testing
- `node serve.js` → http://localhost:5180 (zero-dependency static server), or just open `index.html` directly.
- In the Claude desktop browser pane, `requestAnimationFrame` runs sporadically; for automated checks, stub it out (`window.requestAnimationFrame = () => 0`) and step the game by calling `update(); render();` from JS.
- If the session was started in the Solitaire folder, `preview_start` reads Solitaire's launch config — run `node serve.js` in the background instead of touching Solitaire files.

---

## Remaining chunks

### Chunk 6 — Polish (NEXT)
- Title screen (press any key to start)
- Death screen (you got a ticket closed on you) — replace `drawDeathOverlay()`
- Victory screen ("Connection established.") — replace `drawVictoryOverlay()`
- Sound stubs (AudioContext beeps/noise for shots, enemy hits, explosions, death)
- ~~HUD flavor: kill counter labeled "TICKETS CLOSED"~~ (done in Chunk 3)

### Chunk 7 — Balance & ammo UX (after Chunk 6)
From play-test round 2 (user feedback after Chunk 5):
- **More ammo before the boss.** Players reach the arena short on ammo. Raise drop rates/amounts from regular enemies (currently 60% chance of +4 tickets) and/or place more ticket/closure pickups along the route (especially the Server Room).
- **Boss fight resupply from phase 2.** Once the DNS enters phase 2, periodically spawn HP (coffee) and ammo (tickets/closures) pickups in the arena — e.g. on a timer at random `ARENA_CELLS` away from the boss, with a spawn effect and cap on how many are on the floor. (Shadow IT add drops alone aren't enough.)
- **Make "out of ammo" obvious.** Right now firing on empty silently does nothing. Add a clear on-screen alert (flashing "OUT OF TICKETS" / "OUT OF CLOSURES", red ammo counter pulse, maybe a dry-fire click once sound exists).
- **Auto-switch weapon when empty**, with an on-screen alert (e.g. "CLOSURES EMPTY — SWITCHING TO KEYBOARD"). Only switch if the other weapon has ammo; if both are empty, show "NO AMMO" instead.
- **HUD weapon-switch hints.** Make it obvious you can switch: show both weapon slots with key labels (1 / 2) and their ammo counts, highlight the active one, and dim/strike a slot that's empty or not owned. (Current hint is a faint `[1] keyboard 2 cannon` line under the weapon label.)

**Balance to watch:** two awake enemies take ~60 HP in a few seconds if you stand still. Coffee pickups now offset this; tune `INK_DMG`, router burst damage or coffee count after a real play-through.

---

## Key code structure in index.html

```
LEVEL[]          — map as strings → MAP[][] (40×30), LEVEL_SPAWNS, PLAYER_START, BOSS_DOOR, BOSS_SPAWN
player{}         — x, y, dirX, dirY, planeX, planeY
game{}           — health, ammo, cannonAmmo, hasCannon, weapon, kills, over, won, bossTriggered, boss, flushes, msg…
WEAPONS{}        — per-weapon label / ammo keys / COOL_MS / KICK
weapon{}         — anim state: idle | firing | cooling | lowering | raising; next, kickY, flash
zBuf[]           — Float64Array(800), perpWallDist per column

enemies[] (incl. the DNS) / projectiles[] (ink | rocket | nx) / explosions[] / pickups[] / effects[]
ENEMY_TYPES{}    — per-type stats; BOSS_PHASES{}; PICKUPS{} — per-kind tex/scale/apply; TEX{} — procedural textures

shoot()          — keyboard: 7 hitscan pellets → damageEnemy(); cannon: spawns rocket
explode()        — splash damage (enemies + player self-damage)
castRay()        — shared DDA; hasLOS(); tryMove() — shared collision
doorAhead() / doorLocked() / useAction() / checkBossTrigger() / startBossFight()
updateBoss() / fireVolley() / bossDie() / useFlush()
switchWeapon() / cycleWeapon()

update()         — player → weapon anim → updateEnemies() → updateProjectiles() → updatePickups() → checkBossTrigger()
render()         — [shake] ceiling → floor → raycaster (+drawDoorColumn) → drawSprites() → drawWeapon() → drawKeycaps() [/shake] → drawScreenFlash() → drawGlitch() → drawCrosshair() → drawDamageDirs() → drawHUD() → drawBossBar() → drawMinimap() → drawServfail() / drawDeathOverlay() / drawVictoryOverlay()
```

---

## Design decisions already locked in

- Single HTML file (no build step, no libraries)
- 800×500 canvas
- FOV plane = 0.66 (~66°)
- MOVE_SPEED = 0.055, ROT_SPEED = 0.038, COLLIDE margin = 0.22
- Keyboard: 7 pellets, SPREAD = 0.09, cooldown 650ms
- Closing Cannon: rocket 0.16 tiles/frame, splash 140 over 2.2 tiles, 40% self-damage, cooldown 950ms
- HUD labels: health = "SLA", keyboard ammo = "TICKETS", cannon ammo = "CLOSURES", kills = "TICKETS CLOSED"
- Boss death line: "SERVFAIL"
