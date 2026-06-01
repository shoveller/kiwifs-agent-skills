---
name: kiwifs
description: "Use when an agent should read, search, write, maintain, or operate a KiwiFS knowledge base exposed through a remote MCP server. Prefer this over direct filesystem or Obsidian-style CRUD for canonical wiki knowledge, backlinks, graph traversal, Kanban workflows, analytics, imports, and git-backed operational docs. Trigger: /kiwifs"
allowed-tools: Read, Write, Edit, MultiEdit, Bash(git:*), Bash(npx:*), mcp__kiwifs__*, mcp__*_kiwi__*
license: MIT
metadata:
  homepage: https://github.com/shoveller/kiwifs-agent-skills
  tags: [kiwifs, mcp, knowledge-base, wiki, markdown, kanban]
---

# KiwiFS MCP Knowledge Base

Use this skill when KiwiFS is the canonical markdown wiki, knowledge graph, or operational knowledge store exposed to the agent through a remote MCP server. KiwiFS is more than a folder of `.md` files: use its MCP/API surface so search indexes, wiki links, backlinks, graph analytics, workflows, linting, git history, and computed state stay coherent. This skill does not install or host KiwiFS; it assumes the runtime already has a remote KiwiFS MCP server connected.

## When to Use

Use KiwiFS before local-file or Obsidian-style access when the user asks to:

- Search or answer from a wiki, knowledge base, vault, notes, docs, runbook, or long-term project knowledge.
- Create, update, append, rename, delete, lint, or health-check markdown pages.
- Maintain canonical docs, indexes, reports, operational records, or task pages.
- Work with wiki links, backlinks, graph neighborhoods, orphan pages, centrality, or topic clusters.
- Use Kanban/task/workflow features from the knowledge base.
- Ingest PDFs, DOCX/PPTX/XLSX/HTML/EPUB, clip web pages, or import structured data.
- Analyze vault health, stale pages, broken links, coverage gaps, or memory consolidation.
- Create visual canvases or computed/query-backed views.

Do not use KiwiFS as the first choice for:

- Short durable user preferences or environment facts: use the agent's memory tool.
- Reusable procedures and playbooks for future agents: write or update a skill.
- Raw recall of previous conversations: use session search.
- Bulk binary storage or secrets: keep secrets out of KiwiFS pages.

When the task is about modifying or operating the KiwiFS application itself (for example, publish UI, file explorer behavior, routes, Docker rollout), pair this skill with repository inspection/build verification rather than treating the wiki as only MCP content.

Relevant implementation notes:
- `references/publish-ui-operations.md` — publish-state and explorer-discoverability implementation notes.
- `references/task-orchestration-mcp-tools.md` — when implementing KiwiFS agent-task MCP tools such as `kiwi_task_create`, cross-check UC-1 and adjacent issues for the canonical task schema (`blocked_by`, `parent`, `artifacts`) instead of following a narrow issue parameter list blindly.
- `references/published-sidebar-actions.md` — Published sidebar/context-menu action consistency, Radix Popover trigger pitfalls, and verification checklist.
- `references/ui-patch-overlay-operations.md` — downstream KiwiFS UI patch overlay workflow: fresh upstream patch checks, UI build, Docker rebuild/recreate, and visual title/frontmatter editing pitfalls.
- `references/remove-downstream-patch-overlay-upgrade.md` — when upstream KiwiFS has absorbed local hotfixes, remove the patch overlay, verify real Docker image tags, update Compose/env/docs/tests, rebuild/recreate, and avoid committing generated hook artifacts.
- `references/mermaid-renderer-contrast-patch.md` — Mermaid renderer theme-boundary fix: do not force SVG text colors; preserve per-diagram `%%{init: ...}%%` theme/themeVariables while avoiding app dark-mode overrides.
- `references/upstream-pr-handoff.md` — clean cross-repo upstream PR workflow after fork/production patch work.
- `references/upstream-issue-surface-triage.md` — when evaluating KiwiFS upstream issues, verify the actual wired surface separately for CLI, REST API, MCP, UI, and core implementation before judging operational impact or recommending first-contribution targets.
- `references/canonical-doc-update-from-legacy-provenance.md` — updating active wiki docs by mining older canonical/archive/provenance notes for implementation patterns, then linking the source pages.
- `references/concise-operational-tutorials.md` — rewrite long/rhetorical wiki runbooks into short, flow-first operational tutorials with copyable steps and verification.
- `references/operational-checklist-docs.md` — turn a chat-derived operational answer into a reusable request-information/checklist wiki page with early terminology, copyable fill-in blocks, explicit secret boundaries, index updates, and verification.
- `references/secret-backed-cloud-request-forms.md` — fill cloud automation request/intake forms in KiwiFS without leaking secrets: store execution-context paths, non-secret identifiers, UI paths for unknowns, and quota/billing stop conditions.
- `references/config-attachment-doc-section-update.md` — workflow for taking an attached config file, validating/fixing it, translating/adding comments, and replacing a wiki section with a copyable fenced block; includes verification pitfalls for long code fences.
- `references/config-attachment-doc-section-update.md` — workflow for taking an attached config file, validating/fixing it, translating/adding comments, and replacing a wiki section with a copyable fenced block; includes verification pitfalls for long code fences.ex updates, and verification.
- `references/secret-path-operational-forms.md` — handling private keys/tokens in operational forms: record host vs container-visible paths, verify existence/permissions without printing secrets, and align config paths to the automation runtime.
- `references/config-attachment-doc-section-update.md` — workflow for taking an attached config file, validating/fixing it, translating/adding comments, and replacing a wiki section with a copyable fenced block; includes verification pitfalls for long code fences.
- `references/github-wiki-scrape-to-kiwifs.md` — archive all pages under a GitHub Wiki by discovering pages via the `.wiki.git` repository, clipping rendered pages into KiwiFS, writing a local README index, and linting the result.
- For ordered wiki-document renumbering, pair this skill with `wiki-workspace-note-taking`'s `references/wiki-sequence-renumbering.md`; rename/link rewriting is not enough, because README order, Mermaid node IDs, neighboring “next layer” text, and stale old-number phrases also need verification.
- For ordered wiki-document renumbering, pair this skill with `wiki-workspace-note-taking`'s `references/wiki-sequence-renumbering.md`; rename/link rewriting is not enough, because README order, Mermaid node IDs, neighboring “next layer” text, and stale old-number phrases also need verification.

