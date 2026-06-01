# Cron-backed operational forms

Use this reference when a KiwiFS page is not just documentation, but the control record for a live recurring job, watchdog, polling script, or automation policy.

## Trigger

A user asks to change wording such as:

- "크론잡을 추천안으로 수정"
- retry interval / cadence / jitter / throttling policy
- success/failure alert target
- stop vs retry behavior for capacity, unknown, quota, auth, billing, or storage errors
- status table for an existing scheduled job

## Procedure

1. Read the wiki page first and identify the live-job fields it claims: job ID, schedule, script path, delivery target, success/fatal state files, retry/stop policy.
2. Inspect the scheduler by listing jobs; never guess the job ID from the document alone.
3. Update the actual scheduled job first when the user asked to change the cron/job behavior.
4. Then update the wiki page so it matches the live scheduler exactly:
   - job ID
   - job name
   - schedule string
   - script path
   - delivery/alert target
   - retryable errors
   - fatal/stop errors
   - next run time if useful
5. Validate the wiki page with `lint`; run `health_check` for important operational docs.
6. Verify scheduler state again after the update and report both the live scheduler value and the wiki path/URL.

## Policy-shaping notes

For OCI A1/Flex capacity-retry style jobs, a conservative cadence such as `every 25m` is safer than very frequent polling. Treat explicit `Out of host capacity` as retryable. Plain `Unknown`/`Internal` errors may be treated as retryable only when no quota/auth/parameter/billing/storage-risk marker is present. Quota, auth, permission, invalid-parameter, service-limit, and storage-cost-risk errors should stop and report.

## Pitfalls

- Do not only edit the wiki when the user's wording refers to the cron job; the live scheduler must be changed too.
- Do not only update the scheduler; stale operational docs cause future agents to make the wrong change.
- Do not assume a 20-minute or 25-minute interval is universally correct; preserve the user's chosen policy and document why it was chosen.
- Do not persist secret values in the page. Use paths or "shared through secure channel" language for private keys and tokens.
