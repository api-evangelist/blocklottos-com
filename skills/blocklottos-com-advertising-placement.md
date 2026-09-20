---
name: blocklottos-advertising-placement
description: Place a banner ad on blocklottos.com programmatically - list sizes and current USDC prices, request a submission quote, pay the 1 USDC submit fee from your own wallet and confirm it, poll review status, then request an activation quote for 7/14/30 days, pay it and confirm to go live.
api: openapi/blocklottos-com-openapi.yml
operations: [getAdSizes, submitAd, confirmAdSubmitPayment, getAdStatus, getAdsByWallet, requestAdActivationQuote, confirmAdActivationPayment]
generated: '2026-09-19'
method: generated
grounding: Every operationId above exists verbatim in openapi/blocklottos-com-openapi.yml; prices from GET /api/ads/sizes (probed 2026-09-19) and rules from https://blocklottos.com/api-docs#ads-overview and Terms §10.
---

# Place a banner ad via the Advertising API

Quote-first, non-custodial: the API never pulls funds. Each paid step returns an **unsigned USDC transfer**
you sign and broadcast from your own wallet, then you confirm with the `tx_hash`. Quotes expire after **1 hour**.
No API key; rate limits are per IP. Base URL `https://blocklottos.com/api/ads/`.

## Before you start

- Advertising fees are **non-refundable**: the submit fee regardless of review outcome, the activation fee once the ad is approved and activated (Terms §10). There is no cancel or refund operation.
- Content policy (Terms §10): no nudity, violence, drugs, deceptive claims, phishing/malware links; destination URLs must be live, legitimate sites. Rejected ads forfeit the submit fee.
- Image must exactly match the chosen size, JPEG/PNG/GIF/WEBP, max 2 MB.

## Steps

1. `getAdSizes` - `GET /api/ads/sizes` (2 req/min/IP). Sizes: 728x90, 300x250, 320x50, 160x600. Observed 2026-09-19: every size priced 85 USDC (7d) / 140 USDC (14d) / 250 USDC (30d), `submit_fee` 1 USDC, `discount_pct` 0. Always read the live price; discounts are applied server-side.
2. `submitAd` - `POST /api/ads/submit` (10 req/min/IP), multipart form: `image`, `size_id`, `target_url`, `wallet_address`, optional `network` (default `polygon`; also base, arbitrum, optimism, ethereum, avalanche, bnb). Returns `quote_id` (`sq_...`), `status: awaiting_payment`, `expires_at` and `payment.tx` (an ERC-20 `transfer` to the fee address for the exact amount).
3. Sign and broadcast `payment.tx` from `wallet_address` on the quoted network. Submit it once.
4. `confirmAdSubmitPayment` - `POST /api/ads/submit/pay` with `quote_id` and `tx_hash` (form or JSON). `402` means the payment could not be verified on chain yet - wait for confirmations and retry the *confirm*, never re-send the payment. Success returns a `submission_id` (`ad_42`) in `pending_review`.
5. `getAdStatus` - `GET /api/ads/status/{submission_id}` (2 req/min/IP). Statuses: `pending_review` -> `approved` | `rejected`; later `active` -> `expired`. Manual review "usually takes 24-48 hours". `getAdsByWallet` - `GET /api/ads/wallet/{wallet_address}` recovers submission ids you did not store.
6. When `approved`: `requestAdActivationQuote` - `POST /api/ads/activate` with `submission_id`, `duration` (`7d`|`14d`|`30d`), `wallet_address`, optional `network`. `409` = the ad is not in an activatable state. Returns `quote_id` (`aq_...`) and the payment tx at the current (discount-aware) price.
7. Sign and broadcast, then `confirmAdActivationPayment` - `POST /api/ads/activate/pay` with `quote_id` + `tx_hash`. The ad goes `active` and `expires_at` is set; an `expired` ad can be re-activated by repeating steps 6-7.

## Errors and limits

- Envelope on this API: `{"error":"..."}`. Codes: 400 bad params, 402 payment not verified, 404 unknown submission, 405 wrong method, 409 wrong state, 429 rate limited (`Retry-After` header + `retry_after` seconds), 500.
- Poll status no faster than 2 req/min; a quote you let expire must be re-requested (a new quote may carry a new price).
