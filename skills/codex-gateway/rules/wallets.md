# Wallet Reference

MPP challenges require a funded wallet to sign payment credentials. Which tool to use depends on the network the challenge specifies.

| Network | Tool | Auth method | Currency |
| ------- | ---- | ----------- | -------- |
| Tempo (chain 4217) | `tempo` (`wallet` + `request` extensions) | Passkey via `tempo wallet login` | USDC |
| Base | `awal` (Coinbase Agentic Wallet) | Email OTP via `npx awal auth login` | USDC |

Pick the tool that matches the network in the `WWW-Authenticate` challenge, then follow the corresponding section below.

---

## Tempo — tempo wallet / tempo request

`tempo` is the Tempo CLI. The `wallet` extension manages identity and funding; the `request` extension is an HTTP client that handles the full MPP 402-challenge flow automatically — it detects the challenge, signs a Tempo transaction, and retries the request in one step.

### Install

```bash
curl -fsSL https://tempo.xyz/install | bash
tempo add wallet
```

### Setup

```bash
tempo wallet login
```

This opens a passkey-based auth flow. No private keys to manage.

### Preflight check

```bash
tempo wallet -t whoami
```

Confirm `ready` is `true` and `balance.available` has sufficient USDC before making paid requests.

### Making a paid request

You MUST include the `X-Codex-Payment: mpp` header — `tempo request` does not add it for you:

```bash
tempo request -t -X POST \
  -H "X-Codex-Payment: mpp" \
  -H "Content-Type: application/json" \
  --json '{"query":"query { getNetworks { id name } }"}' \
  https://graph.codex.io/graphql
```

`tempo request` handles the 402 challenge and payment automatically, but only if the server returns a 402. Without the MPP header, the server returns 401 instead.

### Session management

Sessions open a payment channel on-chain once, then use off-chain vouchers for subsequent requests (no gas per request):

```bash
tempo wallet -t sessions list          # view active sessions
tempo wallet -t sessions close --all   # close all sessions when done
```

### Recovery

| Symptom | Action |
| ------- | ------ |
| `No wallet configured` | Run `tempo wallet login` |
| `Insufficient balance` | Tell the user — wallet needs funding |
| `Spending limit exceeded` | Tell the user — key limit reached |

---

## Base — awal

awal is Coinbase's Agentic Wallet CLI for Base. It holds USDC, sends payments, trades tokens, and handles x402 paid API requests — all without the agent touching private keys.

### Setup

Authentication uses email OTP:

```bash
npx awal auth login user@example.com
# sends a 6-digit code to the email

npx awal auth verify <flowId> <otp>
# completes sign-in
```

### Preflight check

```bash
npx awal status --json
```

Confirm the wallet is authenticated and has funds:

```bash
npx awal balance --json
```

### Making a paid request (x402)

```bash
npx awal x402 pay <url>
```

For discovering available paid services:

```bash
npx awal x402 bazaar search <query>
```

### Other operations

| Command | Purpose |
| ------- | ------- |
| `npx awal address` | Get wallet address |
| `npx awal send <amount> <to> [--chain]` | Send USDC (supports ENS) |
| `npx awal trade <amount> <from> <to>` | Swap tokens on Base |
| `npx awal balance [--chain]` | Check USDC balance |

All commands accept `--json` for machine-readable output.

### Recovery

| Symptom | Action |
| ------- | ------ |
| Not authenticated | Run `npx awal auth login <email>` then verify OTP |
| Insufficient balance | Tell the user — wallet needs funding on Base |
| Server not running | Run `npx awal status` to check health |

---

## Rules

- **Never print, log, or read private keys.** Both tools handle key management internally.
- **Always run a preflight check** before attempting paid requests.
- **If auth fails, fix it automatically** — run `tempo wallet login` or `npx awal auth login` as appropriate, then retry.
- **Do not mix tools.** Use tempo for Tempo challenges, awal for Base challenges.
