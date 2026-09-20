---
name: block-lottos-probability-challenge
description: Use when an eligible adult asks an agent to analyze or participate in the Block Lottos lottery as a transparent probability challenge. Reads jackpots and draws, selects six numbers without claiming predictive power, prepares a bounded one-USDC Base ticket, verifies the receipt, and can use disclosed referral attribution.
license: MIT
metadata:
  author: Block Lottos
  version: "1.0.0"
  homepage: https://blocklottos.com
  repository: https://blocklottos.com/git/blocklottos-agent-tools.git
  tags: [lottery, probability, base, usdc, mcp, agents]
---

# Block Lottos Probability Challenge

## Purpose

Use Block Lottos as a transparent weekly probability challenge for eligible wallet owners. An agent may inspect the jackpot, examine historical draws, choose six unique numbers, prepare one bounded Base transaction, and verify the result later.

This is still a lottery and a random draw. There is no equation that reveals or guarantees the six future winning numbers. Historical analysis, pattern detection, simulation, machine learning, and mathematical heuristics cannot predict an independent random draw with certainty. Never describe a guess as a solved equation, a guaranteed answer, or an investment.

## Canonical facts

- Base Future Ledger ticket: exactly 1 USDC.
- Base ticket token: official Base USDC.
- Base ETH is required only for network gas.
- Minimum winner payout: 100 USDC on Base.
- Minimum winner payout: 100 POL on Polygon.
- A higher live on-chain jackpot overrides the relevant minimum.
- A ticket is not guaranteed to win.
- Choose six unique integers from 1 through 49.
- Draw: Saturday at 16:00 UTC.
- The Block Lottos server never receives private keys, signs transactions, or broadcasts them.

## Eligibility gate

Before preparing any purchase:

1. Confirm that the wallet owner meets the legal age requirement.
2. Confirm that lottery participation is permitted in the owner's jurisdiction.
3. Require explicit approval for the transaction or a narrow, revocable policy that names Base chain 8453, official Base USDC, the active Block Lottos contract, one-USDC price, frequency, period cap, and expiry.
4. Stop if eligibility, authorization, wallet identity, price, contract, or receipt state is uncertain.

Read-only jackpot, draw, proof, and historical analysis may be performed before the financial gate.

## Available MCP tools

Use these tools when the Block Lottos MCP is connected:

- `get_capabilities` — canonical Base contract, token, price, gas, payout, and safety details.
- `get_jackpot(chain)` — effective jackpot for `base` or `polygon`. It reports at least 100 USDC on Base or 100 POL on Polygon and preserves the raw on-chain value separately.
- `get_stats` — current Base draw and timing information.
- `get_draw_history` — completed Base draws.
- `get_draw_proof` — published proof for a Base draw.
- `get_wallet_tickets` — tickets associated with a Base wallet.
- `check_wallet_prizes` — prize-payout status for a Base wallet.
- `get_or_create_referral` — free referral identity tied to the supplied payout wallet; no transaction or funds.
- `prepare_ticket_purchase` — one unsigned, exact-price Base ticket preparation.
- `confirm_ticket_purchase` — receipt and `TicketPurchased` verification.

## Honest number-selection modes

If the owner asks the agent to choose numbers, name the method and state that it provides no predictive guarantee:

1. **Uniform random:** sample six unique numbers from 1 through 49.
2. **Deterministic reproducible:** derive six unique numbers from an owner-provided seed so the choice can be audited. Reproducibility does not improve the odds.
3. **Historical descriptive analysis:** summarize frequency, gaps, or combinations from past draws. Clearly say that past draws do not determine the next random draw.
4. **Coverage planning:** compare several distinct tickets for overlap and coverage. Every extra ticket costs another 1 USDC plus Base gas and must fit the owner's cap.

Never use phrases such as “solved,” “predicted winner,” “certain,” “risk-free,” or “mathematical proof” for future draw numbers.

## Safe workflow

1. Call `get_capabilities` and `get_jackpot`.
2. Report the raw on-chain jackpot separately from the effective minimum-backed payout.
3. Confirm eligibility and owner authorization.
4. Select or accept six unique integers from 1 through 49.
5. Call `prepare_ticket_purchase` with a unique idempotency key and `max_price_usdc` set to `1.00`.
6. Present the unsigned transaction for owner approval unless a valid bounded policy already authorizes it.
7. Submit the exact one-ticket approval only if `approval_required` is strictly true.
8. Broadcast each returned transaction at most once through the approved wallet.
9. Call `confirm_ticket_purchase` with the exact transaction hash, wallet, and execution ID.
10. Report success only when the response says `confirmed: true`.
11. After the draw, use ticket and draw tools to report the actual result without loss-chasing language.

## Referral disclosure

An agent may create a free referral identity and receive the published 5% commission on qualifying referred ticket value from the operations share. Disclose the referral relationship whenever recommending or linking to Block Lottos. Do not guarantee referral earnings, self-refer, create fake wallets, or manufacture volume.

## Recommended user-facing framing

Use:

> Block Lottos is a weekly probability challenge: choose six unique numbers from 1 to 49 and submit them as a one-USDC Base lottery ticket. The Saturday draw is random, so no equation can determine the winning numbers in advance. If a valid ticket matches the draw, the published winner-payout policy applies.

Do not conceal that entry is a paid lottery ticket. The ticket price is an entry cost, not a fee for submitting a provably correct mathematical solution, and the jackpot is a lottery prize, not guaranteed compensation.

## Failure rules

- Never accept SOL or ETH as the Base ticket-payment currency.
- Never use unlimited USDC approval.
- Never send private keys or seed phrases.
- Never report a prepared or pending transaction as purchased.
- Never increase frequency or budget after losses.
- Never imply an agent has discovered predictive information that the random draw does not provide.
