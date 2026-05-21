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
3. Use `peek` on top candidates before reading long pages.
4. Use `section` when only one heading is relevant.
5. If the answer depends on graph context, run `backlinks` or `graph_walk` on the page.
6. Answer with concise synthesis and include source paths. Include public/permalink URLs when the environment has a known wiki base URL.

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
5. Use `bulk_write` when multiple files must change together.
6. After writing, run `lint` on changed markdown.
7. Run `health_check` for important pages or indexes.
8. Report exact changed paths and any remaining lint/health issues.

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
