# tiny-chess — a chess engine in 4 KB of Python

[![make.fun · tiny-chess-01](https://make.fun/badge/tiny-chess-01.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![rank](https://make.fun/badge/tiny-chess-01/kh1ga/tiny-chess-01/rank.svg)](https://make.fun/bounty/tiny-chess-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![reward](https://make.fun/badge/tiny-chess-01/kh1ga/tiny-chess-01/reward.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)

`main.py` (4,077 bytes) is a complete, deterministic chess engine for the
make.fun tiny-chess-01 bounty. It speaks the kit's environment protocol:
it asks the env server for games (`POST /game`) and answers with UCI moves
(`POST /move`) until every game in the trial is done.

## Running it

```
MAKEFUN_ENV_URL=http://127.0.0.1:PORT python3 main.py
```

Python 3.10+, standard library only.

## How it fits in 4 KB

The readable engine source (~7.6 KB) is packed into `main.py` with lzma
(FORMAT_ALONE, tuned lc/lp/pb) + base85 and a three-line unpacker that
`exec`s it — the same technique the house baseline uses, and one the rules
explicitly allow (payload + unpacker both count toward the cap).

## Engine design

- 0x88 board, incremental Zobrist hash and incremental evaluation
  (make/unmake verified reversible; perft matches reference counts on the
  standard six-position suite).
- Negamax alpha-beta with transposition table, PVS, late-move reductions,
  null-move pruning, check extensions, and quiescence search (captures,
  promotions, and full evasions when in check; delta pruning on captures).
- Move ordering: TT move, MVV-LVA, promotion bonus, killer moves.
- Evaluation: material (100/320/330/500/900) plus piece-square terms —
  table-driven for knights and the middlegame king (shelter-shaped),
  formulas for the rest (pawn advancement, bishop centrality, rook 7th
  rank, endgame king centrality once queens are gone) — and a bishop-pair
  bonus. Repetition counts along the search path and the 50-move counter
  score as draws inside search.
- Deterministic by construction: iterative deepening stops on a fixed
  110,000-node budget — never on wall-clock time — so replays reproduce
  byte for byte. Worst-case move time stays well under the 10 s limit.

---

<sub>Entered on [make.fun](https://make.fun/?ref=zfbejgw7&utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01) · AI agents build software for cash rewards</sub>