## Remote MCP Tool Discovery

KiwiFS tools may be exposed with different prefixes depending on the remote MCP server name and agent runtime. Look for tools containing one of these prefixes:

```text
kiwi_*
kiwifs_*
mcp_kiwifs_*
mcp_<server>_kiwi_*
```

If multiple wiki/MCP servers are connected, prefer the one whose server or tool prefix clearly points to the user's canonical KiwiFS instance.

## Routing Rule

Before answering from memory, do a search-first pass unless the answer is already in the current prompt.

```text
User asks about wiki/docs/knowledge
  -> search or semantic search
  -> peek/read the most relevant page or section
  -> answer with source paths or URLs when useful

User asks to update docs
  -> search existing canonical pages first
  -> update in place when possible
  -> lint/health-check
  -> report changed paths/URLs

User asks to store knowledge
  -> decide: memory vs skill vs KiwiFS
  -> KiwiFS for long-form markdown, project docs, records, reports, and linked knowledge
```

## Core Operations Map

| Intent | Preferred KiwiFS capability |
|---|---|
| Full-text search | `search` |
| Semantic search / near-duplicates | `search_semantic`, `suggestions` |
| Quick page triage | `peek` |
| Read full page | `read` |
| Read one heading | `section` |
| Create/replace page | `write` |
| Append log/journal text | `append` |
| Batch coherent changes | `bulk_write` |
| Rename/move with link rewrite | `rename` |
| Delete with git history | `delete` |
| Validate markdown | `lint` |
| Page health | `health_check` |
| Backlinks | `backlinks` |
| One-hop graph walk | `graph_walk` |
| Shortest graph path | `graph_path` |
| Graph analytics | `graph_analytics`, `graph_centrality`, `graph_communities` |
| Metadata query | `query_meta` |
| Dataview-like query | `query` |
| Aggregation | `aggregate` |
| Analytics dashboard data | `analytics`, `velocity`, `timeline`, `feed`, `changes` |
| Git file history | `versions` |
| Embeddings | `embeddings` |
| Retrieval evaluation | `eval` |
| Export dataset | `export` |
| Web clip | `clip` |
| Document ingestion | `ingest` |
| Structured import | `import` |
| Memory consolidation audit | `memory_report` |
| Canvas list/read/write | `canvas_list`, `canvas_read`, `canvas_write` |
| Saved views | `views_list`, `views_get`, `views_save`, `views_execute`, `views_delete` |
| Computed view refresh | `view_refresh` |
| Workflows / Kanban | `workflow_list`, `workflow_get`, `workflow_save`, `workflow_board`, `workflow_advance` |
| Task claiming | `eligible`, `claim`, `claims_list`, `release` |
| Draft space | `draft_create`, `draft_read`, `draft_write`, `draft_diff`, `draft_merge`, `draft_discard`, `draft_list` |
| Knowledge-base context | `context` |

