---
name: add-runtime
description: Use when adding a new language/version package, or adding a dependency (a new pip/npm/etc library) to an existing package, in this Piston fork's normal packages/ + docker-compose.dev.yaml dev flow. Covers building the package, installing it into a running dev instance and verifying via the real execute API, then branching and opening a PR.
---

# Add Runtime

End-to-end flow for changing a `packages/<language>/<version>/` package in the
normal dev setup (`docker-compose.dev.yaml`, the `api`+`repo` containers,
`ppman install`): build it, prove it actually works against a running
instance, then ship it as a PR. Applies to both a brand-new language/version
and adding a dependency (e.g. a new pip/npm library) to an existing one.

If this runtime also needs to be baked into the standalone `Dockerfile.prebuilt`
image (no sidecar/volume, e.g. AWS App Runner), use the sibling **bake-runtime**
skill for that part instead — it's a separate build path.

## Procedure

1. **Branch off `develop`**, never `master` (protected, upstream-sync only):
   ```
   git checkout develop && git pull && git checkout -b runtime/<short-slug>
   ```
   e.g. `runtime/python-314-faker`, `runtime/add-ruby-301`.

2. **Make the change**:
   - New language/version: copy the nearest existing version dir, follow
     `packages/CONTRIBUTING.MD` (`build.sh`, `run`, `environment`,
     `metadata.json`, `test.*`).
   - New dependency: edit the existing package's `build.sh` (the
     `pip3 install ...` / `npm install ...` line).

3. **Build**: `./piston build-pkg <language> <version>`. On arm64 hosts,
   prefix with `DOCKER_DEFAULT_PLATFORM=linux/amd64` (see root `CLAUDE.md`).

4. **Get a running instance to install into**: `./piston start`. If host
   port 2000 is already taken by something unrelated, don't edit the tracked
   `docker-compose.dev.yaml` — layer an untracked override instead:
   ```yaml
   # docker-compose.local.yaml (gitignored)
   services:
     api:
       ports: !override
         - 2001:2000
   ```
   then:
   ```
   docker compose -f docker-compose.dev.yaml -f docker-compose.local.yaml \
       up -d --force-recreate api
   ```
   (plain list-merge concatenates `ports` instead of replacing it — only
   `!override` actually replaces the list.)

5. **Install** (or reinstall, if this package was already installed):
   ```
   ./piston ppman uninstall <language>=<version> --piston-url http://127.0.0.1:<port>  # only if already installed
   ./piston ppman install   <language>=<version> --piston-url http://127.0.0.1:<port>
   ```

6. **Run the image and verify.** Don't stop at `./piston run`'s `test.*` —
   the web app talks to the HTTP API, not the CLI, so that's the path that
   needs proving. Hit the real execute endpoint with a snippet that exercises
   exactly what changed (the new import, the new language's syntax, etc.):
   ```bash
   curl -s -X POST http://localhost:<port>/api/v2/execute -H 'Content-Type: application/json' -d '{
     "language": "<language>", "version": "<version>",
     "files": [{"name": "main.<ext>", "content": "<code exercising the change>"}]
   }' | python3 -m json.tool
   ```
   Confirm `code: 0` and the expected stdout. For a new language/version,
   also confirm `GET /api/v2/packages` shows it with `installed: true`.

7. **Commit and push.**
   ```
   git push -u origin runtime/<short-slug>
   ```

8. **Open a PR into `develop`** — do not merge it, the user merges:
   ```
   gh pr create --base develop --title "..." --body "..."
   ```
   Report the PR URL and stop.

## Common mistakes

- Hand-copying a package dir instead of going through `ppman install`, then
  wondering why the API doesn't see it: a package needs both `.env` and
  `.ppman-installed` present to be picked up (`.env` is read on first
  execution and throws if missing; `.ppman-installed` is what the boot-time
  scan checks for — see `api/src/package.js` and `api/src/index.js`).
  `ppman install` creates both; a raw file copy doesn't.
- Testing only via `./piston run` and skipping the `/api/v2/execute` call —
  that CLI path doesn't prove the HTTP path your web app actually uses works.
- Merging or force-pushing the PR yourself.
