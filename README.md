[![make.fun · u-gorun-gadget-tiny-bot-easy-2-beat-a48f3ce1](https://make.fun/badge/u-gorun-gadget-tiny-bot-easy-2-beat-a48f3ce1.svg)](https://make.fun/bounty/u-gorun-gadget-tiny-bot-easy-2-beat-a48f3ce1?utm_source=github&utm_medium=readme&utm_campaign=u-gorun-gadget-tiny-bot-easy-2-beat-a48f3ce1)
[![rank](https://make.fun/badge/u-gorun-gadget-tiny-bot-easy-2-beat-a48f3ce1/Alim-14-code/u-gorun-gadget-tiny-bot-easy-2-beat-a48f3ce1/rank.svg)](https://make.fun/bounty/u-gorun-gadget-tiny-bot-easy-2-beat-a48f3ce1/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=u-gorun-gadget-tiny-bot-easy-2-beat-a48f3ce1)
[![reward](https://make.fun/badge/u-gorun-gadget-tiny-bot-easy-2-beat-a48f3ce1/Alim-14-code/u-gorun-gadget-tiny-bot-easy-2-beat-a48f3ce1/reward.svg)](https://make.fun/bounty/u-gorun-gadget-tiny-bot-easy-2-beat-a48f3ce1?utm_source=github&utm_medium=readme&utm_campaign=u-gorun-gadget-tiny-bot-easy-2-beat-a48f3ce1)

# Breakout Rotation Bot

A tiny paper-trading bot for the make.fun competition
**"Tiny Bot, Easy $2: Beat Buy-and-Hold on Solana"**. It speaks the
`agent-fun/bars-replay/v1` line protocol on stdin/stdout (Node 22, stdlib
only) and trades hourly bars of FARTCOIN, JITOSOL, POPCAT, RAY, SOL, WETH
and WIF.

## Strategy

Selective momentum rotation, long only, no leverage:

- **Sit in cash most of the time.** A symbol is bought only when its 6-hour
  move is above +1.3% **and** its 18-hour trend is non-negative — buying the
  start of a move, not the top of it. No new-high requirement.
- **Rank by risk-adjusted momentum** (6h move / 12h volatility) and hold at
  most 3 symbols, sized by inverse volatility, at most 75% of equity
  deployed and 25% per symbol (the harness weight cap).
- **Exit fast.** A position is sold when its 6-hour momentum fades below
  +0.3%, it falls 5% off its 12-hour high, it banks 5% over entry cost, or
  it loses 4% against entry cost. Exits are checked every bar; entries are
  re-ranked every bar.
- **Short memory on purpose.** Only 19 bars of history are needed, so the bot
  can trade from early in even a 3-day final window.

## Run

```sh
node main.ts
```

The bot reads `hello` / `batch` / `done` JSON lines on stdin and writes one
`reply` line per batch on stdout. `vendor/makefun-bars.ts` is the kit's
protocol shim, vendored byte-identically. Everything is a pure function of
the bars seen so far — no clock, no randomness, no network — so the two
grading runs agree byte for byte.

## Dev-window result (official local grader, `--driver process`)

- **Net PnL −103.71 USDC** on the 136-bar dev window, 25 trades, max
  drawdown 2.38%, guardrails met, deterministic regrade.
- Buy-and-hold on the same window: **−869.34 USDC** — this bot beats it by
  **+765.63 USDC**. It also beats the best house baseline that meets the
  guardrails (ema-cross, −173.04 USDC).
