# DNS: The Reckoning — Session Handoff

## What this is
IT-themed Doom clone built in a single HTML file using canvas raycasting (no libraries).
Working title: **DNS: The Reckoning**. Player is an IT tech who fights office equipment and ultimately battles the DNS boss.

**File:** `C:\Users\JohnMiller\Documents\Solitaire Project\doom-clone.html`

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

### Chunk 3 — Enemies
Sprite renderer (billboard sprites sorted by distance using `zBuf[]`), then three enemy types:

| Enemy | Based on | Behavior |
|---|---|---|
| **Printer** | Doom Imp | Lobs slow ink-blob projectiles at range; claws at close range; medium HP |
| **Rogue Router** | Shotgun Zombie | Fires spread burst; slow shuffle; low HP |
| **Shadow IT** | Pinky | Spawned by DNS boss only; fast charge; melee only; tanky |

Enemy AI pattern: line-of-sight detection → aggro → pathfind toward player → attack timer.
Enemies need to hook into the existing `shoot()` hitscan (7 pellets, each raycast with spread `SPREAD = 0.09`).

### Chunk 4 — Level
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
- HUD flavor: kill counter labeled "TICKETS CLOSED"

---

## Key code structure in index.html

```
MAP[][]          — 2D grid (21×24)
player{}         — x, y, dirX, dirY, planeX, planeY
game{}           — health, maxHealth, ammo, maxAmmo
weapon{}         — state, fireAt, FIRE_MS, COOL_MS, kickY, flash
zBuf[]           — Float64Array(800), perpWallDist per column

shoot()          — fires 7 hitscan pellets; enemy damage hooks go here
isWall(x,y)      — bounds+map check
rotate(angle)    — rotates player.dir and player.plane vectors

render()         — ceiling → floor → raycaster → drawWeapon() → drawHUD() → drawMinimap()
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
