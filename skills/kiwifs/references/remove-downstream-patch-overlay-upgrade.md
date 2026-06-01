# Remove downstream KiwiFS patch overlay during upstream upgrade

Use this when production KiwiFS was previously built from an upstream source ref plus `docker/kiwifs/patches/*.patch`, and upstream has since released a version that contains the hotfixes.

## Pattern

1. Discover the latest upstream tag and matching Docker image tag separately.
   - Source tags may be `vX.Y.Z`.
   - Docker image tags are not always consistent across releases; verify the actual image tag with `docker manifest inspect` or registry tag listing before editing Compose defaults.
   - Example observed transition: old runtime used `0.19.4`, but the latest image existed as `v0.20.0`.
2. Remove the patch overlay rather than carrying stale hotfixes forward:
   - Delete tracked `docker/kiwifs/patches/*.patch` files.
   - Simplify `docker/kiwifs/Dockerfile` to `FROM ${KIWIFS_IMAGE}:${KIWIFS_IMAGE_TAG}` plus only the operational entrypoint/config overlay that is still needed.
   - Remove source-clone, UI build, Go build, and `git apply` stages if their only purpose was applying the downstream patches.
3. Update defaults consistently:
   - `docker-compose.yml` build args and image name defaults.
   - `.stack.env.example` public template.
   - Ignored operational `.stack.env` if the running environment depends on it, but do not stage it.
   - README/runbook text that described the patch overlay.
   - Tests that asserted patch file names or patch markers; invert them to assert no patch overlay and no `KIWIFS_SOURCE_REF` remains.
4. Verify before committing:
   - Focused tests for KiwiFS stack config/tests.
   - Full available unittest discovery if cheap.
   - `git diff --check`.
   - `docker compose --env-file .stack.env build kiwifs` with stale `KIWIFS_REF`, `KIWIFS_IMAGE_TAG`, and `KIWIFS_SOURCE_REF` unset from the shell.
   - `docker compose --env-file .stack.env up -d kiwifs private-wiki`.
   - `docker compose ps kiwifs private-wiki` and container-internal `/health` checks.
5. Commit and push only the intended tracked files.

## Pitfalls

- Do not assume the Docker tag drops or keeps the `v` prefix just because the Git tag does. Verify the manifest.
- If the patch directory becomes empty, Docker `COPY docker/kiwifs/patches/*.patch` will become fragile; remove the entire patch-copy/apply flow.
- Compose commands in this environment can pick up stale exported shell variables over `.stack.env`; explicitly unset old KiwiFS variables for build/recreate commands.
- Git commit hooks may generate untracked artifacts such as `graphify-out/`; inspect `git status` after commit and remove/ignore generated artifacts before pushing unless they are intentionally part of the change.
- External `curl 127.0.0.1:<port>/health` can fail from inside a different container/session namespace even when Docker reports published ports. Prefer `docker compose ps` and `docker exec <container> wget -qO- http://127.0.0.1:3333/health` for final runtime proof.
