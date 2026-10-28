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
| `printer` | 60 | 0.022 | Ink blob projectile (10 dmg, speed 0.07) at ≤9 tiles; paper-cut melee 12 at ≤1.2 |
| `router` | 30 | 0.014 | 3-pellet hitscan burst, 3–5 each, accuracy falls off with range |
| `shadow` | 150 | 0.065 | Melee 15 at ≤1.15, fast cooldown. **Not placed in the map — boss spawns it via `spawnEnemy('shadow', x, y)`** |

- **AI states:** `idle → chase → windup → (attack) → chase`, plus `pain` (interrupts windup) and `dead`. Wake on sight (LOS within `sight` range), on being shot, or on hearing gunfire/explosions (`alertEnemies`: <5 tiles, or <10 with LOS).
- **Pathing:** BFS flow field from the player's cell (`updateFlow`, rebuilt when the player changes cell or when `flowCell = -1` is set after a map change; 8-way with no corner cutting). Enemies with LOS walk straight at you; otherwise they step downhill on the field.
- **Hitscan:** keyboard casts 7 rotated rays; each hits the nearest enemy (ray vs circle `def.radius`) before the wall. `PELLET_DMG = 9` (63 point blank → one-shots a Printer). Damage is summed per enemy per shot so pain is rolled once.
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
| `%` | IT closet shelving (5) | `C` | Closing Cannon |
| `@` | arena wall (6, DNS purple) | `o` | closures (+3 cannon ammo) |
| | | `B` | DNS boss spawn → `BOSS_SPAWN` (unused until Chunk 5) |

