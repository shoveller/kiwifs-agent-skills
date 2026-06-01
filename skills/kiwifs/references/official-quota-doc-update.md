# Updating wiki runbooks from official quota/resource docs

Use this pattern when a wiki page has an operational plan but the user asks for more detailed limits, quotas, free-tier boundaries, storage sizing, or similar provider facts.

## Workflow

1. Read the current canonical wiki page(s) first, plus any checklist/request-form page that will collect user inputs.
2. Search/extract the provider's official docs before editing. Prefer the vendor docs over blogs/forums; use third-party sources only as hints.
3. Convert provider quota language into operational choices, not just copied limits. For OCI Always Free A1.Flex, the useful shape is:
   - compute: `VM.Standard.A1.Flex`, 4 OCPU / 24GB RAM aggregate, often described as 3,000 OCPU-hours and 18,000 GB-hours/month;
   - storage: Always Free Block Volume total 200GB across boot volumes and extra block volumes;
   - boot volume defaults/minima may be described as ~47GB or 50GB depending on Oracle page/console, so tell the reader to verify the current console value;
   - practical layouts: boot 50GB + data block volume 150GB, or single boot volume 200GB.
4. Update both the explanatory runbook and the user-input/request checklist when the new fact changes what the user must decide or provide.
5. Add stop conditions for quota/charging risk. Example: if free storage would exceed 200GB, stop and report instead of continuing automation.
6. Update metadata review dates when present, preserve provenance, and add official source links.
7. Lint every changed page and fix duplicate heading slugs rather than reporting avoidable warnings.

## Pitfalls

- Do not treat “SSD capacity” as only the instance shape; in OCI it is usually boot/block volume sizing from the Always Free Block Volume allocation.
- Do not leave the request checklist CPU/RAM-only after adding storage requirements to the runbook. The future automation needs fields like `boot_volume_size_gb` and `extra_block_volume_size_gb`.
- Do not quote sensitive filled-in forms back to the user when discovery finds partial OCIDs or keys; report the path and summarize safely.
