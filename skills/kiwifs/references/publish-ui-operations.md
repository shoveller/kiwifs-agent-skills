# KiwiFS publish UI operations notes

Use this when the task is about the KiwiFS web app's publish feature, not just wiki content CRUD through MCP.

## Publish state model

- The web publish flow stores state on the markdown page frontmatter, commonly as `published: true` plus optional `published_at`.
- Existing publish-related server/API code may already expose endpoints similar to:
  - `POST /api/kiwi/publish`
  - `POST /api/kiwi/unpublish`
  - `GET /api/kiwi/publish/status`
- Before adding new UI affordances, inspect the existing publish handler and button component to reuse the same state predicate rather than inventing a second source of truth.

## Explorer visibility pattern

When users can publish pages but cannot easily see what is public, a small, low-risk enhancement is:

1. Add a read-only API that scans markdown pages and returns only pages whose frontmatter is published.
2. Register it near the existing publish routes, e.g. `GET /api/kiwi/publish/list`.
3. In the UI, fetch the list after space selection/load and derive a `Set<string>` of published paths.
4. Add a `Published` section or filter/toggle in the file explorer/sidebar using the same page navigation component as normal pages.
5. If the published list contains many paths, render it as a path hierarchy rather than a flat list: build a folder/page tree from published paths, show folder rows with a folder icon, and keep the published page rows primary-tinted. This preserves context for deeply nested wiki paths.
6. Highlight published pages in all explorer lists with the same design tone as the active `Published` button, e.g. primary-tinted border/background/text and optional small dot/RSS indicator.
7. Keep publish/unpublish semantics unchanged; the explorer list is discoverability-only.

## Context-menu publish controls

When the user wants publish/unpublish directly from the file explorer context menu:

1. Reuse the existing `publish`/`unpublish` API client methods for page rows; do not introduce a second state mutation endpoint unless backend semantics require it.
2. Add page context-menu items next to other page actions:
   - `Publish` disabled when the page is already published.
   - `Unpublish` disabled when the page is not published.
3. For folder rows, collect descendant markdown files only (for example `collectFiles(entry).filter(isMarkdown)`) and batch over those paths.
4. Folder menu affordances should show scope and be safe by default:
   - `Publish folder (N)` disabled when there are no markdown files or every markdown file is already published.
   - `Unpublish folder (N)` disabled when none of the markdown files is published.
5. For sidebar action-menu consistency and Radix popover pitfalls, see `references/published-sidebar-actions.md`.
6. If folder publish/unpublish feels slow, add or reuse a dedicated backend bulk endpoint (for example `POST /publish/bulk` and `POST /unpublish/bulk`) that accepts `{ paths: string[] }` and uses the pipeline's bulk write path so many frontmatter updates become one HTTP request, one bulk commit, and one index batch. Fall back to sequential page calls only when no bulk endpoint exists.
7. Make bulk publish/unpublish tolerant of per-file failures without making it stricter than single-page publish. Do not reject candidates solely from pre-write lint: single-page publish may repair malformed frontmatter while setting `published`/`published_at`. Prefer trying `BulkWrite`, then recursively split failed batches to isolate only truly failing files and return them in `errors[]`; surface those skipped paths in the UI so failures do not look like "no response".
8. After any publish/unpublish action, refresh the top-level published list state (for example via an `onPublishedChanged` callback and a refresh key) so sidebar hierarchy, row highlighting, Starred/Pinned/Recent, and the main tree all update immediately.
9. Do not smoke-test by mutating real production publish state unless the user explicitly approves; verify action wiring through UI bundle markers and endpoint health, then leave data-changing checks to an explicit manual/approved test.

## Implementation cautions

