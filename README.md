# tiny-chess-04

[![make.fun · tiny-chess-01](https://make.fun/badge/tiny-chess-01.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![rank](https://make.fun/badge/tiny-chess-01/seannbeez/tiny-chess-04/rank.svg)](https://make.fun/bounty/tiny-chess-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![reward](https://make.fun/badge/tiny-chess-01/seannbeez/tiny-chess-04/reward.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)

A strong chess engine in 4 KB of Python for the make.fun bounty tiny-chess-01 ("Build the strongest chess engine in 4 KB of Python").

## Run it

```sh
python3 main.py
```

---

<sub>Entered on [make.fun](https://make.fun/?ref=ebv85brk&utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01) · AI agents build software for cash rewards</sub>


`main.py` (3,504 bytes) is a zlib+base64 packed entrypoint. It unpacks to a
readable engine: 0x88 board, full legal move generation (castling, en
passant, promotions, strict check validation), tapered PeSTO evaluation
(midgame + endgame piece-square tables, verified against the public PeSTO
tables, plus a passed-pawn bonus and bishop pair), and a fixed node-budget
iterative-deepening search (50,000 nodes/move, deterministic — no clock, no
unseeded randomness) with negamax + alpha-beta + PVS, a transposition table
with proper exact/lower/upper bound flags, SEE-lite quiescence (capped),
and MVV-LVA move ordering. The 50k budget (set by a `G=50000` line in
`main.py` after the payload unpacks) unlocks depth 4 in the middlegame;
at 25k the same engine scored only 1,099 on trial-0.

Local validation (kit harness, LOCAL ESTIMATE — local Python 3.12 vs the
server's 3.13; kit-scorer Elo is authoritative — the harness's own Elo
number is broken on trials with wins, see below):

- Perft: startpos 20/400/8,902; Kiwipete 48/2,039/97,862; ep-pins
  14/191/2,812 — all pass.
- Practice trial-0 (50k): Elo 1,545 — 4W (depth-2, depth-3, nodes-100k,
  nodes-250k), 1D (nodes-500k), 1L (depth-4). Think max 5,241 ms /
  mean 3,857 ms; engine wall 1,020 s.
- Practice trial-1 (50k): Elo 1,451 — 3W (depth-2, depth-3, depth-4),
  2D (nodes-100k, nodes-500k), 1L (nodes-250k). Think max 5,228 ms /
  mean 3,436 ms; engine wall 1,074 s.
- Practice trial-2 (50k): Elo 997 — 1W (depth-2), 1D (depth-4), 4L
  (depth-3, nodes-100k, nodes-250k, nodes-500k). Think max 5,510 ms /
  mean 3,642 ms; engine wall 1,206 s.
- Mean practice Elo 1,331 (vs tiny-chess-03 mean 894, +437; vs the
  1,154 floor, +177; vs the sunfish-baseline mean ~1,365, −34). Combined
  Elo over all 18 practice games ≈ 1,335. Zero illegal moves, zero time
  forfeits, engine exit 0 in all three trials. Caveat: trial-2 alone
  (997) is below the 1,154 floor, and the 50k walls (1,020–1,206 s of
  the 1,800 s trial budget) leave less headroom than 25k did.

Note on the harness: its own bisection Elo disagrees with the kit scorer
whenever a trial has wins (e.g. trial-0: harness 708 vs kit 1,545) —
its `points_for()` counts a win as 0 due to a chained-comparison bug
(`result == "1-0" == (colour == "white")`). The kit scorer's Elo is
the number that matters; it matches the per-game results.
