# Config attachment → wiki section update pattern

Use this when a user asks to take an attached config file, fix/comment it, and replace a wiki section with a copyable fenced block.

## Workflow

1. Obtain the actual config text before editing. If the attachment content is not available in the prompt/session, ask for a pasted code block or a text-renamed attachment rather than guessing.
2. Save the config to a temporary file and validate it with the native parser/loader when available.
   - Example for tmux: `tmux -L <temp-socket> -f /tmp/file.conf new-session -d -s validate`, then kill that temp server.
3. Fix syntax and obvious typos before translating or adding comments.
4. Convert all comments to the requested language and add comments immediately before previously uncommented statements so the result remains copyable.
5. Replace the target document section with a fenced code block using the right language tag, e.g. ````tmux` for `.tmux.conf`.
6. Verify the wiki page after writing:
   - Run KiwiFS lint.
   - Run health check for important pages.
   - Re-read the changed region or page from the backing file/page, because section extraction can summarize or truncate long fenced blocks and may show only the opening fence.

## Pitfalls captured

- Do not assume a code/config attachment was ingested just because the user attached a file; confirm the content is visible.
- For long fenced code blocks, `kiwi_section` may not be enough to verify the full replacement. Use a full read or bounded file/page read around the edited lines.
- Prefer robust commands over shell pipelines embedded in config when tmux has a native command, e.g. `swap-window -t +1`/`-1` instead of parsing `tmux list-windows` with `grep`.
- Guard optional plugin `run-shell` lines with `if-shell test -f ...` so a missing local plugin checkout does not make the whole config fail to load.
