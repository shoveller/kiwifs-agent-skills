# Discord answer → KiwiFS operational guide

Use when a user asks to save a useful chat answer as a wiki page, especially from Discord/mobile context.

## Pattern captured

1. Treat the immediately preceding assistant answer as source unless the user names another scope.
2. If the user gives an exact filename and parent area, honor it directly rather than re-titling. For example: `30 Wiki/<requested filename>.md`.
3. Search first for near-duplicates using the requested title and topic terms.
4. Convert the answer into a standalone guide, not a transcript:
   - add compact YAML frontmatter (`title`, `doc_type`, `topic`, `tags`, `status`, `updated_at`, `source_context`);
   - keep copyable commands;
   - add an early terminology table when the guide contains recurring operational terms;
   - separate host vs container commands when both appear;
   - preserve warnings and architecture prerequisites.
5. Use KiwiFS `write`, then run `lint` and usually `health_check`.
6. Return the exact path and a URL-encoded absolute wiki URL when the wiki host is known.

## Example shape

```markdown
---
title: <requested title>
doc_type: guide
topic: <topic>
tags: [<tags>]
status: published
updated_at: <ISO timestamp>
source_context: Discord 답변 정리
---

# <requested title>

<short purpose paragraph>

## 자주 나오는 용어

| 용어 | 의미 |
|---|---|
| host | ... |
| container | ... |

## 1. ...

```bash
<copyable command>
```
```

## Pitfalls

- Do not save a raw chat log when the user asked for a wiki guide.
- Do not skip validation after writing; report lint/health results.
- Do not invent the URL by hand when non-ASCII paths are involved; URL-encode the path.
