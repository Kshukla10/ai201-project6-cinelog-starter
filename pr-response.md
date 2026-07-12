# PR Response Doc — CineLog Watchlist Feature

## AI Usage
<!-- Fill in at the end — how you used AI tools during this project -->

## Comment 1 — Rename
**What I did:** Renamed `save_to_watchlist()` to `add_to_watchlist()` in
`services/watchlist_service.py` to match the project's `verb_to_noun`
convention used by `add_to_collection()`. Updated the import and call site
in `routes/watchlist.py`.
**How I verified:** Searched the project for any remaining references to
`save_to_watchlist` (found none) and confirmed `add_to_watchlist` appears
in exactly the three expected places (definition + import + call site).
Ran the full test suite to confirm nothing broke.