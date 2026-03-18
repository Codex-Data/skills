---
name: codex-gateway
description: >-
  Machine Payment Protocol (MPP) for keyless, pay-per-query access to the Codex
  Supergraph GraphQL API. Use when the user has no API key and wants to pay per
  query via the 402 challenge flow at https://graph.codex.io/graphql.
metadata:
  author: codex-data
  version: "1.0"
---

# Codex Machine Payment Protocol (MPP)

Use this skill to access the Codex Supergraph without an API key via the MPP challenge flow.

|                       |                                                                 |
| --------------------- | --------------------------------------------------------------- |
| HTTP endpoint         | `https://graph.codex.io/graphql`                                |
| Opt-in header         | `X-Codex-Payment: mpp`                                         |
| Credential header     | `Authorization: Payment <base64url-credential>`                 |

## How it works

1. Send a GraphQL query with `X-Codex-Payment: mpp` (no credential).
2. Server returns `402 Payment Required` with `WWW-Authenticate: Payment ...` challenges.
3. Client solves one challenge and retries with both `X-Codex-Payment: mpp` and `Authorization: Payment <credential>`.
4. Server returns GraphQL data + `Payment-Receipt` header.

## Constraints

- **Query only.** Mutations and subscriptions return `403` in MPP mode.
- If a valid API key or bearer token is also present, API auth takes precedence.

## Rules

- Never print raw credentials.
- Only use MPP for `query` operations.
- For available GraphQL operations and endpoint selection heuristics, see the `codex-supergraph` skill.

## References

| File | Purpose |
| ---- | ------- |
| [rules/wallets.md](rules/wallets.md) | Wallet setup: tempo wallet/request (Tempo) and awal (Base) |
| [references/mpp-flow.md](references/mpp-flow.md) | Auth matrix, challenge details, error codes |
