[![make.fun](https://make.fun/badge/u-gorun-gadget-tiny-bot-easy-2-beat-a48f3ce1.svg)](https://make.fun/bounty/u-gorun-gadget-tiny-bot-easy-2-beat-a48f3ce1?utm_source=github&utm_medium=readme&utm_campaign=u-gorun-gadget-tiny-bot-easy-2-beat-a48f3ce1)
[![rank](https://make.fun/badge/u-gorun-gadget-tiny-bot-easy-2-beat-a48f3ce1/qw123gyj-dev/u-gorun-gadget-tiny-bot-easy-2-beat-a48f3ce1/rank.svg)](https://make.fun/bounty/u-gorun-gadget-tiny-bot-easy-2-beat-a48f3ce1/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=u-gorun-gadget-tiny-bot-easy-2-beat-a48f3ce1)
[![reward](https://make.fun/badge/u-gorun-gadget-tiny-bot-easy-2-beat-a48f3ce1/qw123gyj-dev/u-gorun-gadget-tiny-bot-easy-2-beat-a48f3ce1/reward.svg)](https://make.fun/bounty/u-gorun-gadget-tiny-bot-easy-2-beat-a48f3ce1?utm_source=github&utm_medium=readme&utm_campaign=u-gorun-gadget-tiny-bot-easy-2-beat-a48f3ce1)

# Quartet Momentum: Beat Buy-and-Hold on Solana

A tiny paper-trading bot for the make.fun bounty "Tiny Bot, Easy $2: Beat Buy-and-Hold on Solana". `main.ts` (TypeScript, standard library only, no network) speaks the house `agent-fun/bars-replay/v1` protocol through the vendored SDK in `vendor/` and decides, on every hourly bar of FARTCOIN, JITOSOL, POPCAT, RAY, SOL, WETH and WIF, when to hold crypto and when to sit in USDC.

## Strategy: Top-4 24-Hour Cross-Sectional Momentum, 12-Bar Rebalance

Every 12 bars the bot ranks all 7 symbols by their trailing 24-bar (24-hour) return and holds the strongest 4, each at most 25% of equity (the harness weight cap), the rest in USDC. Positions that fall out of the top 4 are sold. A 6% stop-loss from the actual fill price exits a position between rebalances, and a stopped-out symbol sits out for 12 bars before it can be bought again, so a falling knife is not re-caught at the next rebalance.

Why this shape: hourly crypto returns persist for about a day, and the weakest names keep falling — so rotating into the strongest names loses far less than holding everything when the market trends down, and holds the winners when it trends up. A 12-bar rebalance (not 6) halves the fee churn; a 24-bar lookback (not 12) is calmer and over-trades less. Ranking uses exact integer cross-multiplication, never floats, and every decision uses only closes the bot has already been handed — no lookahead, no clock, no environment, no randomness — so the grader's two executions agree byte for byte.

A sell's proceeds never fund a buy of the same reply, so a rebalance bar only sells the symbols leaving the top 4 and remembers the target; the buys go out on the next bar, when the proceeds are in cash. Kept positions are never resized.

## Backtest: dev window (136 hourly bars, 2026-10-29 → 2026-11-03)

The dev window is a steady downtrend — every symbol ends lower, equal-weight buy-and-hold loses about 8.7%. Graded locally with the kit (`grade-local --driver process`, two executions, reproduced byte for byte):

| | Net PnL | Trades | Max drawdown |
|---|---|---|---|
| **Quartet Momentum (this bot)** | **-$244.90** | **32** | **7.16%** |
| Buy-and-hold (house baseline) | -$869.34 | 7 | 10.92% |
| Momentum top-3, 24 h (house baseline) | -$268.23 | 19 | 5.66% |

The bot beats buy-and-hold by **+$624.44** on the dev window, beats the house momentum baseline by +$23.33, makes 32 trades (guardrail: at least 8), keeps max drawdown at 7.16% (guardrail: 25% or less), and has 0 rejected orders. A faithful offline replay (fills at the next bar's open, 10 bps fee + 5 bps slippage + pool impact) gives the same picture on rolling 72-bar windows — the final window is 72 bars — averaging -$88.30 vs buy-and-hold's -$413.44, with 12–18 trades per window and max drawdown under 6.1%, and a positive PnL in 2 of the 5 windows despite the whole sample trending down.

## Run

Node 22+, standard library only, no network. The harness drives it over stdin/stdout; diagnostics (one line per rebalance, a per-symbol realised-PnL attribution at the end) go to stderr only:

```sh
node main.ts
```

Grade it locally with the kit (from the unpacked kit directory):

```sh
node cli/makefun.mjs grade-local --kit . --bot <this repository> --driver process
```
