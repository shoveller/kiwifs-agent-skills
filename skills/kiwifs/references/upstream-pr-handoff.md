# KiwiFS upstream / fork PR handoff notes

Use this when a KiwiFS fix was developed in a fork, production patch, or operations repository and the user wants it sent to a GitHub repo cleanly.

## First: verify the exact PR target

1. Identify all three repos before pushing:
   - canonical upstream: usually `kiwifs/kiwifs`
   - user fork / target fork: often `shoveller/kiwifs`
   - operations repo that may only carry a patch artifact: for example `ai-assistant`
2. Restate the intended PR shape as `base-owner/base-repo:base <- head-owner/head-repo:branch`.
3. Do not assume “send a PR to shoveller/kiwifs” means a cross-repo PR to `kiwifs/kiwifs`. If the user names the fork repo, a fork-local PR such as `shoveller/kiwifs:main <- shoveller/kiwifs:feature` may be the correct target.
4. If the user explicitly wants upstream maintainers to see it, open a cross-repo PR with base `kiwifs/kiwifs:main` and head `<fork>:<feature-branch>`.

## Operations patch → source PR workflow

1. Create or reuse an explicit working checkout under `/workspace/host/works`, not `/tmp`, for PR work:
   ```bash
   WORK=/workspace/host/works/kiwifs-<topic>-pr
   git clone https://github.com/<target-owner>/kiwifs.git "$WORK"
   cd "$WORK"
   git remote add upstream https://github.com/kiwifs/kiwifs.git || true
   git fetch upstream --prune
   git checkout -B <branch> upstream/main
   ```
2. Apply the operations patch from the source repo:
   ```bash
   git apply --check /path/to/operations-repo/docker/kiwifs/patches/0001-<topic>.patch
   git apply /path/to/operations-repo/docker/kiwifs/patches/0001-<topic>.patch
   git diff --stat
   ```
3. Run focused local checks from the source checkout before committing. For editor/UI changes this is usually:
   ```bash
   npm --prefix ui ci --no-audit --no-fund --loglevel=error
   npm --prefix ui run typecheck
   npm --prefix ui run build
   git diff --check
   ```
4. Clean-base patch gate before push/open PR:
   - If changes are not committed yet, generate the verification patch with `git diff --binary upstream/main`, not `git diff upstream/main...HEAD` (the three-dot committed-range diff is empty before commit).
   - If changes are already committed, `git diff --binary upstream/main...HEAD` is fine.
   ```bash
   git diff --binary upstream/main > /workspace/host/works/kiwifs-<topic>.patch
   VERIFY=/workspace/host/works/kiwifs-<topic>-verify
   rm -rf "$VERIFY"
   git clone https://github.com/kiwifs/kiwifs.git "$VERIFY"
   cd "$VERIFY"
   git reset --hard origin/main
   git apply --check /workspace/host/works/kiwifs-<topic>.patch
   git apply /workspace/host/works/kiwifs-<topic>.patch
   npm --prefix ui ci --no-audit --no-fund --loglevel=error
   npm --prefix ui run typecheck
   npm --prefix ui run build
   git diff --check
   ```
5. Commit with a concise public message and push to the intended head repo:
   ```bash
   git config user.name "Hermes Agent"
   git config user.email "hermes-agent@users.noreply.github.com"
   git add <changed-files>
   git commit -m "feat: make editor title editable"
   git push -u origin <branch>
   ```
6. Create the PR with an English, maintainer-facing body. Keep it product-oriented: why the UI behavior is confusing, what changed, and how it was tested. Avoid private wiki URLs, production hostnames, container mount paths, and temporary local workspace names.
7. Immediately check PR state and CI:
   ```bash
   gh pr view <n> --repo <owner>/kiwifs --json url,state,isCrossRepository,baseRefName,headRefName,headRepositoryOwner,mergeable,mergeStateStatus,statusCheckRollup
   gh pr checks <n> --repo <owner>/kiwifs
   ```

## PR body framing for editor title/frontmatter changes

Good framing:

- The page title display is where users naturally expect to rename the visible title.
- Editing the frontmatter `title` property directly is not intuitive, especially when the editor header shows a static title.
- Turning the title display into an input keeps frontmatter as the source of truth while making the operation discoverable.
- State explicitly whether the behavior works in Visual mode, Source mode, or both.

Example summary bullets:

```markdown
## Summary
- Replaces the editor header title display with an editable title input.
- Keeps the input synchronized with the YAML frontmatter `title` property.
- Supports both Visual mode and Source mode: editing the header title updates frontmatter, and editing frontmatter updates the header title.

## Motivation
Editing the frontmatter `title` property is not very intuitive when the page title is shown as a static heading in the editor. Since the title display is the natural place users look when they want to rename the visible page title, this PR turns that title area into an input while preserving frontmatter as the source of truth.
```

## Pitfalls

- Do not treat a fork-local PR (`fork:feature -> fork:main`) as an upstream contribution. It is useful when the user asks for that fork, but it is not visible as a contribution to canonical upstream unless the base is `kiwifs/kiwifs`.
- Do not push production-only operations patches directly without rebasing/validating against a clean upstream tree.
- Do not use `git diff upstream/main...HEAD` as a patch artifact before committing; it only captures committed range differences and can produce an empty patch.
- Do not overstate success: say the PR is open and list check statuses; only say it is mergeable after GitHub reports `MERGEABLE` and required checks pass.
