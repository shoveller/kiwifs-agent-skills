# Live operational runbook continuation from chat

Use this when a chat session is actively changing infrastructure and the user asks to keep the wiki updated as the setup evolves.

## Pattern

1. Search/read the existing canonical page first; do not create a new session-log page when a runbook already exists.
2. Append or insert dated sections for each confirmed operational change, keeping prior design/attempt history intact when it explains why the final shape differs.
3. Record the reproducible artifact, not just the narrative:
   - exact file paths
   - service/task names
   - copyable commands/scripts
   - expected success output
   - verification commands
   - final observed verification output
4. If a first attempt fails and is corrected in-session, document the durable pitfall and the final fix, not a long blow-by-blow transcript.
5. Keep secrets out: never persist private keys, passwords, bearer tokens, cookies, or sensitive host-only credentials. Prefer paths and non-secret identifiers.
6. When the operational state changes again in the same session, continue updating the same page so it remains the canonical reproduction guide.
7. After every write, run `lint`; for important operational pages, also run `health_check`.

## Useful dated-section shape

```markdown
## YYYY-MM-DD 추가: <change title>

왜 필요한가 / 어떤 문제를 해결하는가.

### 채택 구조

```text
<small architecture or sequence>
```

### 재현 절차

```powershell
# or bash
<copyable commands>
```

### 검증

```powershell
<verification commands>
```

성공 기준:

```text
<expected/observed output>
```

### 운영 팁 / 함정

- <pitfall discovered during the live session>
- <how to fix or avoid it next time>
```

## Example lessons from SUMIGA Ollama work

- For Windows scheduled operational tasks, record both the task creation command and `schtasks /Query /TN ... /V /FO LIST` success criteria.
- For model warmup, the durable lesson is: warm up the model name that actually appears in `/api/tags`; a stale model name causes a 404.
- Separate a long-running service task from a short warmup/verification task so warmup failures do not kill the service.
- Include log paths and representative success lines, e.g. `WARMED ... elapsed_seconds=... done=True`.
