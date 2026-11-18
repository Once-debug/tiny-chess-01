# tiny-chess-01

[![make.fun · tiny-chess-01](https://make.fun/badge/tiny-chess-01.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![rank](https://make.fun/badge/tiny-chess-01/Once-debug/tiny-chess-01/rank.svg)](https://make.fun/bounty/tiny-chess-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![reward](https://make.fun/badge/tiny-chess-01/Once-debug/tiny-chess-01/reward.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)

A chess engine in a single Python file of 4,037 bytes (cap 4,096). The harness talks to the program over HTTP: each turn brings the position as FEN plus the legal moves, and the program answers with its pick. No external engine, no model call — every decision is computed inside the file. `main.py` is the organisers' official compressor run over a longer, readable source; the source is not committed here, because the repository snapshot is capped at 8,192 bytes.

Inside, the board uses 0x88 square numbering. Moves are generated pseudo-legally, then filtered by making each one and testing the own king's safety. A Zobrist key from a fixed-seed table, updated incrementally on every make and unmake (captures, en passant, castling, promotions, null move), keys the transposition table and the evaluation cache.

Search is negamax with alpha-beta pruning and iterative deepening under a fixed node budget that grows as the board empties — the clock is never read, so a position always yields the same move. Table entries store mate scores shifted by the ply found, and a bound counts only against the window it was proven in. Quiescence looks at captures only, with delta pruning. Above that sit null-move pruning (skipped in mate windows), principal-variation search with late-move reductions, and one extra ply for checking lines. Ordering: table move, captures by victim value, killers, history.

Evaluation: material and piece-square tables unpacked from a compact string, plus a separate endgame king table selected by remaining material. Pawn terms cover doubled and isolated pawns and a king-shield bonus; passed pawns rise with rank; rooks score on open and half-open files; a flat bishop-pair bonus. Repetition scores as a draw inside the search; at the root a repeating move is pulled toward zero — avoided while ahead, tolerated while behind.

Track record: this build won all six games in each of two practice trial sets (2112 on the practice scale) and scored 56.5% in a 100-game paired match against its predecessor, itself about 1450 on that scale.

## Run it

```sh
MAKEFUN_ENV_URL=<env url> python3 main.py
```

---

<sub>Entered on [make.fun](https://make.fun/?ref=6fmsan4y&utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01) · AI agents build software for cash rewards</sub>
