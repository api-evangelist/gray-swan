---
name: gray-swan-manage-policies
description: >-
  Create, version, export, and import Cygnal enforcement policies via the Gray
  Swan AI API so the rules that filter your LLM traffic are managed as code.
api: Gray Swan AI (Cygnal)
source: openapi/gray-swan-cygnal-openapi.json
operations:
  - get_policies_policies_get
  - create_policy_policies_post
  - get_policy_by_id_policies__policy_id__get
  - update_policy_policies__policy_id__put
  - get_policy_versions_policies__policy_id__versions_get
  - import_policy_policies_import_post
generated: '2026-07-19'
method: generated
---

# Manage Cygnal enforcement policies

Policies are the rule sets Cygnal enforces on your proxied LLM traffic. They are
versioned and portable (export/import), so you can manage them as code.

## Steps

1. **Authenticate** with the `grayswan-api-key` header.
2. **List existing policies** — `GET /policies`
   (operationId `get_policies_policies_get`).
3. **Create a policy** — `POST /policies`
   (operationId `create_policy_policies_post`) with a `PolicyCreateRequest`
   body (name + rule/threshold definitions).
4. **Fetch one** — `GET /policies/{policy_id}`
   (operationId `get_policy_by_id_policies__policy_id__get`).
5. **Update it** — `PUT /policies/{policy_id}`
   (operationId `update_policy_policies__policy_id__put`). Each save creates a
   new version.
6. **Review history** — `GET /policies/{policy_id}/versions`
   (operationId `get_policy_versions_policies__policy_id__versions_get`); fetch
   a specific revision with `GET /policies/{policy_id}/versions/{version_id}`.
7. **Promote across environments** — export with
   `GET /policies/{policy_id}/export` and load elsewhere with
   `POST /policies/import` (operationId `import_policy_policies_import_post`) or
   `POST /policies/bulk-import`.
8. **Use it at inference time** by passing the `policy-id` header to any
   `/cygnal/*` completion (see `gray-swan-secure-completions`).

## Errors

- `401` invalid key · `403` no access to that policy · `404`-class via
  `StandardErrorResponse` · `422` invalid body. See
  `errors/gray-swan-problem-types.yml`.

## Notes

- No idempotency key is supported; updates are versioned instead
  (`conventions/gray-swan-conventions.yml`).
