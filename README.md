# MINEXE releases (public)

This **public** repo hosts MINEXE builds and the update manifest the utility
reads to auto-update itself. No secrets live here — the client pulls updates
from here without any key or token (runtime unlock is still key-locked by the
separate license server).

## How auto-update works

The client reads **`latest.json`** on launch:

```json
{ "version": "1.0.2",
  "tag": "v1.0.2",
  "sha256": "<sha-256 hex of the exe>",
  "url": "https://github.com/OWNER/minexe-releases/releases/download/v1.0.2/MINEXE.exe" }
```

If `version` differs from the running build, the client downloads `url`,
verifies its SHA-256 against `sha256`, and silently swaps + relaunches. A
mismatch is refused, so a corrupt/substituted download never runs.

`latest.json` is **generated automatically** by
`.github/workflows/build-manifest.yml` every time you publish a Release — you
never edit it by hand.

## Publishing a new build

1. In the **client** repo, bump `version.txt` (e.g. `1.0.2`) and rebuild
   `Rewrite.exe` (rename it `MINEXE.exe` if you like — the manifest just needs
   an `.exe` asset).
2. Create a GitHub **Release** here with tag **`v<version>`** (must match
   `version.txt`, e.g. `v1.0.2`) and attach the built `.exe`.
3. The Action runs, computes the SHA-256, writes `latest.json`, and commits it.
4. Every running client picks up the update on its next launch.

**Keep the tag and `version.txt` in lockstep** (`v1.0.2` ⇄ `1.0.2`), or the
client's version comparison won't detect the update.

## One-time client wiring

In the client's `src/Updater.hpp`, set `kManifestUrl` to this repo's raw file:

```
https://raw.githubusercontent.com/OWNER/minexe-releases/main/latest.json
```

(replace `OWNER`). Rebuild once after setting it.
