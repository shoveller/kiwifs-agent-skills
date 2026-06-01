# Upstream KiwiFS issue surface triage

Use this when evaluating a KiwiFS GitHub issue before recommending an implementation target or first contribution.

## Checklist

1. Read the issue body and labels, but do not treat them as the whole spec.
2. Read the linked wiki/use-case page when present; KiwiFS issues often reference roadmap-level concepts that clarify whether a change is CLI, REST API, MCP, UI, docs, or template work.
3. Inspect adjacent/related issues in the same use-case label to identify sequencing dependencies. Example: task orchestration issues #145/#148/#149 imply schema/template -> task creation -> progress helper ordering.
4. Inspect code surfaces separately:
   - CLI: `cmd/*.go`
   - REST API: `internal/api/handlers_*.go`
   - MCP: `internal/mcpserver/mcpserver.go`
   - UI source picker/wizard: `ui/src/lib/importSourceLabels.ts` and related UI components
   - Core implementation: `internal/<domain>/`
5. Report which surfaces are actually wired today, not just what docs or roadmap claim.
6. For first-contribution recommendations, separate:
   - test-only changes: low upstream risk, little/no operational impact
   - docs/template/init changes: low-medium risk, some operational utility
   - MCP/API/UI feature changes: higher operational utility, higher design/review risk

## Captured example: Google Sheets importer

A Google Sheets importer may exist in `internal/importer/gsheets.go` and be reachable from CLI via `cmd/import.go` (`kiwifs import --from gsheets ...`) while not being exposed in the UI, REST API, or MCP import tool.

Observed surface split:

- CLI: `--from gsheets`, `--spreadsheet-id`, `--sheet`, `--credentials`
- UI: `ui/src/lib/importSourceLabels.ts` may keep `gsheets` commented as a future source
- REST API: `internal/api/handlers_import.go` may lack request fields such as `spreadsheet_id`/`sheet` and omit `gsheets` from `buildBuiltinSource`
- MCP: `kiwi_import` registration/buildMCPSource may omit `gsheets`

Implication: an issue that says “add test for Google Sheets importer” is likely test coverage for a CLI-only connector, not a UI/API/MCP feature. Do not imply operational UI impact unless those surfaces are wired.