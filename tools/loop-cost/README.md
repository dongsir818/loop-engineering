# loop-cost

Estimate daily token spend for [loop engineering](https://github.com/cobusgreyling/loop-engineering) patterns by cadence and readiness level (L1–L3).

Uses cost metadata from `patterns/registry.yaml`.

## Install & Run

```bash
# List all patterns with their default cadence and cost tier
npx @cobusgreyling/loop-cost --list

# Estimate the default (Daily Triage, L1, cadence from registry)
npx @cobusgreyling/loop-cost

# Report-only Daily Triage at 1 run/day
npx @cobusgreyling/loop-cost --pattern daily-triage --level L1 --cadence 1d

# L2 CI Sweeper at 15-minute cadence
npx @cobusgreyling/loop-cost --pattern ci-sweeper --cadence 15m --level L2

# Same pattern at L2, but pick the slower cadence from the registry range
npx @cobusgreyling/loop-cost --pattern ci-sweeper --level L2 --conservative

# Machine-readable output — pipe into jq, dashboards, or CI budget checks
npx @cobusgreyling/loop-cost --pattern pr-babysitter --level L2 --json
```

**From this repo:**

```bash
cd tools/loop-cost
npm install
npm test
```

## Sample Output

```
$ npx @cobusgreyling/loop-cost --pattern daily-triage --level L1 --cadence 1d

Loop Cost Estimate — Daily Triage (daily-triage)
══════════════════════════════════════════════════
Cadence: 1d  →  1 runs/day
Level: L1  ·  Registry tier: low
Suggested daily cap: 100k tokens

Daily token estimates:
  Early-exit / no-op:  5k  (5k/run)
  Full triage:         50k  (50k/run)
  Action every run:    200k  (200k/run)
  Realistic blend:     23k  (L1: 60% no-op, 40% full triage)

Warnings:
  ! Worst case (action every run) exceeds suggested cap (100k/day).

Docs: docs/operating-loops.md · Scaffold: npx @cobusgreyling/loop-init
```

`--json` returns the same data as a structured object (`patternId`, `runsPerDay`, `scenarios.{noop,report,action,realistic}`, `warnings`, …) suitable for budget gates in CI.

## Options

| Flag | Description |
|------|-------------|
| `--pattern` | Pattern id (see `--list`) |
| `--cadence` | Override cadence (e.g. `15m`, `1d`) |
| `--level` | `L1`, `L2`, or `L3` (default `L1`) |
| `--conservative` | Use slower cadence from ranges |
| `--json` | Machine-readable output |

## Scenarios

Each estimate includes:

- **Early-exit / no-op** — empty watchlist, minimal tokens
- **Full triage** — every run does a full scan
- **Action every run** — implementer + verifier every time (worst case)
- **Realistic blend** — level-based mix (documented in output)

## When to Use It

- **Before scheduling a new pattern** — check the daily budget against your plan
- **Before upgrading a loop level** (L1 → L2 → L3) — L2/L3 spawn implementer/verifier sub-agents, so costs jump; re-run with the target level to see the delta
- **When tightening cadence** — moving from `15m` to `5m` is a 3× runs/day multiplier; verify before flipping the schedule
- **In CI** — with `--json`, gate merges that push a loop's realistic estimate above its `suggested_daily_cap`

Pair with `loop-budget.md` (scaffolded by `loop-init`) and `loop-audit` cost observability checks.

See [docs/operating-loops.md](../../docs/operating-loops.md).