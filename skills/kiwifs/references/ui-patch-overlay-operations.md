# KiwiFS UI patch overlay operations

Use this reference when maintaining a downstream KiwiFS production patch that rebuilds the UI/binary from a pinned upstream tag and overlays the result into a runtime image.

## Captured pattern

- Keep downstream UI changes as a clean `docker/kiwifs/patches/*.patch` file against the exact upstream source ref, rather than editing vendored source in place.
- Validate patch portability with a fresh upstream clone:

```bash
rm -rf /tmp/kiwifs-check
git clone --depth 1 --branch "$KIWIFS_SOURCE_REF" https://github.com/kiwifs/kiwifs.git /tmp/kiwifs-check
cd /tmp/kiwifs-check
git apply --check /path/to/docker/kiwifs/patches/0001-*.patch
git apply /path/to/docker/kiwifs/patches/0001-*.patch
git diff --check
```

- Run the UI build in that fresh checkout before trusting the Docker build. TypeScript catches React callback signature mistakes that simple marker checks miss:

```bash
cd /tmp/kiwifs-check/ui
npm ci --no-audit --no-fund --loglevel=error
npm run build
```

- Then build the operational image and recreate the dependent services. In environments where shell exports may override Compose defaults, explicitly unset stale KiwiFS env vars:

```bash
env -u KIWIFS_REF -u KIWIFS_IMAGE_TAG docker compose --env-file .stack.env build kiwifs
env -u KIWIFS_REF -u KIWIFS_IMAGE_TAG docker compose --env-file .stack.env up -d kiwifs private-wiki
env -u KIWIFS_REF -u KIWIFS_IMAGE_TAG docker compose --env-file .stack.env ps kiwifs private-wiki
```

## UI title/frontmatter pitfall

When patching visual editor title editing:

- Patch the normal `EditorInner` path, not `ExcalidrawEditorInner`, unless Excalidraw explicitly has the same state and handlers. Accidentally adding `editorMode`, `displayTitle`, or `handleVisualTitleChange` to Excalidraw breaks unrelated editor paths.
- Source-mode title can be derived from `sourceText`; visual-mode title must be derived from the live visual frontmatter buffer (`fmText`) so saved frontmatter follows current edits.
- Do not bind the visual title `<Input>` directly to a parsed/derived `displayTitle` if that value is recomputed from YAML/frontmatter on every render. Use a small local input state, e.g. `titleInputValue`, update it first in `onChange`, then update `fmText`. Sync the local state from `displayTitle` in an effect for document/path changes. This prevents keystrokes from appearing to be ignored or reverted while `fmText` parsing/save state catches up.
- Keep the sync bidirectional. Title input changes must update `fmText`, and direct edits in the visual-mode Frontmatter textarea must parse the new `title:` line and update the title input's local state. A common bug is fixing input → frontmatter while leaving frontmatter → input stale.
- Parse YAML title defensively. Prefer `gray-matter` for `fmText`, but add a line-based fallback that recognizes leading whitespace and quoted titles; unquote double/single quoted values before assigning the input state. This matters for Korean/English mixed titles like `title: "Shadcn 의 React Hook Form 추상화 가이드다"`.
- Use nullish fallback (`fmTitle ?? titleize(path)`) so an intentionally empty title is not immediately replaced by the path title.
- If the input sits inside a row with action buttons, give the title wrapper `flex-1 min-w-0` and the input `w-full min-w-0`; otherwise `w-full` can still render as a short input because the parent did not take remaining flex space.
- Do not route title input changes through a generic visual editor dirty callback if that callback intentionally ignores edits before BlockNote is ready. Use a title-specific dirty/autosave helper or otherwise ensure `setSaveStatus("dirty")` and autosave scheduling happen even before body-editor readiness.
- Preserve the existing visual save path (`joinFrontmatter(fmText, body)`) so updating `fmText` is sufficient for persistence.

## Source editor save/ETag conflict pitfall

When investigating `409 : {"error":"file modified since last read — re-fetch and retry"}` from Edit mode + Source mode after pressing the save shortcut, do not assume an external writer first. Check for duplicate client-side saves with a stale ETag:

- CodeMirror source editor may register its own `Mod-s` keymap while the app also has a global `keydown` handler for `Ctrl/Cmd+S`. If both fire, the first write succeeds and updates the file ETag; the second write may reuse the pre-save ETag and trigger the server's normal conflict protection.
- Fix both layers: make the global shortcut handler ignore events that were already prevented (`if (e.defaultPrevented) return;`) and add a save-in-flight guard in the editor (`useRef<Promise<boolean> | null>`) so concurrent save callers share the same promise instead of issuing multiple writes.
- Clear any pending autosave timer at the start of an explicit save. Otherwise a delayed autosave can run immediately after the manual save with stale state/ETag and reproduce the same conflict.
- Keep Source-mode shortcut behavior aligned with the visible Save button semantics (`Save & Close` if that is the product behavior) so keyboard and button paths do not diverge.
- Verification: patch a fresh upstream checkout, run `git apply --check`, `git apply`, `git diff --check`, then run `npm run build` from `ui/`. Browser reproduction is ideal, but the build plus source inspection is a minimum for this UI hotfix.

## Verification checklist

- Unit tests or repo tests that assert the patch file contains the expected markers.
- Fresh upstream patch apply check.
- `git diff --check` for non-patch files; patch files may intentionally contain context blank lines that look like `+ ` in unified diffs, but avoid unnecessary trailing whitespace where possible.
- UI `npm run build` from a patched fresh checkout.
- Docker image build.
- Service health after recreate.
- Runtime marker check against the built binary if browser automation is unavailable:

```bash
docker exec <kiwifs-container> sh -lc \
  'strings /usr/local/bin/kiwifs | grep -q "Page title" && strings /usr/local/bin/kiwifs | grep -q "w-full min-w-0 h-auto" && echo markers-ok'
```

Runtime bundles are minified, so local variable names such as `titleInputValue` or `setTitleInputValue` may not survive as literal strings. Prefer durable literal markers (`Page title`, class names, helper names like `setTitleInFrontmatterText`) and compile-time tests for source-level symbols.
