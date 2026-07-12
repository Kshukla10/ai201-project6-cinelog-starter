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

## Comment 2 — Deduplication
**What I did:** Added a dedup check inside `add_to_watchlist()` that queries
for an existing `WatchlistEntry` matching the same `user_id` and `film_id`,
raising a new `AlreadyInWatchlistError` if one is found — mirroring the
pattern in `add_to_collection()`/`AlreadyInCollectionError`.
**How I verified:** Ran the full test suite to confirm nothing broke, and
manually tested via curl that adding the same film twice now returns an
error instead of creating a duplicate row.

## Comment 3 — Missing test
**What I did:** Created `tests/test_watchlist.py` with a test for the
nonexistent-film case, following the same fixture and assertion pattern as
`test_add_to_collection_nonexistent_film_raises` in `test_collection.py`.
Used a fake integer film_id (999999) rather than a UUID string, since
`Film.id` is still `db.Integer` at this point in the branch (pre-Comment-6
rebase).
**How I verified:** Ran `pytest tests/test_watchlist.py -v` (passed) and
the full suite `pytest tests/ -v` (all 5 tests passing, nothing broken).

## Comment 4 — Default visibility

**My position:**
Watchlist entries should default to `public=True`.

**Reasoning:**
CineLog's core loop depends on discovery, social proof, and shared taste.
A watchlist is aspirational — it answers "what am I planning to watch
next?" — which makes it a natural conversation starter in a community app:
friends can spot overlapping watchlists for movie nights, and users can
draw inspiration from people they follow. A collection (already watched)
and a watchlist (intend to watch) are different kinds of data, but both
drive discovery for CineLog specifically, so both benefit more from
visibility than privacy. Defaulting to private would quietly undercut that
for the majority of users who never touch their settings.

**Tradeoff acknowledged:**
The real cost is that some entries reveal more than intended — a "guilty
pleasure" title added without a second thought about who sees it. Since
most users never change defaults, anyone who'd have preferred privacy is
exposed unless they opt out. The fix isn't a different default, but making
the privacy control impossible to miss — surfaced when adding a film, not
buried in settings — so opting out takes one tap, not a search.

## Comment 5 — Sort order

**My position:**
Switch the default sort order to `date_added.desc()` (most recent first) to
match the maintainer's request and the collection's existing behavior, but
treat this as a starting point — a user-controlled sort toggle (recency vs.
alphabetical) would serve the watchlist better long-term.

**Reasoning:**
A watchlist's ideal order shifts with size. When it's small, recency
answers the immediate question "what did I just add that I want to watch
tonight?" But as it grows into a backlog of dozens of films, recency turns
into a chaotic chronological ledger, and users naturally pivot to
scanning — checking alphabetically whether a film is already saved, or
browsing by title. Recency serves the short-term case well; alphabetical
becomes a real need as the list grows.

**Engagement with reviewer's point:**
The maintainer's reasoning — "most users want to see what they added
recently" — holds for collections, where "most recently watched" reflects
real activity. A watchlist is different: it's a buffer of intent, not a
history log. Someone might add *Citizen Kane* today with no plan to watch
it for a month, so its spot at the top of a recency sort quickly stops
meaning anything. I agree with `date_added.desc()` for consistency with
the rest of the app, but the order shouldn't be fixed — a toggle would
serve both "what did I just add" and "let me find something specific."