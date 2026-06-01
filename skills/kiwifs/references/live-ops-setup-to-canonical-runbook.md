# Live operations setup → canonical runbook update

Use this when a chat session turns a planned operating model into a working setup and the user asks to make it reproducible in KiwiFS.

## Pattern

1. **Search first for the existing canonical/near-canonical page.** Use exact hostnames, service names, account names, paths, and semantic variants. Prefer updating the existing runbook over creating a new session note.
2. **Preserve the previous design context.** If the old page describes an earlier candidate architecture, do not delete it unless the user asks. Add a clearly dated "confirmed setup" or "validated implementation" section that distinguishes final reality from earlier plan.
3. **Record only durable, non-secret facts.** Good: host alias, private Mesh IP, service/task name, install path, model names, bind address, verification commands, observed success criteria. Bad: private keys, tokens, passwords, cookies, raw secrets.
4. **Make the section reproducible.** Include copyable commands for:
   - installing/configuring the service or scheduled task,
   - assigning required account/group permissions,
   - checking service/task status,
   - probing API/port health,
   - post-reboot verification.
5. **Capture the actual final pivot.** If the session moved from a lower-privilege/wrapper ideal to an admin-account or scheduled-task operational compromise, state why and define the security boundary explicitly.
6. **Include real verification output shape, not invented output.** Summarize verified status such as task exists, last result `0`, bind `127.0.0.1:11434`, model names, or timeout caveats.
7. **Run `lint` and usually `health_check`.** Report the path, absolute wiki URL, validation result, and any remaining caveats.
8. **Update compact memory only for durable environment facts.** Example: host/IP, admin status, service path, task name, hardware class. Do not store one-off logs or commit-style progress.

## Windows Mesh Ollama example cues

For a Windows native Ollama node controlled through Mesh SSH, the reusable details to preserve are:

- Windows OpenSSH control account and whether it is local admin.
- Cloudflare Mesh IP/alias and that ICMP may be irrelevant; verify TCP/SSH/API instead.
- Ollama root, model directory, log directory, and `OLLAMA_HOST` binding.
- Boot persistence mechanism (`schtasks.exe` task name, trigger, `SYSTEM` user, target script).
- Verification commands: `schtasks /Query`, `/api/tags`, `netstat`, and post-reboot checks.
- Security boundary: keep Ollama on localhost unless the user explicitly designs a Mesh-exposed API path.

This is a documentation workflow reference, not a mandate to use those exact paths on other hosts.