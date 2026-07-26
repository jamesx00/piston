# Piston

Code execution engine (engineer-man/piston, fork under jamesx00/piston, remote `origin`).

## Layout

- `api/` — Node API server that runs code inside `isolate` sandboxes. `api/Dockerfile` builds it.
- `builder/` — thin image (`FROM ghcr.io/engineer-man/piston:latest`) used to bundle package sources for publishing.
- `repo/` — the **package builder** image (`repo/Dockerfile`, `debian:buster-slim`). `docker build repo` produces `piston-repo-builder`, which compiles a language from source and packages it as a `.pkg.tar.gz`.
- `packages/<language>/<version>/` — one dir per runtime, each with:
  - `build.sh` — downloads + compiles the interpreter into the package dir (must leave only what's needed for `run` to work).
  - `run` — invoked with `$1` = main file, rest = args; stdin piped in directly.
  - `compile` — only if the language needs a separate compile step.
  - `environment` — `export` statements using `$PWD` (set to the package dir at runtime).
  - `test.<ext>` — must print `OK`.
  - `metadata.json` — `language`, `version`, `aliases`, optional `dependencies`/`provides`/`limit_overrides`.
- `cli/` — Node CLI (`node cli/index.js`), wrapped by the root `./piston` script.
- `./piston` — dev entrypoint: `start`/`stop`/`restart`/`bash`/`logs` wrap docker-compose; `build-pkg <lang> <version>` builds one package; `list-pkgs`, `clean-pkgs`, `clean-repo`, `lint` are dev helpers.

Runtimes are **not** baked into the API image. They're built as standalone packages and installed at runtime via `ppman` into a shared volume the API reads from. "Add Python 3.14" means adding a `packages/python/3.14.0/` dir, not touching `api/Dockerfile`.

## Adding/updating a package

Follow `packages/CONTRIBUTING.MD`. Fastest path: copy the nearest existing version dir (e.g. `packages/python/3.12.0/`) and bump the version/URL/binary name in `build.sh`, `run`, and `metadata.json`.

Test locally:
```
./piston build-pkg <language> <version>
./piston ppman install <language>=<version>
./piston run <language> -l <version> packages/<language>/<version>/test.*
```

## Local build environment quirks (discovered 2026-07-26)

`repo/Dockerfile`'s `debian:buster-slim` base is EOL — `deb.debian.org` 404s on buster now. Patched the Dockerfile to repoint `sources.list` at `archive.debian.org` and disable `Acquire::Check-Valid-Until`, otherwise `docker build repo` fails at `apt-get update` before any package build ever starts.

On Apple Silicon (arm64) hosts, `repo/Dockerfile` still fails on `linux-headers-amd64` (no arm64 equivalent package name) unless the build is forced to amd64:
```
DOCKER_DEFAULT_PLATFORM=linux/amd64 ./piston build-pkg <language> <version>
```
This runs the builder under QEMU emulation, so compiling a language from source (e.g. CPython) takes noticeably longer than on native amd64.

## Working branch

Active work happens on `develop`, branched from `master`.
