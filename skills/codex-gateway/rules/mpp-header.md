# MPP Header Rule

Every request to the Codex Supergraph in MPP mode MUST include the header:

```
X-Codex-Payment: mpp
```

This applies to both the initial challenge request and the credential retry. Without this header the server does not activate MPP — it will not return a 402 challenge and the request will fail silently or with an auth error.

## Correct initial request

```bash
curl -i -sS https://graph.codex.io/graphql \
  -H "Content-Type: application/json" \
  -H "X-Codex-Payment: mpp" \
  --data-binary '{"query":"query { getNetworks { id name } }"}'
```

## Correct retry with credential

```bash
curl -i -sS https://graph.codex.io/graphql \
  -H "Content-Type: application/json" \
  -H "X-Codex-Payment: mpp" \
  -H "Authorization: Payment <base64url-credential>" \
  --data-binary '{"query":"query { getNetworks { id name } }"}'
```

## Common mistake

Sending a request without `X-Codex-Payment: mpp` and expecting a 402 challenge. The server will not return a challenge without this header — it treats the request as a standard unauthenticated request and rejects it.
