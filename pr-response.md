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
**My position:** I'm keeping `public=True` as the default for watchlists.

**Reasoning:** A watchlist is meant to be seen — it's how other users find out what someone is planning to watch, which supports the "community" part of a community film-tracking app. If watchlists defaulted to private, that discovery function would mostly disappear, since most users won't go out of their way to change a default. Keeping it public by default optimizes for visibility and engagement between users, which is core to what CineLog is for.

**Tradeoff acknowledged:** The risk is that a user adds something more personal or unfinished to their watchlist — an item they're still deciding on, or something a bit off-brand for how they present themselves — without realizing it's visible to everyone by default. Unlike a `CollectionEntry` (a completed, settled fact — "I watched this"), a `WatchlistEntry` reflects an in-progress intention, which is a more exposing thing to share involuntarily. That's part of why the model gives watchlists a `public` toggle at all, while collections have none — the developers already recognized watchlists carry more exposure risk. I'm accepting that risk in exchange for the discovery value, but it's worth flagging that a clearer onboarding hint (e.g., a one-time note the first time a user adds something) could mitigate the downside without changing the default itself.

## Comment 5 — Sort order
**My position:** I agree with switching to date-added order (newest first) instead of alphabetical.

**Reasoning:** This also matches the existing precedent in `get_collection()`, which already sorts by `date_added.desc()` — alphabetical order in `get_watchlist()` was actually the inconsistent one, not the norm. Alphabetical sort optimizes for finding a specific known title fast, like a contacts list, but a watchlist is more of a recency-driven feed of "what I'm currently excited about." A user is more likely to want to see what they just added, or clear out the oldest item they've been meaning to get to, than to scan alphabetically for one title.

**Engagement with reviewer's point:** The reviewer's reasoning — "most users want to see what they added recently" — lines up with how the rest of the app already treats film lists, so I don't see a strong reason to keep watchlists as the outlier. The tradeoff I'm accepting is that alphabetical is genuinely better if someone has a long watchlist and is hunting for one specific title they remember by name — date-added order makes that harder. A reasonable follow-up (not implemented here) would be exposing sort order as a query parameter, defaulting to date-added but letting a user request alphabetical when they want it.

## Comment 6 — Rebase
**What conflicted:** While my PR was open, `main` was refactored to change film IDs from integers to UUIDs (`Film.id`, and all foreign keys referencing it, became `db.String(36)`). My `feature/watchlist` branch still used integer `film_id` values in `models.py` (the `WatchlistEntry` model) and in `tests/test_watchlist.py`. When I ran `git rebase origin/main`, the first conflict was in `.gitignore`, where both branches had independently added ignore rules. After resolving that, the rebase completed, but `WatchlistEntry` had been dropped entirely from `models.py` — the refactor commit on `main` only updated the models that existed there at the time (`User`, `Film`, `CollectionEntry`), so replaying history lost my `WatchlistEntry` class in the process.

**How I resolved it:** For `.gitignore`, I merged both versions into one list with no duplicates. For the missing model, I re-added `WatchlistEntry` to `models.py`, updating `film_id` to `db.String(36)` (UUID) to match the new `Film.id` type, consistent with how `CollectionEntry` was already updated by the refactor. I also updated `tests/test_watchlist.py` to use a fake UUID string (`"00000000-0000-0000-0000-000000000000"`) instead of an integer for the nonexistent-film test, matching the pattern in `test_collection.py`.

**How I verified no conflict remains:** Ran `pytest tests/ -v` after each fix — all tests passed with no import errors or type mismatches. Confirmed with `git log --oneline` that the branch history is now linear on top of `origin/main` with no merge commits.

## PR Description
<!-- written at the end -->

## Final Commit History
\```
15c51df docs: add rebase notes for comment 6
5f3a15b fix: restore WatchlistEntry model and update tests to use UUID film_id after rebase
305b0ff docs: add design decision for comment 5 - sort order
bc0a8ec fix: sort watchlist by date_added descending instead of alphabetical
5d5e791 docs: add pr-response entries for comments 1-4
5c780a2 test: add test for nonexistent film_id in add_to_watchlist
d68c846 fix: add deduplication check to prevent duplicate watchlist entries
8e5b978 fix: rename save_to_watchlist to add_to_watchlist per naming convention
42e331e fix: update film retrieval method to use db.session.get in collection and watchlist services
7de97c8 feat: add watchlist model and endpoints
\```