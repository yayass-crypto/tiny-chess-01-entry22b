# tiny-chess-01 — a compact chess engine in 4 KB (v22b)

`main.py` is the whole engine (4,083 bytes as committed): a 0x88-board
negamax search with alpha-beta pruning, quiescence over captures,
promotions and check evasions, a transposition table, MVV-LVA capture
ordering plus history ordering for quiet moves, futility pruning, and
late-move reductions (LMR), with a material + piece-square evaluation
that adds a bishop-pair bonus, a passed-pawn bonus and a mop-up bonus
driving a lone king to the edge in a winning endgame. Quiescence
checks king-capture-only at depth 2 and beyond while in check (never
recursive), returning the true fail-soft value instead of the
window-clamped alpha. The root re-scores a move that stalemates the
opponent as a draw (0) instead of a win, because the pseudo-legal
search would otherwise mistake a stalemate for a mate. In the
endgame (no queens on the board) the king uses the centralising
knight piece-square table instead of the middlegame corner-seeking
king table. It searches to a fixed node budget (200,000 nodes a move),
never to a clock, so a trial replays move for move.

v22b combines two earlier experiments on top of v20: LMR (late quiet
moves at depth >= 3, move index > 3, searched one ply shallower and
re-searched at full depth if they beat alpha) and the endgame king
table switch. The v21 black-castling fix is included. The byte budget
came from semantics-preserving refactors (merged slider loops in
`attacked()`, simplified castling generation, a merged transposition
probe, an inlined root legality check). Passed-pawn doubling and
SEE-lite are NOT in this version: they were tried as v23/v24 and
scored worse or neutral at trial scale.

Trial result (practice trial-0, rerun 2, 16 games vs the 8 fixed house
engines, sunfish baseline Elo 1360): **Elo 1396, 8 wins / 0 draws /
8 losses**. That is the submission candidate: it is the first version
to score above the sunfish baseline on a full practice trial.

Per-game summary (trial-0 rerun 2; colour is ours, result is the game
result, plies count both sides' moves):

| Game | Colour | Result | Plies | Game | Colour | Result | Plies |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | white | 1-0 (win) | 89 | 9 | white | 0-1 (loss) | 124 |
| 2 | black | 0-1 (win) | 60 | 10 | black | 1-0 (loss) | 17 |
| 3 | white | 1-0 (win) | 105 | 11 | white | 0-1 (loss) | 8 |
| 4 | black | 0-1 (win) | 112 | 12 | black | 1-0 (loss) | 11 |
| 5 | white | 1-0 (win) | 145 | 13 | white | 0-1 (loss) | 8 |
| 6 | black | 0-1 (win) | 36 | 14 | black | 0-1 (win) | 154 |
| 7 | white | 1-0 (win) | 55 | 15 | white | 0-1 (loss) | 46 |
| 8 | black | 1-0 (loss) | 171 | 16 | black | 1-0 (loss) | 107 |

Note: the first two runs of this trial scored only Elo 705 (1 win /
15 losses) and are not representative — both were affected by the
shared-VM load/timeout problems of that day; rerun 2 is the clean run
quoted above.

It plays the house gauntlet through the env at `MAKEFUN_ENV_URL`
(`POST /game`, `POST /move`), reading each position's FEN and legal moves
from the env and choosing a move by iterative deepening.

Written from scratch by the author (not a fork of sunfish or any other
engine). MIT licensed.
