# Live Mesh Service Runbook Updates

Use this pattern when a chat session turns into a live operational setup for a private Mesh/VPN service and the user wants the wiki kept current while the work proceeds.

## What to capture

- The final reproducible state, not only the original plan.
- Exact service names, task names, scripts, bind addresses, model names, and verification commands.
- Security boundaries: localhost-only bind, Mesh-only exposure, firewall allowlist, key-only SSH, and what must not be stored.
- Container vs host name-resolution boundaries. Host `/etc/hosts` does not automatically update Docker container `/etc/hosts`; Compose `extra_hosts` or container DNS config is required.
- Consumer-side repo/config changes separately from provider-host changes. For example, prepare Compose/env/application config first, then list the host actions still needed on the Mesh provider node.
- Direct-call prerequisites for Mesh services:
  1. caller resolves a stable service name to the Mesh IP;
  2. every actual caller context resolves it too (host shell, target containers, profile-specific gateways);
  3. provider service binds to a Mesh-reachable address, not only `127.0.0.1`;
  4. provider firewall allows only intended Mesh callers/ports;
  5. validation runs from the actual caller context, ideally inside the target container.
- For Docker Compose consumers, capture both declarative config and rendered proof: service-level `extra_hosts`, app endpoint env vars, app config files (for example `config.toml`), any init script/templates that regenerate those files, and `docker compose --env-file ... --profile ... config --quiet` output.
- When LLM/embedding services move to another Mesh node, update the model names and OpenAI-compatible `/v1` endpoints consistently across all consumers (for example API LLM base URL, embedding base URL, KiwiFS vector base URL, and generated config defaults), then verify rendered config before runtime rollout.
- Warmup/cold-start behavior as an operational section when LLM/embedding services are involved: warmup script, scheduled task/timer, keep-alive, logs, and `ps`/status evidence.
- Layered verification when Docker/root access is unavailable: check published localhost ports and health endpoints from the host shell, test upstream Mesh provider APIs directly, and use the application/MCP surface (for example KiwiFS `embeddings`) to prove the active model/dimensions. Do not mark a dependent service healthy merely because its port is open; `connection reset` on health/MCP/root paths means the next evidence needed is container logs from an account with Docker access.

## Workflow

1. Search for an existing canonical system/runbook page first.
2. Append or patch a timestamped section for live findings, but avoid turning the page into an unreadable chat log.
3. For each live change, record the reproducible command block and a short measured result.
4. If the work has two operational sides, maintain two explicit blocks in the wiki: **already changed in repo/config** and **host/provider actions still required**. This matches the user's preference to receive host-required work separately and prevents overstating readiness.
5. If a first attempt fails because the current state differs from the assumption (for example model renamed), capture the durable lesson: verify installed names with the service API before hard-coding warmup targets.
6. For LLM/embedding consumers, verify both halves independently before blaming the integration: provider `/api/tags`, OpenAI-compatible `/v1/models`, `/v1/chat/completions`, `/v1/embeddings`, then the consumer health/MCP/API endpoint. This distinguishes “provider reachable” from “consumer initialized successfully.”
7. When the user asks whether a restarted service is “working,” report a three-state result instead of a binary if needed: upstream provider OK, intermediary/app OK, dependent app failing/unknown. Include the exact blocker and the next host-side command, usually `docker compose ... logs --tail=... <service>` when socket access is unavailable.
8. After writes, run lint and health checks and report both.

## Pitfalls

- Do not claim a service is callable from other Mesh nodes just because a hostname resolves. Bind address and firewall policy are separate requirements.
- Do not assume host-level `/etc/hosts` affects app containers.
- Do not expose local inference APIs on `0.0.0.0` without an explicit firewall/ACL decision.
- Keep secrets, private keys, tokens, passwords, cookies, and bearer strings out of the wiki.