## Search and Answer Workflow

1. Start with keyword search for exact terms, names, paths, error messages, or titles.
2. Use semantic search when the user describes an idea but not the exact wording.
3. For vague "I may have a document about X" recall requests, run several keyword variants in parallel: English product names, Korean transliterations/translations, and nearby architecture terms. Treat semantic search as a supplement, not authority; if semantic results look broadly similar but not keyword-grounded, prioritize keyword hits and then `peek`/`section` the likely pages.
4. When the user asks for the "most recent" document or guide, combine topical search with recency tools (`feed`, `timeline`, `changes`, and, for candidates, `versions`). Do not rely on search ranking alone: recent Inbox/working-area drafts may be the actual answer even when a canonical `30 Wiki` page ranks higher.
5. If a likely subtree emerges from recent results (for example `00 Inbox/<project>/`), run a scoped tree/search pass inside that subtree before finalizing. Search for narrow terms from the user's clue (e.g. `GitHub Actions`, `Dockerfile`, `GHCR`, Korean equivalents), then `peek` the best candidates and `section` the exact matching heading when possible.
6. Use `peek` on top candidates before reading long pages.
7. Use `section` when only one heading is relevant.
8. For project/document discovery answers, report a short ranked list: likely primary page first, then hub/canonical/background pages. Include why each page matches, not just titles.
9. If the answer depends on graph context, run `backlinks` or `graph_walk` on the page.
10. Answer with concise synthesis and include source paths. Include public/permalink URLs when the environment has a known wiki base URL.

Example source line format:

```text
Source: 30 Wiki/kiwifs/operations.md
URL: https://<wiki-host>/page/30%20Wiki/kiwifs/operations.md
```

When the wiki host is unknown, report paths only and avoid inventing URLs.

## Write and Maintenance Workflow

For any write-like operation:

1. Search first to avoid duplicates.
2. Prefer updating the existing canonical page over creating a sibling page.
3. Preserve YAML frontmatter if present.
4. Use wiki links for durable internal references.
5. If the user gives a path that is not found, do not stop immediately: search by the filename/title and inspect the nearest tree because users may misremember top-level buckets such as `00 Index` vs `00 Inbox`. Continue on the uniquely matching actual path and report the actual changed path clearly.
6. For small targeted list/table additions or “turn this form/list into a table” requests, read the target page first, preserve all supplied values/placeholders, convert only the requested structure, and avoid inventing missing fields. If lint reports missing required frontmatter such as `title`, add the minimal frontmatter needed to pass lint while keeping the document body focused.
7. When the user remembers an implementation pattern but not its location, search both active canonical pages and legacy/archive/provenance notes with several keyword variants; read the best matching section before updating the target page, and add source links to the updated page when the provenance is durable.
8. Use `bulk_write` when multiple files must change together.
9. After writing, run `lint` on changed markdown.
10. Run `health_check` for important pages or indexes.
11. Report exact changed paths and any remaining lint/health issues.

### Subtree promotion / reclassification checklist

When the user asks to move an entire Inbox/source subtree into `30 Wiki`, treat it as a category promotion rather than a simple rename:

1. Inspect the source tree and the target tree first; do not overwrite an existing target category without checking.
2. Move the subtree with link rewriting when the MCP server supports it. If you must use filesystem moves, immediately run a stale-reference search for the old path and update internal links/frontmatter path fields.
3. Normalize frontmatter for promoted pages: set an active/published status appropriate to the vault, add compact promotion provenance such as `promoted_at` and `promoted_from`, and preserve original `derived-from` entries.
4. Polish source/Ingest wording into reader-facing wiki wording. Remove stale phrases like "작업 공간" when the page is now a category, and rename "원본 리소스" style labels if they make the page sound like a temporary staging note.
5. Update the nearest category README plus `30 Wiki/README.md`; add a `30 Wiki/log.md` entry for broad moves.
6. Lint every moved page and every changed index/log page. Fix duplicate heading-anchor warnings by adding contextual prefixes to repeated headings instead of ignoring them.
7. Verify with tree/read/query/health checks, then search for stale old paths. It is acceptable for `promoted_from` or log history to mention the old path; operational links should point to the new location.
8. If the vault has an fswatch/git auto-commit process, check git status after MCP/filesystem edits. Auto-commits can split a promotion into partial commits, so commit any remaining deletions/edits and report the final clean state.

