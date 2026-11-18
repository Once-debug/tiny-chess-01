# Tiny Chess — contest entry

A chess engine in a single Python file of at most 4,096 bytes, entered in the tiny-chess-01 bounty on make.fun.

## The entry file

`main.py` is the program that plays; it currently takes 4,037 bytes. I wrote a longer, readable implementation first and then ran the organisers' official compression tool over it. The tool records the pairing, so the shipped file traces back to its source. The readable source is not committed here: the contest caps the whole repository snapshot at 8,192 bytes, and the source alone would break that budget.

## How a game runs

The harness talks to the program over HTTP. For every move it sends the position as FEN plus the legal moves, and the program answers with its pick. Nothing is delegated — no external engine, no model call; every decision is computed inside the file.

## Inside the engine

The board uses 0x88 square numbering, so off-board tests are a single AND. Moves are generated pseudo-legally, then filtered by making each one and checking the own king is not left under attack. A Zobrist key from a fixed-seed table, updated incrementally on every make and unmake — captures, en-passant, castling, promotions and the null move included — identifies positions for the transposition table and the evaluation cache.

Search is negamax with alpha-beta pruning and iterative deepening under a fixed node budget that grows a little as the board empties. The clock is never consulted, so the same position always yields the same move. Table entries store mate scores shifted by the ply they were found at, and a stored bound is only trusted against the window it was proven in. Quiescence search looks at captures only, with delta pruning against the stand-pat score. Above that sit null-move pruning (skipped inside mate windows), principal-variation search with late-move reductions, and one extra ply for lines that give check. Ordering tries the table move first, then captures by victim value, then killers and a history table.

Evaluation starts from material and piece-square tables unpacked from a compact character string, with a separate king table for the endgame selected by remaining material. Pawn terms cover doubled and isolated pawns, a bonus for pawns shielding their king, and passed pawns rising with rank; rooks score on open and half-open files, and holding both bishops earns a flat bonus.

Repetition is handled twice: a position already seen in the current game scores as a draw inside the search, and at the root a move that repeats an earlier position has its score pulled toward zero — so the engine avoids repetition while ahead and tolerates it while behind.

## Track record

This build replaces an earlier one of mine that played at about 1450 on the practice scale. It won all six games in each of two official practice trial sets (2112 on that scale) and scored 56.5% in a 100-game paired match against its predecessor.
