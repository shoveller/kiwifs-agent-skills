# External repository feasibility notes → KiwiFS page

Use this reference when a user asks whether a GitHub/open-source project can satisfy a planned workflow or deployment need and asks to leave the answer in the wiki.

## Pattern captured

1. **Inspect implementation, not only README.** Clone or fetch the repository and identify the code paths that implement the claimed capability. For OpenHuman, the useful evidence was split across docs, backend Rust, and frontend onboarding/settings UI.
   - For UI-driven questions, inspect both the backend capability and the actual visible screen/component that should expose it. Do not infer “user can configure this in Settings” just because RPCs exist; search for the provider-specific action in the active panel and compare with screenshots when supplied.
2. **Separate capability layers.** For hosting/model-provider questions, answer each layer independently:
   - Can the server/core be hosted?
   - Is the UI officially supported remotely, or only local/desktop against a remote core?
   - How are credentials stored and selected?
   - Does the provider path use official API keys, OAuth, local routing, or managed backend?
   - What is the fallback/precedence order?
3. **Write a decision-first wiki page.** Start with `가능/불가능/제한적` and a small table of layers. Then include evidence paths, deployment steps, verification, and caveats.
4. **Avoid credential leakage.** Never quote API keys, bearer tokens, OAuth tokens, callback codes, auth profile contents, or connection strings. Replace any credential-like values with `[REDACTED]` or placeholders.
5. **Preserve concrete implementation evidence.** Include repository file paths and function/RPC names so future agents can re-check upstream changes.
6. **Separate host vs client steps.** For Korean operational docs, explicitly divide “서버/host에서 할 일” and “로컬 client/desktop에서 할 일” when deployment spans both sides.
7. **Flag policy/semantics uncertainty.** If a project uses an OAuth/subscription path that is not the same as an official API key, explain that it may work but is not equivalent to officially billed API usage and may be brittle.
8. **Verify the wiki artifact.** Run KiwiFS lint after writing and report the absolute wiki URL plus validation result.

## Suggested page shape

```markdown
---
title: <Project> <capability> 배포/운영 가이드
tags: [project, provider, self-hosting]
status: draft
source:
  - <repo URL>
---

# <Title>

## 결론

**가능/제한적/불가능.** One-paragraph answer.

| 층 | 가능 여부 | 설명 |
|---|---:|---|
| Server/core hosting | 가능 | ... |
| UI access | 제한적 | ... |
| Provider/OAuth/API usage | 가능하나 주의 | ... |

## 확인한 구현 근거

- `<path>` — function/constant/RPC names and why they matter.

## 권장 배포 방식

### 서버에서 할 일
...

### 클라이언트에서 할 일
...

## 운영 체크리스트
...

## 주의점과 리스크
...

## 참고 경로
...
```

## OpenHuman-specific evidence captured in session

Repository: `https://github.com/tinyhumansai/openhuman`

Key paths inspected:

- `gitbooks/features/cloud-deploy.md` — remote core deployment model: `openhuman-core serve` exposes `/health` and bearer-protected `/rpc`; supported remote pattern is core remote + UI local.
- `docker-compose.yml` — persists `/home/openhuman/.openhuman` via `openhuman-workspace` volume and exposes core port 7788.
- `.env.example` — core variables include `OPENHUMAN_CORE_TOKEN`, `OPENHUMAN_CORE_RPC_URL`, `OPENHUMAN_WORKSPACE`, `OPENHUMAN_CORE_ALLOWED_ORIGINS`.
- `src/openhuman/inference/openai_oauth/config.rs` — Codex/ChatGPT OAuth config; redirect URI `http://127.0.0.1:1455/auth/callback`.
- `src/openhuman/inference/openai_oauth/flow.rs` — `start_openai_oauth`, `complete_openai_oauth`, `openai_oauth_status`, `disconnect_openai_oauth`.
- `src/openhuman/inference/openai_oauth/store.rs` — stores OAuth profile under provider key `provider:openai`, profile `oauth`, and refreshes expiring tokens.
- `src/openhuman/inference/provider/factory.rs` — API key lookup first, then OpenAI OAuth fallback only for slug `openai`.
- `app/src/pages/onboarding/steps/ApiKeysStep.tsx` — desktop onboarding calls OAuth status/start/complete and instructs the user to paste the loopback callback URL.
- `app/src/services/api/aiSettingsApi.ts` — cloud providers, workload routing, API key storage, and local provider facade.
- `app/src/components/settings/panels/AIPanel.tsx` — AI settings panel for cloud providers/local provider/routing. Important UI nuance: it has a generic `oauthAction` slot, but in the inspected version only `keyDialogFor === 'openrouter'` wires OAuth in Settings; OpenAI in Settings shows API-key UI only. The OpenAI/ChatGPT OAuth UI exists in onboarding (`ApiKeysStep.tsx`), not the Settings LLM Provider screen.
- GitHub issue/PR trail: issue `#1953` requested OpenAI LLM Provider OAuth, and PR `#2265` (`inference: oauth (chatgpt-style) for openai llm provider`) was merged. Treat a merged support PR as implementation evidence, not product-availability proof; still verify release version, routes, and visible Settings/onboarding wiring.

OpenHuman conclusion: hosting the core and using ChatGPT/OpenAI OAuth is feasible at the backend/core level, but treat it as a Codex/ChatGPT OAuth path, not as proof that normal OpenAI API access is included in a ChatGPT subscription. Persistent workspace storage is required because OAuth profiles live with the remote core state. When answering “can I configure it in the UI?”, distinguish backend support from product UI exposure: in the inspected 0.54.15-style flow, OpenAI OAuth RPCs and onboarding code exist, but the Settings LLM Provider screen may still show only API-key UI unless an `oauthAction` branch for `openai` is wired and the callback-URL flow is reachable.