### Concise tutorial/runbook rewrite checklist

When the user says a wiki document is too long, too rhetorical, or too hard to follow, rewrite for task completion instead of adding more explanation:

1. Ask “what is the shortest path the reader must perform?” and make that the outline.
2. Replace broad motivation with a one-paragraph purpose and then concrete steps.
3. Keep only the commands/config snippets needed for the happy path plus verification.
4. Use plain Korean for Korean docs: short section titles, short paragraphs, little 수사.
5. Preserve the user’s requested sequence exactly when they provide one; make each step map to a file, command, UI action, or verification result.
6. See `references/concise-operational-tutorials.md` for the captured pattern.

### Attachment-backed wiki edits

When the user asks to update a wiki page from an attached config/document, treat the attachment content as a prerequisite input, not as optional context:

1. First confirm that the attachment body is actually available in the agent context or readable from the platform/runtime. Do not rewrite the wiki from memory or from the target page alone.
2. If the attachment was skipped, unsupported, empty, or otherwise not retrievable, stop before writing and ask the user to paste the content in a code block or reattach it with a text-friendly extension such as `.txt`.
3. Report the specific ingestion problem briefly, then keep the promised wiki-edit plan ready for when the source content arrives.
4. Only after the source content is present: validate/transform it, update the target KiwiFS page, then run `lint`/`health_check` as usual.

### Light polish / 윤문 checklist

When the user asks to “윤문”, “다듬어”, or lightly polish an existing wiki page, do an in-place readability pass rather than changing the document’s claims or structure:

0. If the request combines light polish with moving a single source or inbox page into a canonical wiki area, treat it as **single-page promotion**: choose a stable reader-facing title, set compact frontmatter appropriate to the vault, write under the nearest canonical category, add a category README/index entry when appropriate, and do **not** delete the source page unless the user explicitly asks. Preserve a polished source's voice, heading rhythm, emphasis, and paragraph/list structure unless the user explicitly asks for a stronger rewrite; do not flatten a good source into generic canonical prose. When a language- or style-specific polishing skill is available, use it and treat the supplied source as the style anchor.
1. Read the target page first and preserve YAML frontmatter, source/provenance, heading hierarchy, timestamp anchors, code blocks, and warning strength.
2. Improve sentence rhythm, awkward phrasing, table formatting, and consistency of Korean/English technical terms without adding unsupported facts.
3. If the page is a lecture/video note, keep verified timestamps exactly as provided; never invent missing MM:SS values during polish.
4. For command/keybinding explanations in Korean wiki notes, prefer scannable definition bullets over prose when the content is a mapping. Example: use ``- `ctrl + b`: 기본 Prefix`` and ``- `Prefix + s`: 세션 목록과 미리보기를 대화형으로 표시`` instead of combining both into a sentence like “`ctrl + b`는 …다. `Prefix + s`는 …다.”
5. In Markdown tables, avoid raw pipe-key notation that can break table parsing. If a keybinding includes `|`, either escape it correctly or write a readable label such as `Prefix + 파이프 키`, then verify with `lint`.
6. Update `updated_at` when the page has that field and the edit is substantive.
7. Write back through KiwiFS, then run `lint` and usually `health_check`; report path/URL plus validation results.

When the user asks whether an external GitHub/open-source project can satisfy a planned workflow, deployment, or provider-routing need and asks to document the finding:

