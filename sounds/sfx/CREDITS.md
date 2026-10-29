# Sound effect credits

The `.wav` files in this folder come from **Freedoom** (https://freedoom.github.io, https://github.com/freedoom/freedoom, `sounds/` folder).
Copyright © 2001-2024 Contributors to the Freedoom project. They are used under the BSD 3-clause license, reproduced in `FREEDOOM-LICENSE.adoc`. Keep that file with the game when you share it.

## What plays where
The `SFX_FILES` table in `index.html` maps game sounds to these files.

| Game sound | File(s) |
|---|---|
| Keyboard (shotgun) | `dsshotgn` |
| Closing Cannon fire / explosion | `dsrlaunc` / `dsbarexp` |
| Printer, Router, Shadow IT: spot you / wind up / pain / die | `dsbgsit1`, `dsbgsit2` / `dsbgact` / `dspopain` / `dsbgdth1`, `dsbgdth2` |
| Enemy shots / ink splat / Shadow IT claw | `dsfirsht` / `dsfirxpl` / `dsclaw` |
| The DNS: appears / winds up / pain / NXDOMAIN volley / NXDOMAIN impact / dies | `dscacsit` / `dsdmact` / `dsdmpain` / `dsfirsht` / `dsfirxpl` / `dscacdth` |
| Door opens / locked / override switch | `dsdoropn` / `dsnoway` / `dsswtchn` |
| Ammo & coffee / Closing Cannon / Flush DNS Cache pickups | `dsitemup` / `dswpnup` / `dsgetpow` |

Any sound not listed, or whose file is missing, uses the game's built-in synthesized version.

To use a different sound, drop a `.wav` in this folder and point its entry in `SFX_FILES` at the new file name (without `.wav`).
