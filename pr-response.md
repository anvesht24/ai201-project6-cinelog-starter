# PR Response Doc — CineLog Watchlist Feature

## AI Usage
<!-- fill in at the end -->

## Comment 1 — Rename
**What I did:** Renamed `save_to_watchlist()` to `add_to_watchlist()` in `services/watchlist_service.py` and updated the call site in `routes/watchlist/watchlist.py` to match the project's `verb_to_noun` convention used by `add_to_collection()`.
**How I verified:** Searched the whole codebase with `Select-String -Pattern "save_to_watchlist"` to confirm no remaining references, then ran `pytest tests/ -v` to confirm all tests passed.

## Comment 2 — Deduplication
**What I did:** Added a duplicate check in `add_to_watchlist()` that queries `WatchlistEntry` by `user_id` and `film_id` before inserting, raising a new `AlreadyInWatchlistError` if a match exists — mirroring the pattern in `add_to_collection()` (`FilmNotFoundError` check first, then duplicate check).
**How I verified:** Ran the full test suite; added coverage for this in `test_watchlist.py`.

## Comment 3 — Missing test
**What I did:** Created `tests/test_watchlist.py` following the fixture and structure pattern in `test_collection.py`, including `test_add_to_watchlist_nonexistent_film_raises`, mirroring `test_add_to_collection_nonexistent_film_raises`.
**How I verified:** Ran `pytest tests/test_watchlist.py -v` — all passed.

## Comment 4 — Default visibility
**My position:**
**Reasoning:**
**Tradeoff acknowledged:**

## Comment 5 — Sort order
**My position:**
**Reasoning:**
**Engagement with reviewer's point:**

## Comment 6 — Rebase
**What conflicted:**
**How I resolved it:**
**How I verified no conflict remains:**

## PR Description
<!-- written at the end -->