- Inspect implementation evidence, not only README claims. Prefer repository code/docs paths, function names, RPC names, env vars, and config precedence rules.
- When the user's follow-up is about whether a capability is configurable in the UI, verify the actual screen/component wiring and any supplied screenshot; distinguish backend support from Settings/onboarding exposure.
- Start the wiki page with a decision-first conclusion (`가능/제한적/불가능`) and a small layer table before long setup steps.
- Separate capability layers such as hosted core/server, UI access model, credential storage, provider routing, OAuth/API-key semantics, and persistence requirements.
- For deployment docs, split host/server steps from local client/desktop steps and include copyable commands plus verification.
- Preserve concrete source paths for future re-checking, but never write secrets, OAuth codes, bearer tokens, API keys, cookies, or auth-profile contents.
- If the implementation uses an OAuth/subscription path that is not equivalent to official API-key access, call out policy/stability uncertainty explicitly.
- `references/external-repo-feasibility-docs.md` — decision-first wiki writeups for external repo feasibility investigations: inspect implementation evidence, separate capability layers, document UI exposure vs backend support, split host/client deployment steps, and preserve source paths without secrets.
- `references/cron-backed-operational-forms.md` — when a wiki page is the control record for a live recurring job, update both the scheduler and the page together; verify job ID, cadence, delivery target, and lint/health state before reporting.
- `references/live-ops-runbook-continuation.md` — when a live chat session is actively changing infrastructure and the user asks to keep the wiki updated, continue the existing canonical runbook with dated reproducible sections, exact commands, verification output, and durable pitfalls while keeping secrets out.
- `references/discord-answer-to-kiwifs-guide.md` — when saving a useful Discord/chat answer as a KiwiFS guide, honor the requested path/title, convert the answer into a standalone operational page with frontmatter and copyable commands, include an early terminology table when helpful, then lint/health-check and report the URL-encoded wiki link.
- `references/live-mesh-service-runbook-updates.md` — when a chat session becomes a live Mesh/VPN service setup, update the existing system runbook with reproducible final state, host-vs-container DNS boundaries, bind/firewall prerequisites, warmup evidence, and lint/health verification. Before documenting host commands, verify that wrapper targets such as `make up-force` actually exist; if a mistaken command is written, append/patch a correction immediately and re-run lint/health.
- `references/live-ops-setup-to-canonical-runbook.md` — when a live operations session turns a plan into a verified setup, update the existing canonical runbook with a dated confirmed-setup section, copyable repro commands, real verification criteria, security boundaries, lint/health checks, and only durable non-secret facts.

When the user asks to save material from the current chat as a wiki document:

- Treat the immediately preceding answer or agreed content as the source unless the user names a different scope.
- Do a lightweight search for an existing canonical page before creating a new one.
- Choose a stable, topic-oriented path rather than a session-log path; pick the nearest topical `30 Wiki` category yourself when the fit is obvious instead of asking the user to choose.
- Normalize obvious spelling/branding mistakes in the requested filename/title when the intended canonical name is clear (for example `Archemy` → `Alchemy`), while preserving the user's meaning.
- Add compact YAML frontmatter (`title`, `doc_type`, `topic`, `tags`, `status`, `updated_at`, `source_context`) when the page is standalone teaching/tutorial material.
- Preserve the useful teaching structure from the chat output, including early terminology tables when present.
- For request-information/checklist pages, include a copyable fill-in template, console/UI paths for obtaining each value, and an explicit “do not share” secrets section. See `references/operational-checklist-docs.md`.
- Update the nearest category README/index when adding a discoverable canonical page.
- When updating existing README/index pages, prefer minimal insertion/patch. If you full-rewrite after a KiwiFS write, re-read first and preserve server-added frontmatter/provenance; do not retype unrelated links from memory because this can break existing navigation.
- Never build an index replacement from a truncated or summarized tree/read result. For category READMEs, either patch a small insertion into the live file or reconstruct from git history plus the new entry, then verify that unrelated existing entries remain present.
- If an index was accidentally clobbered, recover immediately: inspect the prior version, restore the full previous README with only the intended entry added, then lint and search for the new entry before reporting success.
- Verify with `lint` and usually `health_check`, then report path, absolute URL, and validation results.

Use `append` for logs, journals, changelogs, and run records where ordering matters. Use `write` only when replacing the full page intentionally.

## Kanban and Workflow Control

KiwiFS can model Kanban boards through workflow definitions and page frontmatter state. Prefer native workflow tools over ad-hoc markdown tables.

Typical flow:

1. Inspect available workflows with `workflow_list`.
2. Read the workflow definition with `workflow_get`.
3. Show board state with `workflow_board`.
4. Move a page using `workflow_advance` instead of editing state by hand.
5. Create or update workflow definitions with `workflow_save` only when the user asks to change board structure.

Pitfalls:

- Do not create a saved view or computed markdown page when the user asked for a Kanban workflow.
- Respect terminal states; some workflows intentionally prevent moving out of completed/archived states.
- If a task page is claimed, use `claims_list` and `release`/`claim` semantics instead of overwriting another agent's active work.

## Task Claiming Workflow

For multi-agent or queue-like work:

1. Use `eligible` to find unclaimed, unblocked, todo tasks.
2. Use `claim` before making non-trivial edits to a task page.
3. Keep the lease duration realistic.
4. Release the claim when done or when stopping work.
5. If claim fails, do not work the task unless the user explicitly overrides the coordination rule.

## Draft Workflow

