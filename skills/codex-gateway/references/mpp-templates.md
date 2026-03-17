# MPP Query Templates

## 1) MPP opt-in query (challenge request)

```bash
curl -i -sS https://graph.codex.io/graphql \
  -H 'Content-Type: application/json' \
  -H 'X-Codex-Payment: mpp' \
  --data-binary @- <<'JSON'
{
  "query": "query GetNetworks { getNetworks { id name } }"
}
JSON
```

Expected:
- `402 Payment Required`
- Multiple `WWW-Authenticate: Payment ...` headers

## 2) MPP query retry with credential

```bash
curl -i -sS https://graph.codex.io/graphql \
  -H 'Content-Type: application/json' \
  -H 'X-Codex-Payment: mpp' \
  -H "Authorization: Payment $MPP_CREDENTIAL" \
  --data-binary @- <<'JSON'
{
  "query": "query GetNetworks { getNetworks { id name } }"
}
JSON
```

Expected:
- `200 OK`
- GraphQL data
- `Payment-Receipt` header
