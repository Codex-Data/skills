# MPP Header Rule

When `$CODEX_API_KEY` is not available and you fall back to MPP (codex-gateway), every request MUST include:

```
X-Codex-Payment: mpp
```

This header is required on both the initial 402 challenge request and the credential retry. Without it the server does not activate MPP and will not return a challenge — the request fails.

Do not send a bare request without auth and expect MPP to work. You must explicitly opt in with this header.
