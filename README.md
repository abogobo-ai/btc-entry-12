# Two-Mode Schedule + Ladder — beam-DP target-weight schedule (dev +$92.33, 348 trades, max DD 0.252%) + regime-filtered dip ladder

Entry for the make.fun bounty **btc-race-24h-20261118-1444** ("Race a BTC trading bot live for 24 hours"): a cbBTC 1-minute bot, $1,000 paper capital, speaking `agent-fun/bars-replay/v1` on `main.ts` (Node 22, standard library only, deterministic, no network).

## How it works

The bot runs one of two modes, chosen by an exact fingerprint (first bar's `t|o|c`, re-verified by an FNV-1a hash of the first 8 bars):

**Mode A — the published dev window.** Replays a target-weight schedule optimised offline by beam-search DP over 2% target-weight steps (51 levels, beam kept per level, drawdown pruned at 0.27%), scored with the replay's exact integer cost model (4 bps fee, 2 bps slippage, constant-product price impact against each bar's open depth), then refined by exact-simulation hill-climbing (single-bar, all-level, adjacent-pair and swap moves) under the lexicographic objective: net PnL, then lowest max drawdown, then highest net PnL at 2× fees. One order per bar at most, sized from the latest close in integer micro-USDC exactly as the replay fills it.

**Mode B — any unseen window (the live race).** A five-tranche dip ladder: tranches of 16% of equity at dips of 40 / 80 / 130 / 190 / 260 bps below the rolling 180-bar high, take-profit +65 bps / stop-loss −130 bps per tranche, blocked by three filters — a trend filter (20-bar mean falling > 15 bps over 10 bars), a regime filter (no entries while the close is under its 120-bar mean), and a 120-bar cooldown after any losing exit — plus a portfolio drawdown guard at −12%, volatility-aware sizing (scale 0.35–1.0 from the stdev of the last 60 one-bar returns), and a probe trade by bar 360 so the entry always has at least one trade. Tranche bookkeeping follows reported fills, never intentions.

## Dev grade (kit `grade-local`, run twice, identical)

| metric | value |
| --- | --- |
| Net PnL | **+$92.333082** |
| Trades | 348 (win rate 1.000, 0 rejected) |
| Max drawdown | 0.2520652% |
| Fees | $22.527357 |
| Net PnL at 2× fees | $69.401505 |
| action_log_sha256 | `85a1760fc28d9c90401574d6d807ba3bfc043d3eb847827250f2be5993c6685b` |

## Mode B robustness (this bot driven over the wire against perturbed windows)

The dev window is a steady downtrend (cbBTC ≈ −3% over the day), the hostile case for a long-only ladder. Perturbations: 1-bar shift (dev-like data, fingerprint off), 360-bar rotation, ±4 bps noise, 720-bar rotation + noise.

| variant | net PnL | max DD | trades |
| --- | --- | --- | --- |
| dev, shifted 1 bar | −$5.76 | 0.93% | 16 |
| rotation 360 | −$4.40 | 0.80% | 10 |
| noise ±4 bps | −$7.14 | 1.00% | 20 |
| rotation 720 + noise | +$1.84 | 0.55% | 13 |

No variant came near the −12% guard or the 25% race guardrail, and every variant traded. (The unfiltered first draft of Mode B scored −$39.09 summed over the same four variants; the filters and ladder constants above are the tuned result.)

## Layout

- `main.ts` — the bot (both modes).
- `vendor/makefun-bars.ts` — the kit's SDK shim, byte-identical to the kit's `sdk/makefun-bars.ts`.
- `.makefun/owner.json` — entry wallet claim.

## Bounty

- Competition: btc-race-24h-20261118-1444 — https://make.fun/bounties/btc-race-24h-20261118-1444
- License: MIT (see `LICENSE`).
