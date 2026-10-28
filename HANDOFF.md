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
- Three wall types: `1` concrete, `2` cubicle dividers (tan), `3` server racks (dark blue)
- Dark navy ceiling, dark brown carpet floor
- Player movement: W/S move, A/D strafe, ←/→ turn, mouse look via pointer lock
- Slide-based collision with margin
- Mini-map overlay (toggle M)
- Z-buffer array `zBuf[]` already in place for sprite depth sorting

### Chunk 2 — Combat Foundation ✅
- `game` object: `{ health: 100, maxHealth: 100, ammo: 25, maxAmmo: 25 }`
- `weapon` object with states: `idle | firing | cooling`
- `shoot()` function: deducts ammo, fires 7 hitscan pellets with spread (stubs — no enemies yet)
- Keyboard weapon sprite drawn with canvas rects (key rows, spacebar, 3D key bevel)
- Weapon kick animation (springs down on fire, eases back) + radial muzzle flash glow
- HUD: SLA health bar (green→yellow→red), TICKETS ammo counter, weapon name label
- Fire: left-click (after pointer lock) or Space

### Chunk 3 — Enemies ✅
- **Turning fix:** ←/→ and mouse look were inverted (left turned right). Fixed in `update()`.
- `castRay(ox, oy, dx, dy)` — shared DDA (render, hitscan, LOS). Sets globals `rayHit`, `raySide`.
- `hasLOS(ax, ay, bx, by)`, `tryMove(obj, mx, my, r)` — shared slide collision (player + enemies)
- **Sprite renderer** `drawSprites()` — billboards sorted far→near, per-column clip against `zBuf[]`. Sprite entry: `{x, y, tex, scale (fraction of wall height), z (bottom height above floor)}`
- **Procedural textures** in `TEX` — drawn once into offscreen canvases (`makeTex`, `tintTex`). Each enemy has `walk0 / walk1 / attack / pain / dead` frames.
- **Enemies** (`ENEMY_TYPES`, `spawnEnemy(type, x, y)`):

| Type | HP | Speed | Attack |
|---|---|---|---|
| `printer` | 60 | 0.022 | Ink blob projectile (10 dmg, speed 0.07) at ≤9 tiles; paper-cut melee 12 at ≤1.2 |
| `router` | 30 | 0.014 | 3-pellet hitscan burst, 3–5 each, accuracy falls off with range |
| `shadow` | 150 | 0.065 | Melee 15 at ≤1.15, fast cooldown. **Not placed in the map — boss spawns it via `spawnEnemy('shadow', x, y)`** |

- **AI states:** `idle → chase → windup → (attack) → chase`, plus `pain` (interrupts windup) and `dead`. Wake on sight (LOS within `sight` range), on being shot, or on hearing gunfire (`alertEnemies`: <5 tiles, or <10 with LOS).
- **Pathing:** BFS flow field from the player's cell (`updateFlow`, rebuilt only when the player changes cell, 8-way with no corner cutting). Enemies with LOS walk straight at you; otherwise they step downhill on the field.
- **Hitscan:** `shoot()` now casts 7 rotated rays; each hits the nearest enemy (ray vs circle `def.radius`) before the wall. `PELLET_DMG = 9` (63 point blank → one-shots a Printer). Damage is summed per enemy per shot so pain is rolled once.
- **Drops:** 60% chance a kill drops a ticket stack (+4 ammo), picked up by walking over it (green flash).
- **Player damage:** `hurtPlayer(dmg)` + red flash. At 0 HP: `game.over`, placeholder "YOUR TICKET HAS BEEN CLOSED / Press R" overlay (real death screen is Chunk 6).
- **Pulled forward from Chunk 6:** `game.kills` + "TICKETS CLOSED" counter (top right).
- Minimap shows enemies (dim = idle, bright = alerted, dark = dead) and ink blobs.
- Current placements (temporary until Chunk 4 map): 4 Printers in the cubicle farm, 4 Routers in the server room.

**Balance note:** no health pickups exist yet, and two awake enemies will take ~60 HP in a few seconds if you stand still. Add health pickups in Chunk 4 (coffee?) or tune `INK_DMG` / router burst damage.

