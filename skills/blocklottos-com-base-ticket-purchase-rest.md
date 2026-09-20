---
name: blocklottos-base-ticket-purchase-rest
description: Buy and verify exactly one Base Future Ledger lottery ticket (1 USDC) through the Block Lottos REST API without the MCP server - read live capabilities, optionally enroll a free referral, prepare one unsigned ticket with a price cap and idempotency key, sign/broadcast in the owner's wallet, confirm on chain, then monitor tickets and results.
api: openapi/blocklottos-com-openapi.yml
operations: [getBaseAgentCapabilities, getOrCreateUnifiedAffiliateProfile, prepareBaseAgentPurchase, confirmBaseTicketPurchase, getLotteryTickets, getLotteryDrawHistory, checkLotteryPrizes]
generated: '2026-09-19'
method: generated
grounding: Every operationId above exists verbatim in openapi/blocklottos-com-openapi.yml; rules are taken from the provider's llms.txt, agents.txt, api-docs and Terms of Use.
---

# Buy one Base ticket through the REST API

This is the REST twin of the provider-authored `blocklottos-com-block-lottos-probability-challenge.md`
skill, which is written against MCP tool names. Use this one when you call HTTPS directly.
No API key is needed. Base URL: `https://blocklottos.com`. All bodies and responses are JSON.

## Gate before you spend anything

1. Confirm the wallet owner is of legal age and lottery participation is permitted in their jurisdiction (Terms §2).
2. Confirm explicit approval for this purchase, or a narrow revocable policy naming chain 8453, official Base USDC, the active contract, 1 USDC per ticket, a period cap and an expiry.
3. Remember: **all ticket purchases are final** once the transaction confirms (Terms §4). There is no cancel, refund or reversal. The only checkpoint is *before* you broadcast.

## Steps

1. `getBaseAgentCapabilities` - `GET /api/lottery/agent-capabilities` (10 req/min/IP). Treat the response as the source of truth for price, contract, payment token, next draw and safety limits. Re-fetch before **every** purchase.
2. Optional: `getOrCreateUnifiedAffiliateProfile` - `POST /api/lottery/agent-referral` (5 req/min/IP). Send `{"action":"challenge","primary_chain":"evm","connected_wallet":"0x..."}`, sign the exact returned message in the identity wallet, resubmit with `challenge_id` + `signature`. Costs 0 USDC, moves no funds. Store the one-time `management_token` and send it only as `Authorization: Bearer blm_...` for later payout-wallet changes or private balances. Use the returned `referral_id` in step 3.
3. `prepareBaseAgentPurchase` - `POST /api/lottery/agent-purchase` (10 req/min/IP) with `wallet_address`, six unique `numbers` (1-49), `max_price_usdc` (`"1.00"`), a unique `idempotency_key` (8-128 chars, `[A-Za-z0-9._:-]`), optional `referral_id`, optional `valid_until` (ISO-8601 with timezone, <= 7 days ahead), optional `agent_id`. The response carries `approval_required`, an exact one-ticket USDC `approval_transaction`, the ticket `transaction` and an `execution_id`. `409` means the live price exceeded your cap or the intent expired - stop, do not retry blindly.
4. Sign and broadcast in the owner's wallet (`eth_sendTransaction`): the approval **only if `approval_required` is true**, then the ticket transaction. Never grant unlimited USDC approval. Re-check `valid_until` immediately before broadcasting - the contract call has no deadline parameter. Submit each returned transaction **at most once**: the idempotency key correlates preparation with confirmation, it does **not** stop a duplicate broadcast.
5. `confirmBaseTicketPurchase` - `POST /api/lottery/confirm-ticket-tx` with `tx_hash`, `wallet_address`, `execution_id`. `202` = still pending, poll again. `409` = failed or mismatched transaction. Report success **only** when the response says `confirmed: true` (it checks status, sender, active contract, method and the `TicketPurchased` event).
6. Monitor: `getLotteryTickets` - `GET /api/lottery/tickets/{wallet}?chain=base`; `getLotteryDrawHistory` - `GET /api/lottery/draw-history?chain=base&limit=1`; after a draw, `checkLotteryPrizes` - `GET /api/lottery/check-prizes/{wallet}?chain=base` (payouts are automatic; this reports only a failed transfer that can be retried).

## Errors and limits

- Envelope: `{"status":"error","message":"..."}`; `429` carries a `Retry-After` header and a `retry_after` body field - back off for that many seconds.
- Read endpoints are cached 60 s and limited to 2 req/min/IP; do not poll faster than the cache.
- `503` = upstream RPC unavailable; wait and retry the *read*, never re-broadcast a transaction on a 503 from the confirm step.

## Truthfulness

Call this a lottery / weekly probability challenge. Never claim a ticket will win, never chase losses, never exceed the owner's limits, never state a purchase succeeded before step 5 confirms it. A verified winning Base ticket receives at least 100 USDC (higher live jackpot applies); that is a payout policy, not a prediction.