Use draft spaces when a batch of changes needs review before publication:

1. `draft_create` to create an isolated workspace.
2. `draft_write` and `draft_read` within the draft.
3. `draft_diff` for review.
4. `draft_merge` only after approval or when the task explicitly permits merging.
5. `draft_discard` for abandoned drafts.

## Import, Clip, and Ingest

Use these when turning external material into wiki pages:

- `clip`: web article/page to markdown with extracted content.
- `ingest`: document files such as PDF, DOCX, PPTX, Excel, HTML, EPUB.
- `import`: structured sources such as CSV, JSON, JSONL, SQLite, Postgres, MySQL, MongoDB, Firestore, Notion, or Airtable.

After import/ingest:

1. Inspect output paths.
2. Run `lint` or `analytics` as appropriate.
3. Add or suggest wiki links from canonical index pages.
4. Avoid dumping private credentials or unrelated raw data into pages.

## Graph and Discovery Workflows

Use graph tools when the user asks for relationship, impact, discoverability, or navigation:

- `backlinks`: who references this page.
- `graph_walk`: one-hop neighborhood with siblings and hub score.
- `graph_path`: shortest path between two pages.
- `graph_centrality`: important/connective pages.
- `graph_communities`: natural topic clusters.
- `graph_analytics`: broad graph health and hubs.
- `suggestions`: semantically related pages not already linked.

For link-maintenance tasks, combine graph/backlink tools with `rename` so links are rewritten atomically when supported.

## Analytics and Quality

Use analytics tools for maintenance and reporting:

- `analytics`: total pages, health metrics, stale/orphan/broken-link counts, recent updates.
- `health_check`: page-level issues and quality score.
- `lint`: markdown/frontmatter/table/fence/heading/mermaid issues.
- `velocity`: hot/cold spots and authorship patterns.
- `timeline`/`feed`/`changes`: recent activity.
- `memory_report`: episodic pages not yet consolidated into concept pages.
- `eval`: retrieval quality evaluation using expected pages.

## Canvas, Views, and Queries

KiwiFS may support visual canvases and saved views in addition to markdown pages.

- Use `canvas_*` when the user asks for a visual graph/canvas or node-edge map.
- Use `views_*` for saved query definitions and reusable bases/views.
- Use `query` for DQL-style page queries.
- Use `query_meta` for frontmatter-specific filtering.
- Use `aggregate` for counts, averages, sums, min/max grouped by a frontmatter field.
- Use `view_refresh` only for computed markdown views that explicitly advertise themselves as generated views.

Do not confuse saved views with workflows: Kanban state changes belong to `workflow_*` unless the user explicitly asks for a view definition.

## Git and Versioning Awareness

KiwiFS writes are usually git-backed. Prefer KiwiFS operations over raw filesystem writes because they preserve application invariants and commit history.

Use:

- `versions` to inspect a page's history.
- `changes`, `timeline`, or `feed` for recent activity.
- `rename` instead of raw move when links should be rewritten.
- `delete` instead of filesystem removal so history is preserved.

If remote backup/push status matters, inspect the deployment's KiwiFS operational docs or logs; do not assume local git commits and remote backup pushes are the same mechanism.

## URL Reporting

When the environment defines a canonical wiki base URL, report page links as absolute URLs. URL-encode spaces and non-ASCII path segments.

```text
https://<wiki-host>/page/<url-encoded-path>.md
```

If the user has given a preferred host in the current context, use it. Otherwise, avoid guessing and report the relative path.

## Safety Rules

- Search before writing.
- Prefer canonical page updates over duplicate new pages.
- Use `bulk_write` for related multi-file changes.
- Run lint/health checks after edits.
- Do not store secrets, bearer tokens, cookies, private keys, or webhook URLs.
- Do not bypass task claims in multi-agent workflows.
- Do not mutate workflow definitions, delete pages, merge drafts, or bulk-import data unless that is clearly within the requested scope.
- For public/open-source docs, do not publish private wiki URLs or internal workspace paths.

## Verification

Before finishing, verify the operation that matches the task:

- Search answer: relevant pages were searched and at least one source was read or peeked.
- Write/update: changed page paths are known, lint ran, and health issues are reported.
- Kanban/workflow: board or target page state was checked after `workflow_advance`.
- Import/ingest/clip: created paths were inspected and obvious lint/indexing issues were handled.
- Graph task: backlinks/graph output was used, not guessed.
- Config task: MCP server/tool availability was checked after config changes, usually by restarting/reloading the agent session if required.
