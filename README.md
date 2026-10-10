# VEIL

**Public accountability. Private execution.**

VEIL is a private buyback engine for token communities, powered by NEAR Confidential Intents. Projects fund buybacks from revenue, set a mandate, execute privately, and publish delayed spending reports.

This repo is the web front end: a single static `index.html` with no build step.

## Deploy

1. Set your backend URL at the top of the script in `index.html`:
   ```js
   const VEIL_API = "https://api.your-domain.com";
   ```
2. Push to GitHub and enable **GitHub Pages** (Settings → Pages → Deploy from branch → `main` / root), or deploy the folder to Vercel or Netlify.

Wallet extensions (Phantom, Solflare, MetaMask) only inject on `https://` pages or `localhost`, so test locally with a static server such as `npx serve .`.

## Backend API

The page reads and writes through these endpoints. Every POST body is signed by the connected wallet and includes:

| Field | Meaning |
|---|---|
| `owner` | Wallet address |
| `chain` | `solana` or `evm` |
| `message` | The exact text the wallet signed (action, wallet, timestamp, nonce, payload) |
| `signature` | Base64 ed25519 signature (Solana) or hex `personal_sign` signature (EVM) |

The backend should verify the signature against `owner`, reject stale timestamps, and reject reused nonces.

| Method | Path | Body (besides signed fields) | Returns |
|---|---|---|---|
| GET | `/state?owner=` | — | Treasury state (below) |
| POST | `/mandates` | `token, budget, slippage, windowHours, delayHours, destination, repeat` | `{ id }` |
| POST | `/mandates/:id/close` | `mandateId` | `{ ok }` |
| POST | `/keys` | `who, mandateId, scope, expires` | `{ key }` (full key, shown once) |
| POST | `/keys/:id/revoke` | `keyId` | `{ ok }` |
| GET | `/keys/:key` | — | `{ who, scope, expires, mandates[] }`; 404 unknown, 410 revoked or expired |
| POST | `/stake` | `amount` | `{ ok }` |
| POST | `/unstake` | `amount` | `{ ok }` |

Errors should return JSON `{ "error": "Plain-language message" }`. The page shows it to the user.

### `GET /state` shape

```json
{
  "vault": { "balance": 40.0, "address": "VaultAddress..." },
  "deposits": [{ "source": "Creator fees", "amount": 10, "time": "2026-10-09T18:00:00Z" }],
  "mandates": [{
    "id": "m_1", "token": "Mint...", "symbol": "TOKEN",
    "budget": 25, "spent": 6.1, "acquired": 14300000, "fills": 4,
    "slippage": 1.0, "windowHours": 72, "delayHours": 72,
    "destination": "burn", "repeat": "once",
    "status": "live", "createdAt": "2026-10-09T12:00:00Z",
    "log": [{ "time": "2026-10-09T13:02:00Z", "size": 1.5, "price": 0.00042 }]
  }],
  "pending": [{ "token": "Mint...", "symbol": "TOKEN", "window": "Oct 5 – Oct 8", "spent": 25, "acquired": 59000000, "publishesAt": "2026-10-11T00:00:00Z" }],
  "keys": [{ "id": "k_1", "keyPrefix": "veil_vk_3fa1", "who": "DAO committee", "mandateId": "all", "scope": "totals", "expires": "2026-10-16T00:00:00Z" }],
  "staked": 10000,
  "veilBalance": 240000
}
```

`destination` is `treasury`, `burn`, or `vesting`. `scope` is `totals` or `fills`. `expires` is an ISO date or `"report"`. Only return `log` to the owner and to view keys with `scope: "fills"`.

## Privacy notes

Confidential execution does not hide vault deposits, withdrawals, or market impact. The backend should randomize tranche size and timing, and the reporting layer should publish only aggregated totals after each mandate's delay.

## Links

- X: https://x.com/VeilPrivacyTech
- NEAR Confidential Intents: https://intents.near.org/confidential