---

## Running / testing
- `node serve.js` → http://localhost:5180 (zero-dependency static server), or just open `index.html` directly.
- In the Claude desktop browser pane, `requestAnimationFrame` runs sporadically; for automated checks, step the game by calling `update(); render();` from JS.

---

## Map layout

```
const MAP = 21 rows × 24 cols, values:
  0 = walkable floor
  1 = concrete outer wall
  2 = cubicle divider (tan)
  3 = server rack (dark blue)
```

Player starts at `x:2.5, y:1.5` facing right (`dirX:1, dirY:0`).

The map has two zones divided by a wall with two gaps (row 10):
- **Top half (rows 0–9):** cubicle farm — rows of `2`-walled cubicle pairs
- **Bottom half (rows 11–20):** server room — `3`-walled server rack clusters

---

## Remaining chunks

### Chunk 4 — Level (NEXT)
Full level map layout: **Office → Hallways → IT Closet → Server Room → Boss Door**

- Closing Cannon pickup in IT Closet (functions like rocket launcher: slow projectile, splash damage, self-damage possible)
- Trigger at boss door to start Chunk 5
- Keyboard weapon renamed `[ KEYBOARD v1.0 ]` (already labeled in HUD)
- Closing Cannon should label as `[ CLOSING CANNON ]` in HUD

### Chunk 5 — Boss: The DNS
- Large floating entity in an open arena (final room)
- **Phase 1 (100–50% HP):** Fires `NXDOMAIN` projectiles (slow, curving)
- **Phase 2 (<50% HP):** Starts spawning Shadow IT adds; arena gets chaotic
- **Phase 3 (<25% HP):** `TTL 0` mode — faster attacks, faster Shadow IT spawns, screen flicker effect
- `Flush DNS Cache` power-up somewhere in the level — briefly stuns boss
- Death line: *"SERVFAIL"*
- Win condition triggers Chunk 6 victory screen

### Chunk 6 — Polish
- Title screen (press any key to start)
- Death screen (you got a ticket closed on you)
- Victory screen ("Connection established.")
- Sound stubs (AudioContext beeps/noise for shots, enemy hits, death)
- ~~HUD flavor: kill counter labeled "TICKETS CLOSED"~~ (done in Chunk 3)

---

## Key code structure in index.html

```
MAP[][]          — 2D grid (21×24)
player{}         — x, y, dirX, dirY, planeX, planeY
game{}           — health, maxHealth, ammo, maxAmmo
weapon{}         — state, fireAt, FIRE_MS, COOL_MS, kickY, flash
zBuf[]           — Float64Array(800), perpWallDist per column

enemies[] / projectiles[] / pickups[]
ENEMY_TYPES{}    — per-type stats; TEX{} — procedural sprite textures

shoot()          — 7 hitscan pellets → damageEnemy()
castRay()        — shared DDA; hasLOS(); tryMove() — shared collision
isWall(x,y)      — bounds+map check
rotate(angle)    — rotates player.dir and player.plane vectors (positive = turn right)

update()         — player → weapon anim → updateEnemies() → updateProjectiles() → updatePickups()
render()         — ceiling → floor → raycaster → drawSprites() → drawWeapon() → drawScreenFlash() → drawHUD() → drawMinimap() → drawDeathOverlay()
drawWeapon()     — keyboard sprite + muzzle flash glow
drawHUD()        — SLA bar + TICKETS counter + weapon label
drawMinimap()    — top-left overlay, toggle with M
```

---

## Design decisions already locked in

- Single HTML file (no build step, no libraries)
- 800×500 canvas
- FOV plane = 0.66 (~66°)
- MOVE_SPEED = 0.055, ROT_SPEED = 0.038, COLLIDE margin = 0.22
- Keyboard: 7 pellets, SPREAD = 0.09, cooldown 650ms
- Closing Cannon: slow projectile, splash (to implement)
- Wall color arrays: `WALL_COLORS = { 1: concrete, 2: cubicle, 3: server rack }`
- HUD labels: health = "SLA", ammo = "TICKETS", kills = "TICKETS CLOSED"
- Boss death line: "SERVFAIL"
