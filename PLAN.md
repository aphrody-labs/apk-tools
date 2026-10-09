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

## Windows natif (MS-d, 2026-10-09)

Question : remplacer pacman de MSYS2 par apk-tools compilé en PE natif (sans msys-2.0.dll).
Vérification : `clang 23 -fsyntax-only -ferror-limit=0` sur les 70 TU d'apk 3.0.8 (lib + applets + libfetch).

| cible | erreurs | en-têtes manquants | types | identifiants/constantes | fonctions | autres |
|---|---|---|---|---|---|---|
| x86_64-pc-windows-msvc (UCRT) | 420 (vague 1, arrêts fatals) | 48 (unistd.h ×32, getopt.h ×7, poll, fnmatch, dirent, sys/mman, sys/socket…) | 263 | 37 | 9 | 63 |
| msvc + en-têtes POSIX vides | 1 695 | 31 (getopt.h) | 817 (mode_t, uid_t, gid_t, ssize_t, nlink_t) | 453 (AT_FDCWD ×50, PATH_MAX, O_CLOEXEC, O_DIRECTORY) | 199 (openat, unlinkat, fstatat, close, read…) | 195 |
| x86_64-w64-windows-gnu, en-têtes mingw-w64 ucrt64 | 711 | 15 (fnmatch, sys/mman, poll, sys/xattr, netdb, sys/socket) | 518 (uid_t ×237, gid_t ×236, nlink_t ×45) | 98 (AT_FDCWD ×70) | 22 | 58 |

Sites POSIX bloquants (82 fichiers .c) : `*at`/dirfd 87 (database.c 19, fs_fsys.c 18, io.c 18), process
fork/exec/pipe 27 (process.c 14, database.c 10), sockets libfetch 18, fcntl/flock 16, signaux 8, chown/uid 6,
mmap 4, xattr via apk_xattr.h ; aide générée par lua (désactivable : `-Dhelp=disabled`).

Effort estimé d'un port natif : couche `*at` sur handles NT (`OBJECT_ATTRIBUTES.RootDirectory`) 5-8 j, process
(CreateProcess + scripts sous bunsh) 2-3 j, Winsock pour libfetch 2 j, liens/propriétaires/mmap 2 j, meson +
openssl/zlib/zstd 1 j, tests 2-3 j : 15-20 jours (12-15 sur mingw-ucrt).

Verdict : pas de port pour l'instant. Le client Windows retenu est en Bun, sans patch d'apk :
`aphrody-labs/bun` `scripts/aphrody/win/apk/apk.ts` (index v2 signé RSA/RSA256, résolution depends/provides/so:,
datahash et sommes par fichier, base `lib/apk/db/installed` au format apk), installeur `scripts/aphrody/win/install.ts`
et action `.github/actions/setup-aphrody-win`. À rouvrir si le dépôt passe au format v3 (adb, `apk mkndx`).
