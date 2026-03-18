# MPP Header Rule

Every request to the Codex Supergraph in MPP mode MUST include the header:

```
X-Codex-Payment: mpp
```

This applies to ALL tools — `curl`, `tempo request`, or any other HTTP client. Without this header the server returns 401 Unauthorized instead of the 402 challenge needed for payment.

## With curl

```bash
curl -i -sS https://graph.codex.io/graphql \
  -H "Content-Type: application/json" \
  -H "X-Codex-Payment: mpp" \
  --data-binary '{"query":"query { getNetworks { id name } }"}'
```

## With tempo request

`tempo request` handles the 402 challenge/payment automatically, but it does NOT add the `X-Codex-Payment: mpp` header for you. You must pass it explicitly:

```bash
tempo request -t -X POST \
  -H "X-Codex-Payment: mpp" \
  -H "Content-Type: application/json" \
  --json '{"query":"query { getNetworks { id name } }"}' \
  https://graph.codex.io/graphql
```

## Common mistake

Sending a request without `X-Codex-Payment: mpp` and expecting MPP to activate. Without the header, the server treats it as a standard unauthenticated request and returns 401 — not a 402 challenge.
