# makefun-tiny-chess-01

A chess engine in 4KB of Python for the make.fun tiny-chess-01 competition.
`main.py` (3,796 bytes) is the LZMA-packed entrypoint produced by the kit's
`makefun compress` from `engine.py` (the readable source, kept uncommitted in
the working tree so the bundle can send it beside the sessions).

## Design

- 0x88 board, incremental evaluation, Zobrist hashing (seeded, deterministic).
- Full legal move generation: castling, en passant, promotions, check /
  checkmate / stalemate, repetition and the 50-move rule.
- Negamax alpha-beta with quiescence (captures, promotions, all evasions
  when in check), transposition table (2^18 slots), killer moves, history
  heuristic, MVV-LVA ordering, LMR and null-move pruning.
- Iterative deepening to a **fixed node budget of 70,000 nodes per move**
  (no wall clock in the search, so replays are deterministic).
- Eval mirrors the house engine: material P100 N320 B330 R500 Q900,
  piece-square tables, king middlegame/endgame interpolation, bishop pair +30.

## Local results (practice trial-0, 6 games)

- v6 (60k nodes): 4.0/6 — W/W/D/W/D/D, Elo ≈ 1450, slowest move 3,358 ms.
- v7 (80k nodes): 4.5/6 — W/W/W/W/D/L, Elo ≈ 1550, slowest move 8,086 ms.
- Final (70k nodes, this entry, packed `main.py`): 3.5/6 —
  W/W/W/D/L/L, Elo ≈ 1365 on this trial alone (pooled with the v6/v7
  trials ≈ 1450), slowest move 2,929 ms, average 1,696 ms.
- Perft verified (startpos, Kiwipete and 4 more positions) and move
  generation cross-checked against python-chess over 300 random games.

## How to test

```sh
# move generation (from the repo root, needs python-chess for the diff test)
MAKEFUN_ENV_URL=http://127.0.0.1:9 python3 <workdir>/perft5.py
# play a practice trial against the local env server (kit)
MAKEFUN_ENV_DATA_DIR=<kit>/practice/trial-0/input/env \
MAKEFUN_STATE_DIR=/tmp/tc-state MAKEFUN_ENV_PORT=18080 \
node <kit>/program/env/server.mjs &
MAKEFUN_ENV_URL=http://127.0.0.1:18080 python3 main.py
```

## Files

- `main.py` — the entry (packed, 3,796 bytes).
- `engine.py` — the readable source (uncommitted; `makefun compress`
  records the pair in `~/.makefun/compress.json`).
- `LICENSE` — MIT (from the kit starter).
- `.makefun/owner.json` — the entry's wallet claim.
