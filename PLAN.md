# aphrody-labs/apk-tools

Fork of [alpinelinux/apk-tools](https://github.com/alpinelinux/apk-tools) for Aphrody Alpine
(chantier U1, see `PLAN-ALPINE-BUN.md` section U in aphrody-labs/bun).

## Status

- `master` tracks upstream, synced every 6 h by `.github/workflows/aphrody-upstream-sync.yml`.
- **No patch.** Aphrody Alpine 3.24 ships the stock `apk-tools` 3.0.8 from Alpine.

## Why no patch

apk is complemented by Bun without any change to apk itself:

- apk 3 runs package scripts, triggers and commit hooks with `execve()` (shebang honoured), so
  `#!/usr/bin/bun` and `#!/bin/bunsh` scripts work as-is.
- An apk v3 repository line ending in `/APKINDEX.tar.gz` (NDX type) resolves packages relative to
  the index and substitutes `${APK_ARCH}`, so the signed repository is hosted on GitHub Release
  assets of aphrody-labs/aports without a custom transport.
- No `apk bun …` subcommand: apk owns system packages (`/usr/lib/bun/node_modules` via aports),
  `bun add -g` owns `/usr/local/lib/bun` (excluded from `apk audit` by
  `/etc/apk/protected_paths.d/bun-global.list`), and both share `/var/cache/bun/install`.

## When to patch

Only if a Bun-side need cannot be met by scripts, triggers, hooks or repository configuration.
Patches go on a `aphrody/<topic>` branch and are listed here.
