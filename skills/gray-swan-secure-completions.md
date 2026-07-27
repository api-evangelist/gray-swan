---
name: gray-swan-secure-completions
description: >-
  Send an OpenAI-compatible chat completion through Gray Swan Cygnal so it is
  filtered for jailbreaks/prompt-injection and monitored against an enforcement
  policy, and handle policy-violation refusals correctly.
api: Gray Swan AI (Cygnal)
source: https://docs.grayswan.ai/cygnal/creating-completions
operations:
  - cygnal_completions_cygnal_v1_chat_completions_post
  - cygnal_monitor_cygnal_monitor_post
generated: '2026-07-19'
method: generated
---

# Secure completions through Cygnal

Cygnal is a drop-in proxy: point your existing OpenAI client at
`https://api.grayswan.ai/cygnal` and it filters inputs/outputs and monitors for
threats before/after the model runs.

## Steps

1. **Authenticate.** Set the `grayswan-api-key` header to your platform key
   (get one at https://platform.grayswan.ai).
2. **Select a policy — required.** Set `policy-id` (or `agent-id` for an agent
   with an associated policy). Cygnal applies **no** default policy; omitting
   both returns `400 No policy specified`.
3. **Call the OpenAI-compatible endpoint** — `POST /cygnal/v1/chat/completions`
   (operationId `cygnal_completions_cygnal_v1_chat_completions_post`) with a
   normal chat completions body (`model`, `messages`, optional `tools`).
4. **Tune strictness (optional)** via threshold headers `pre-violation`,
   `post-violation`, `pre-jailbreak`, `post-violation-jb` (0.0–1.0; lower =
   stricter). Attach ad-hoc rules with `rule-<name>` / `cygnal-rule-<name>`.
5. **Handle violations.** A blocked request returns **HTTP 200** with
   `finish_reason: "violation"` and a refusal message
   (`"Sorry, I can't help with that."`). Treat `finish_reason == "violation"`
   as a block, not a normal completion — do not retry blindly.
6. **Monitor-only (optional).** To evaluate messages/tools WITHOUT proxying the
   model, use `POST /cygnal/monitor`
   (operationId `cygnal_monitor_cygnal_monitor_post`).

## Errors

- `400` — malformed request or missing policy selector.
- `401` — missing/invalid `grayswan-api-key`.
- `422` — body failed validation (see `detail[].loc` / `detail[].msg`).
- `502` — upstream provider error. Retry with backoff.
- Error envelope: `StandardErrorResponse { error, message, detail, error_code, details }`.
  See `errors/gray-swan-problem-types.yml`.

## Limits

Free tier: 200,000 tokens/min, 10,000,000 tokens/day. Over quota, Cygnal still
proxies to the provider but **without** filtering — check quota before relying
on protection. See `rate-limits/gray-swan-rate-limits.yml`.
