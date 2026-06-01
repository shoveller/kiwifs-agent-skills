# Operational checklist docs in KiwiFS

Use this reference when a user asks to turn a chat answer into a reusable operational checklist or request-information page in the wiki.

## Pattern

1. Search first for an existing canonical page in the relevant category. If there is an adjacent guide, create a companion checklist only when the new page has a distinct job.
2. Put the page near the operational topic, not in a session-log path. For infrastructure/cloud runbooks, `30 Wiki/infra/` is usually preferable to Inbox.
3. Include an early terms table when the task introduces provider-specific identifiers or credentials. For OCI-style request pages, distinguish API signing keys from SSH keys and OCIDs from display names.
4. Add a fill-in template block the user can copy, with secrets clearly marked as separate/safe-channel material.
5. Explicitly list what not to share: passwords, MFA codes, browser cookies, card data, SSH private keys, and other unnecessary secrets.
6. Include console/UI paths and official source links when the user needs to gather data from a web console.
7. Update the nearest category README/index with a short discoverability entry.

## Editing pitfall

When updating an existing README/index, prefer a minimal patch/insertion. If a full rewrite is necessary, read the current file after any KiwiFS write because the server may add provenance/frontmatter. Preserve that frontmatter and avoid retyping unrelated links; a single typo in an existing index link can silently break navigation even when the new page is correct.

## Verification

- Lint the new page and the index.
- Run a page health check on the new standalone page.
- Report the absolute wiki URL using the known base URL and URL-encoded path.
