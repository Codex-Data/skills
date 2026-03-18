---
name: codex-gateway
description: >-
  Use when the user wants to query the Codex Supergraph but $CODEX_API_KEY is
  not set. Pays per query via the MPP 402 challenge flow. Only supports queries,
  not mutations or subscriptions.
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
- **Before constructing any query**, read `references/query-templates.md` below for the correct GraphQL schema. Do not guess query or field names.

## References

| File | Purpose |
| ---- | ------- |
| [../codex-supergraph/references/query-templates.md](../codex-supergraph/references/query-templates.md) | **GraphQL query schema and examples — read before constructing queries** |
| [../codex-supergraph/references/gotchas.md](../codex-supergraph/references/gotchas.md) | Common query failure points |
| [references/gotchas.md](references/gotchas.md) | MPP-specific failure points |
| [rules/wallets.md](rules/wallets.md) | Wallet setup: tempo wallet/request (Tempo) and awal (Base) |
| [references/mpp-flow.md](references/mpp-flow.md) | Auth matrix, challenge details, error codes |
