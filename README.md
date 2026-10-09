# Opuity Frontend

Next.js App Router x402 paywall for **Opuity** — a paid API route that returns an x402 `402 Payment Required` challenge until the caller supplies a valid `PAYMENT-SIGNATURE` header, verifies/settles the payment through the Opuity SDK's `x402Serve()` middleware, and only then serves Opuity signal data.

This repository is one half of the split of the original project:

- **[`opuity`](https://github.com/snooopdog/opuity)** — SDKs (TypeScript/Python), MCP server, CLI, docs and examples.
- **`opuity-frontend`** (this repo) — the Next.js x402 seller app, standalone and runnable on its own.

Both repositories inherit the complete Git history of the original project. Package identifiers on npm are unchanged by the rename: the app depends on the published `nirium` package, exactly as before.

## Setup

```bash
npm install
cp .env.example .env.local
```

Fill in `.env.local`:

```bash
NIRIUM_API_KEY=sk_inst_your_key_here
NIRIUM_BASE_URL=https://nirium-agent.fly.dev
NIRIUM_X402_NETWORK=stellar:testnet
NIRIUM_X402_PRICE_USDC=0.02
NIRIUM_X402_PAY_TO=GBRPYHIL2CI3FNQ4BXLFMNDLFJUNPU2HY3ZMFXYSFZUKW3T2TRWECCHX
NIRIUM_X402_FACILITATOR_API_KEY=oz_test_key
```

- `NIRIUM_X402_PAY_TO` should be the Stellar address (public key starting with a `G...`) that receives USDC payments.
- `NIRIUM_X402_FACILITATOR_API_KEY` (or `FACILITATOR_API_KEY`) / `NIRIUM_X402_FACILITATOR_URL`: OpenZeppelin Channels facilitator key or a custom facilitator URL used to verify and settle payment proofs.

The app fails closed: without a valid `NIRIUM_X402_PAY_TO` and a facilitator configuration, the paid route answers `500` with a misconfiguration error instead of serving data for free.

## Run locally

```bash
npm run dev
```

Request the paid endpoint without payment:

```bash
curl -i http://localhost:3000/api/premium/signals
```

Expected result: HTTP `402 Payment Required` with x402 payment requirements.

After your client signs and submits the x402 payment, retry with the payment signature:

```bash
curl -i http://localhost:3000/api/premium/signals \
  -H 'PAYMENT-SIGNATURE: <signed-payment-proof>'
```

## Scripts

| Command | What it does |
|---|---|
| `npm run dev` | Start the Next.js dev server |
| `npm run build` | Production build |
| `npm start` | Serve the production build |
| `npm run typecheck` | `tsc --noEmit` |
| `npm test` | Node's built-in test runner against `test/smoke.test.js` |

## Notes

- The x402 wrapper lives in [`app/lib/nirium-x402-seller.ts`](./app/lib/nirium-x402-seller.ts); the paid route is [`app/api/premium/signals/route.ts`](./app/api/premium/signals/route.ts).
- Payment proofs are not deduplicated across requests and callers are not rate-limited by this middleware — add your own protection if you need either (see [issue #91](https://github.com/nirium-protocol/nirium/issues/91)).
- Report vulnerabilities privately per [SECURITY.md](https://github.com/snooopdog/opuity/blob/main/SECURITY.md) in the main repository.

## License

Apache-2.0 — see [LICENSE](./LICENSE).