- **Route:** Office/cubicle farm (top-left, 3 Printers) → east hallway (Router) → **IT Closet** (top-right, shelving aisles, Printer + Router guarding the **Closing Cannon**) → hallway down (Printer) → **Server Room** (bottom-left, rack clusters, 4 Routers + Printer) → **Boss Door** (east wall of server room, `BOSS_DOOR = {x:25, y:22}`) → **Arena** (bottom-right, 13×17 open room with 4 pillar pairs, coffee + closures by the entrance). Office also connects straight down to the server room, so the closet is optional but rewarded.
- 12 enemies placed; 14 pickups (4 tickets, 5 coffee, 4 closures, 1 cannon) + 60% ticket drops (+4) from kills.
- **Weapons:** `WEAPONS` table (`label`, `ammoLabel`, ammo/max keys on `game`, `COOL_MS`, `KICK`). `game.weapon` = `'keyboard' | 'cannon'`. Switch with **1 / 2 / Q / mouse wheel**; `switchWeapon(name)` lowers then raises the weapon (`weapon.state` `lowering → raising`, `SWITCH_MS = 160` each). A switch requested mid-shot starts when cooldown ends; switching back mid-swap cancels it.
- **Closing Cannon** `[ CLOSING CANNON ]`, ammo label **CLOSURES** (max 10, pickup gives 5 and auto-switches): rocket projectile `ROCKET_SPEED = 0.16` (sub-stepped ×4 so it can't tunnel), explodes on wall or enemy contact. `explode()` deals `SPLASH_DMG = 110` with linear falloff over `SPLASH_RADIUS = 2.2`, blocked by walls (LOS). **Self-damage** at `SELF_DMG_MULT = 0.5` (≈48 if you fire into a wall in your face). Cooldown 950 ms. Explosion = growing/fading `TEX.explosion` sprite for 350 ms.
- **Pickups** generalized: `pickups[] = {kind, x, y, amount?}`; `PICKUPS[kind] = {tex, scale, apply}` — `apply` returns a message string, or `false` to leave it on the floor (full health / full ammo).
- **Boss door:** red door with frame, seams and lock light (`drawDoorColumn`, uses `rayWallX`). `[ E ] OPEN` prompt when facing it within ~1.8 tiles (`doorAhead()`); **E** opens it (cell → 0). Once the player is >1.3 tiles past it, `checkBossTrigger()` re-seals it, nudges any enemy out of the doorway, and calls **`startBossFight()` — the Chunk 5 hook** (currently just sets `game.bossTriggered` and shows "THE DOOR SEALS BEHIND YOU. RESOLVING DNS..."). Door can't be reopened after that.
- HUD: weapon label + `[1] keyboard 2 cannon` slot hint once you own the cannon; ammo label/count follow the current weapon. Top-center message line (`showMessage(text, ms)`) for pickups and events.
- Minimap shrunk to 5px cells and colored by wall type (door red, arena purple); rockets shown in orange.

---

## Running / testing
- `node serve.js` → http://localhost:5180 (zero-dependency static server), or just open `index.html` directly.
- In the Claude desktop browser pane, `requestAnimationFrame` runs sporadically; for automated checks, stub it out (`window.requestAnimationFrame = () => 0`) and step the game by calling `update(); render();` from JS.
- If the session was started in the Solitaire folder, `preview_start` reads Solitaire's launch config — run `node serve.js` in the background instead of touching Solitaire files.

---

## Remaining chunks

### Chunk 5 — Boss: The DNS (NEXT)
- Implement inside `startBossFight()`; spawn at `BOSS_SPAWN` (32.5, 20.5), arena interior is x 26–38, y 12–28
- Large floating entity in the open arena
- **Phase 1 (100–50% HP):** Fires `NXDOMAIN` projectiles (slow, curving) — add a new `projectiles` kind
- **Phase 2 (<50% HP):** Starts spawning Shadow IT adds (`spawnEnemy('shadow', x, y)`); arena gets chaotic
- **Phase 3 (<25% HP):** `TTL 0` mode — faster attacks, faster Shadow IT spawns, screen flicker effect
- `Flush DNS Cache` power-up somewhere in the level — briefly stuns boss (add a `PICKUPS` kind + map char)
- Boss must take damage from both keyboard hitscan (needs a `radius`) and cannon splash (`explode()` iterates `enemies` — either make the boss an enemy entry or add it there)
- Death line: *"SERVFAIL"*
- Win condition triggers Chunk 6 victory screen

### Chunk 6 — Polish
- Title screen (press any key to start)
- Death screen (you got a ticket closed on you) — replace `drawDeathOverlay()`
- Victory screen ("Connection established.")
- Sound stubs (AudioContext beeps/noise for shots, enemy hits, explosions, death)
- ~~HUD flavor: kill counter labeled "TICKETS CLOSED"~~ (done in Chunk 3)

**Balance to watch:** two awake enemies take ~60 HP in a few seconds if you stand still. Coffee pickups now offset this; tune `INK_DMG`, router burst damage or coffee count after a real play-through.

---

## Key code structure in index.html

```
LEVEL[]          — map as strings → MAP[][] (40×30), LEVEL_SPAWNS, PLAYER_START, BOSS_DOOR, BOSS_SPAWN
player{}         — x, y, dirX, dirY, planeX, planeY
game{}           — health, ammo, cannonAmmo, hasCannon, weapon, kills, over, bossTriggered, msg…
WEAPONS{}        — per-weapon label / ammo keys / COOL_MS / KICK
weapon{}         — anim state: idle | firing | cooling | lowering | raising; next, kickY, flash
zBuf[]           — Float64Array(800), perpWallDist per column

enemies[] / projectiles[] (ink | rocket) / explosions[] / pickups[]
ENEMY_TYPES{}    — per-type stats; PICKUPS{} — per-kind tex/scale/apply; TEX{} — procedural textures

shoot()          — keyboard: 7 hitscan pellets → damageEnemy(); cannon: spawns rocket
explode()        — splash damage (enemies + player self-damage)
castRay()        — shared DDA; hasLOS(); tryMove() — shared collision
doorAhead() / useAction() / checkBossTrigger() / startBossFight()
switchWeapon() / cycleWeapon()

update()         — player → weapon anim → updateEnemies() → updateProjectiles() → updatePickups() → checkBossTrigger()
render()         — ceiling → floor → raycaster (+drawDoorColumn) → drawSprites() → drawWeapon() → drawScreenFlash() → drawHUD() → drawMinimap() → drawDeathOverlay()
```

---

## Design decisions already locked in

- Single HTML file (no build step, no libraries)
- 800×500 canvas
- FOV plane = 0.66 (~66°)
- MOVE_SPEED = 0.055, ROT_SPEED = 0.038, COLLIDE margin = 0.22
- Keyboard: 7 pellets, SPREAD = 0.09, cooldown 650ms
- Closing Cannon: rocket 0.16 tiles/frame, splash 110 over 2.2 tiles, 50% self-damage, cooldown 950ms
- HUD labels: health = "SLA", keyboard ammo = "TICKETS", cannon ammo = "CLOSURES", kills = "TICKETS CLOSED"
- Boss death line: "SERVFAIL"
