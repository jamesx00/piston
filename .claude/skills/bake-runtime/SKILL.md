---
name: bake-runtime
description: Use when adding a language/version, or a new dependency for one already there, to Dockerfile.prebuilt — the standalone image that bakes a fixed runtime set into /piston/packages at build time for platforms with no sidecar repo service or shared volume (e.g. AWS App Runner). Covers editing the three Dockerfile.prebuilt blocks, building the image, running it standalone, and verifying via the real execute API before opening a PR.
---

# Bake Runtime (Dockerfile.prebuilt)

`Dockerfile.prebuilt` bakes a fixed runtime set directly into `/piston/packages`
at `docker build` time — no `repo` sidecar, no shared volume, no `ppman install`
at container start. See readme's "Standalone Deployments (Dockerfile.prebuilt)"
section for why it exists. This skill is for changing *that* image's runtime
set. For adding/updating a package in the normal dev flow (`packages/` +
`./piston build-pkg` + `ppman install` against the `docker-compose.dev.yaml`
stack), use the sibling **add-runtime** skill instead.

## Procedure

1. **Branch off `develop`**, never `master` (protected, upstream-sync only):
   ```
   git checkout develop && git pull && git checkout -b runtime/<short-slug>
   ```
   e.g. `runtime/prebuilt-add-ruby`, `runtime/prebuilt-python-orjson`.

2. **If the package doesn't exist under `packages/` yet**, create it first
   following `packages/CONTRIBUTING.MD` (copy the nearest version dir, adjust
   `build.sh`/`run`/`environment`/`metadata.json`/`test.*`), and sanity-check
   it builds standalone before touching `Dockerfile.prebuilt`:
   ```
   ./piston build-pkg <language> <version>
   ```
   If you're only adding a dependency to a package already baked in (e.g.
   another pip/npm library), just edit that package's `build.sh` — skip to
   step 3.

3. **Edit `Dockerfile.prebuilt` in exactly three places** (same three spots
   the readme's "Adding another runtime" walkthrough shows):
   - `pkgbuilder` stage's `make` target list — add `<language>-<version>.pkg.tar.gz`.
   - Final stage's `COPY --from=pkgbuilder` block — add the matching
     `COPY --from=pkgbuilder /piston/packages/<language>/<version> /piston/packages/<language>/<version>` line.
   - Final stage's `for pkgdir in ...` loop — add that same path, so `.env`
     and `.ppman-installed` get generated for it (these two files are what a
     live `ppman install` would normally create; without them the package
     either won't register at boot or will crash on first execution — see
     `api/src/package.js`/`api/src/index.js`).

   Adding a dependency to an already-baked package needs none of this — the
   `pkgbuilder` stage rebuilds that package's `build.sh` from scratch every
   time, so the new dependency is picked up automatically.

4. **Build the image**:
   ```
   docker build -f Dockerfile.prebuilt -t piston-prebuilt-test .
   ```

5. **Run it standalone** — no compose, no volume, exactly how App Runner would:
   ```
   docker run -d --name piston-prebuilt-test --privileged -p 2002:2000 piston-prebuilt-test
   ```
   The entrypoint does a recursive `chown -R piston:piston /piston` before
   starting the server, which takes noticeably longer the more/bigger the
   baked-in runtimes are. Poll logs instead of guessing a sleep:
   ```bash
   for i in $(seq 1 60); do
     docker logs piston-prebuilt-test 2>&1 | grep -q "API server started" && break
     sleep 2
   done
   ```
   Don't `docker exec` a manual `node src` into the container while waiting —
   it'll grab port 2000 itself and crash the entrypoint's own boot with
   `EADDRINUSE` once chown finishes.

6. **Verify against the real execute API**, not just the package's `test.*`:
   ```bash
   curl -s http://localhost:2002/api/v2/packages | python3 -c \
     "import sys,json; d=json.load(sys.stdin); print([p for p in d if p['language']=='<language>'])"

   curl -s -X POST http://localhost:2002/api/v2/execute -H 'Content-Type: application/json' -d '{
     "language": "<language>", "version": "<version>",
     "files": [{"name": "main.<ext>", "content": "<code exercising the change>"}]
   }' | python3 -m json.tool
   ```
   Confirm `installed: true` and `code: 0` with the expected stdout.

7. **Clean up the test container/image**:
   ```
   docker rm -f piston-prebuilt-test && docker image rm piston-prebuilt-test
   ```

8. **Commit and push.** If step 2 added a brand-new package, include both
   `packages/<language>/<version>/` and `Dockerfile.prebuilt`.
   ```
   git push -u origin runtime/<short-slug>
   ```

9. **Open a PR into `develop`** — do not merge it, the user merges:
   ```
   gh pr create --base develop --title "..." --body "..."
   ```
   Report the PR URL and stop.

## Common mistakes

- Editing only one or two of the three `Dockerfile.prebuilt` blocks (`make`
  target, `COPY`, the `.env`/`.ppman-installed` loop) — all three are
  required, none of them alone is sufficient.
- Verifying with `./piston run` against the dev stack instead of actually
  building `Dockerfile.prebuilt` and running a container from it — that
  proves the package builds, not that the baked-in image boots and serves it.
- Assuming a fixed sleep is enough before curling the API — poll for "API
  server started" in the logs instead, since boot time scales with how much
  is baked in.
- Merging or force-pushing the PR yourself.
