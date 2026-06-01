# Concise operational tutorial rewrites

Session-derived guidance for rewriting long KiwiFS tutorial/runbook pages when the user says the document is too verbose or rhetorical.

## Preferred shape

- Start from the user’s desired flow, not from a long conceptual introduction.
- Use plain language and imperative steps.
- Keep scope narrow: one path through the task, then optional notes only if they prevent common mistakes.
- For Korean user-facing docs, prefer short section titles and short paragraphs.
- Remove sales-like phrasing, extended metaphors, and repeated motivation.
- Preserve copyable commands/config snippets that make the guide actionable.
- End with a concise “what to verify” section.

## Example flow captured from Cloudflare custom container guide cleanup

When explaining a custom Cloudflare container project, the user wanted exactly this sequence:

1. Scaffold the simplest container project with `c3`.
2. Customize that project’s `Dockerfile`.
3. Avoid local image builds by using GitHub Actions to build and push the image to Cloudflare Registry.
4. Configure Cloudflare to pull and run the image from Cloudflare Registry.

The reusable lesson is not Cloudflare-specific: for operational docs, identify the shortest practical path and make each step map to a visible artifact the reader edits or verifies.