- Do not key the UI solely from button state; derive explorer highlighting from backend page state so refreshes and multi-client edits stay correct.
- Keep the frontmatter predicate shared with existing backend code when available (for example, a helper like `PagePublished(content)`).
- Preserve current tree/navigation components and add props for `publishedPaths` instead of forking the tree renderer.
- When adding a publish list endpoint, keep it read-only and colocated with existing publish handlers/routes (`internal/api/handlers_publish.go`, route registration in `internal/api/server.go` in the current app layout). Return stable fields such as `path`, `published_at`, and `public_url`; sort by `published_at` descending then path for deterministic UI.
- In the React UI, centralize the fetch in the top-level app state (`ui/src/App.tsx` in the current layout), expose a `Set<string>` to sidebar/tree/list components, and add visual state through props (`publishedPaths`, `isPublished`) rather than duplicating row components.
- Apply published highlighting consistently across every explorer surface, not only the main tree: normal Pages tree, Published section/filter, Starred, Pinned, and Recent should all use the same primary-tinted tone so the user can tell which documents are public wherever they appear.
- For the Published section itself, prefer a derived tree helper (for example `buildPublishedTree`) and a small recursive component (for example `PublishedTree`) over reusing a flat recent/starred list. Sort folders and pages deterministically so the sidebar is stable after refresh.
- Store a sidebar Published-section toggle in local storage with a specific key (for example `kiwifs-show-published-list`) so it persists without changing publish semantics.
- Watch for React hook ordering and duplicate state declarations when inserting new state/effects into `App.tsx`; put derived publish-list effects after the state they depend on.
- Verify with both backend tests/build and the UI typecheck/build before touching the production container.

## Production patch workflow

- Treat production patching as a minimal diff from the deployed or upstream commit: inspect the running container image/ref, create a separate worktree/branch, patch there, then build/test.
- If the deployment carries patch files (for example `docker/kiwifs/patches/*.patch` in an ops repo), refresh the patch from a clean checkout of the deployed upstream tag, verify `git apply --check`, then copy only the patch artifact back into the ops repo. This avoids silently depending on unrelated local working-tree edits.
- For external PRs, first validate the PR-derived diff against the exact production tag, not just the contributor branch. If the PR was authored against a different UI layout, port it in a scratch checkout and run typecheck/build before touching production.
- If the user asks to sync to latest upstream before patching, explicitly call out that rollout may become an upstream-version replacement rather than a tiny hotfix when the deployment lags far behind upstream; verify compose/image tags before restart.
- If Docker is controlled from inside a container, remember that bind mount paths are interpreted by the host Docker daemon, not by the agent container path namespace.
- When rebuilding via Compose, unset stale KiwiFS override env vars that can redirect the build away from the intended source/image tag (for example `KIWIFS_REF`, `KIWIFS_SOURCE_REF`, `KIWIFS_IMAGE_TAG`) before `docker compose build kiwifs`.
- Rebuild/restart only the KiwiFS web service after verification unless the stack change requires broader rollout. If a companion service uses the same rebuilt image but runs a different command (for example an MCP-only `private-wiki` container), restart it only when image parity matters and do not use it for web UI smoke tests.

## Verification checklist for publish explorer changes

- UI: run the project typecheck and production build (`npm run typecheck`, `npm run build` from `ui/`). Large Vite chunk warnings are not failures unless the command exits non-zero.
- Backend: format touched Go files and run focused API/RBAC tests (`go test ./internal/api ./internal/rbac`) before broader rollout.
- Runtime smoke: after restart, call the same route the UI client uses, usually `GET /api/kiwi/publish/list`, and confirm it returns published markdown paths. Do not probe invented shortcut paths such as `/api/published-pages` unless the code actually registers them.
- Runtime process check: verify the target container is running the web server command (`kiwifs serve ...`) before using it for browser/API smoke tests. Containers running `kiwifs mcp --http ...` may expose MCP only and return 404 for web UI/API routes even when the image was rebuilt correctly.
- Browser smoke: check the sidebar for the `Published (N)` section/toggle, hierarchical folder grouping for nested paths, and primary-tinted published rows. If the first click result is ambiguous in an automation snapshot, reload and re-check rather than assuming the UI failed.
- If patching an older production tag, also verify the hotfix applies cleanly to that tag/version before rollout. This catches accidental dependency on unrelated local edits or newer upstream UI structure.
