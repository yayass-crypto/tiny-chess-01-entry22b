# tiny-chess-01 — compact chess engine in 4 KB (v22b)

`main.py` (4,083 bytes) is the whole engine: 0x88-board negamax with
alpha-beta, quiescence over captures/promotions/check evasions,
transposition table, MVV-LVA + history ordering, futility pruning and
late-move reductions (LMR). Evaluation is material + piece-square
tables with bishop-pair, passed-pawn and mop-up bonuses; in the
endgame (no queens) the king uses the centralising knight table.
Quiescence checks king-capture-only at depth 2+ while in check (never
recursive) and returns the true fail-soft value. The root re-scores a
stalemating move as a draw (0). Fixed node budget (200,000/move),
never a clock, so trials replay move for move.

v22b = v20 + LMR (late quiet moves at depth >= 3, index > 3, one ply
shallower, re-searched at full depth if they beat alpha) + the endgame
king-table switch, including the v21 black-castling fix. Byte budget
from semantics-preserving refactors. Passed-pawn doubling and SEE-lite
are NOT here (tried as v23/v24: worse/neutral at trial scale).

Trial (practice trial-0 rerun 2, 16 games vs the 8 fixed house
engines, sunfish baseline Elo 1360): **Elo 1396, 8 wins / 0 draws /
8 losses** — the first version above the baseline. Wins: games 1-7
and 14; losses: 8-13, 15, 16. Earlier versions on the same trial:
v19/v20 Elo 1253, v21 Elo 1131. (First two runs scored Elo 705, 1W/15L,
both hit by shared-VM load/timeouts that day; rerun 2 is the clean run.)

Plays the house gauntlet via `MAKEFUN_ENV_URL` (`POST /game`,
`POST /move`), reading FEN + legal moves and choosing by iterative
deepening. Written from scratch (not a fork of sunfish). MIT licensed.
