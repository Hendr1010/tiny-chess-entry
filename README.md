# tiny-chess

A compact chess engine in a single 4,079-byte Python file (stdlib only).
The engine source is packed as an LZMA payload (base85) with a tiny
unpacker, so a full-featured search fits the 4,096-byte cap.

- 0x88 board, iterative-deepening alpha-beta with a fixed node budget
  (40,000 nodes/move), transposition table (Zobrist), killer moves,
  MVV-LVA capture ordering, quiescence over captures/promotions/checks
  with delta pruning, repetition detection.
- Evaluation: material + piece-square tables + bishop pair + king
  safety/centralisation by game phase.
- Deterministic: no wall clock in the search, no randomness; the node
  budget (not time) bounds every move.
- Talks to the make.fun env at `MAKEFUN_ENV_URL` (POST /game, /move).

Measured on the kit's practice trial-0 (local harness):
- 15 games: 5W 1D 9L, Elo 1218 (game 16 unplayed — the local run hit
  its 3,000 s shell timeout; the engine itself stayed within the
  3,600 s trial wall in projection).
- Packed main.py verified end-to-end on games 1–4: 3W 1L, Elo 1172,
  identical to the unpacked engine.
- For reference: the earlier 4,096-byte material-only engine scored
  Elo 0 (0W 0D 16L); the unpacked prototype scored Elo 1131.
- Honest status: 1218 is below the house baseline (sunfish 4K, mean
  Elo 1360), so this entry would not place. The limiter is search
  speed in Python (~10–15k nodes/s), which caps the depth the node
  budget can buy.
- Round 2 (Nov 12–13) tried PVS + LMR + null-move + futility pruning
  (proto6, Elo 1173), the same without null-move (Elo 1040), and
  null-move + PVS only (proto7, Elo 1087) — none beat this engine, so
  the entry stays as the round-1 build. Details in
  ../chess-dev/REPAIR_NOTES.md.

Run: `MAKEFUN_ENV_URL=<url> python3 main.py`
