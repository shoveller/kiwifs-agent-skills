# KiwiFS task orchestration MCP tools

Use this when implementing or reviewing KiwiFS MCP tools around agent task orchestration, especially issues related to `kiwi_task_create`, `kiwi_task_progress`, task workflows, claims, and decomposition.

## Source hierarchy

When a GitHub issue gives a narrow parameter list, do not treat it as the whole orchestration model. Cross-check:

1. The issue body for acceptance criteria.
2. The UC-1 Agent Task Orchestration wiki page for the intended Symphony-like control-plane model.
3. Adjacent issues that define the shared schema or convention, especially the default task workflow/template and progress protocol.
4. Existing code in `internal/mcpserver/`, `internal/workflow/`, and `internal/claims/`.

## Canonical task schema cues

The UC-1 direction and default task-template issue point to this frontmatter shape:

```yaml
---
title: ""
workflow: tasks
state: backlog
assignee: ""
priority: 3
due_date: ""
blocked_by: []
labels: []
parent: ""
artifacts: []
---
```

Important implications:

- Prefer KiwiFS terminology `blocked_by` over external names like `dependsOn` unless explicitly adding aliases.
- Include `parent` for child/subtask creation; it is part of the intended decomposition model.
- Do not store `children` redundantly in task frontmatter by default. Treat children as a derived reverse lookup of pages where `parent == <current path>`, or leave bulk child creation to a later `kiwi_decompose`-style tool.
- Default state should follow the task template (`backlog`) unless the specific workflow/template has changed or the issue explicitly requests another initial state.
- `artifacts` belongs in the schema for PRs, branches, logs, or other outputs linked to a task.

## Review/implementation pitfall

For `kiwi_task_create`, a minimal issue body may list only `title`, `description`, `assignee`, `priority`, `blocked_by`, `labels`, and `claim`. Before coding, decide whether compatibility with the shared task schema requires adding `due_date`, `parent`, and `artifacts` as optional parameters. If you keep the issue-minimal surface, document why and ensure generated frontmatter still includes the canonical fields with defaults.

## Testing expectation

At minimum, add an MCP integration-style test that:

1. Calls the task creation tool.
2. Reads the created page with `kiwi_read` or backend `ReadFile`.
3. Verifies the returned path and frontmatter fields.
4. Covers dependency/decomposition fields (`blocked_by`, `parent`) and optional arrays (`labels`, `artifacts`) if they are accepted by the tool.
5. If `claim=true` is supported, verifies the claim through the existing claims backend/listing rather than only checking the text response.
