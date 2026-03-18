# MPP Header Rule

When `$CODEX_API_KEY` is not available and you fall back to MPP (codex-gateway), every request MUST include:

```
-H "X-Codex-Payment: mpp"
```

This applies to ALL HTTP tools — `curl`, `tempo request`, or any other client. `tempo request` handles the 402 payment automatically but does NOT add this header for you.

Without this header the server returns 401 Unauthorized, not a 402 challenge.
