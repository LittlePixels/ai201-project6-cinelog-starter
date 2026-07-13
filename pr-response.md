# PR Response Doc — CineLog Watchlist Feature

## AI Usage
<!-- Fill in at the end — how you used AI tools during this project -->

## Comment 1 — Rename
**What I did:**
Renamed `save_to_watchlist()` to `add_to_watchlist()` in `services/watchlist_service.py`
(the function definition) and updated every call site.

**Where I looked to find all call sites:**
Before renaming, I ran a project-wide search for `save_to_watchlist` across the whole
repo. That surfaced exactly three references:
1. `services/watchlist_service.py:12` — the function definition itself.
2. `routes/watchlist/watchlist.py:8` — the `from services.watchlist_service import ...` line.
3. `routes/watchlist/watchlist.py:32` — the call inside the `add_film` route handler.
After editing all three, I re-ran the same project-wide search and it returned
**no matches**, confirming nothing was missed (no stray references in tests, other
services, or docs).

**How I verified:**
- Project-wide search for the old name returns zero matches.
- Ran the full suite (`pytest tests/ -v`): 4 passed, 0 failed — nothing broke.
- The rename is behavior-preserving; the new name matches the naming convention used
  by the sibling `add_to_collection()` in the collection service.

## Comment 2 — Deduplication
**What I did:**
Added deduplication to `add_to_watchlist()` in `services/watchlist_service.py`, following
the exact pattern used by `add_to_collection()` in `services/collection_service.py`:
- Defined a new `AlreadyInWatchlistError(Exception)` in the watchlist service, mirroring
  how the collection service defines its own `AlreadyInCollectionError`.
- Added the dedup check *after* the film-existence guard and *before* creating the entry:
  query `WatchlistEntry` filtered by `user_id` + `film_id`, take `.first()`, and if a row
  already exists, `raise AlreadyInWatchlistError` (no entry is created, nothing is committed).
- Updated the docstring's `Raises:` section to document the new exception.

**Model I followed (from Milestone 1 analysis of `add_to_collection`):**
The collection dedup does a read-then-check: `Entry.query.filter_by(user_id=..., film_id=...).first()`,
and if truthy, raises `AlreadyInCollectionError`. When a duplicate is detected it *raises*
(returns nothing) and performs no DB write. I reproduced that ordering and behavior rather
than relying only on the model's `UniqueConstraint`, so callers get a clean, named error
instead of a raw `IntegrityError`.

**How I verified the deduplication logic works:**
- Ad-hoc check against an in-memory SQLite DB (same config the tests use): added a film to
  a user's watchlist once (succeeds), then added the identical (user_id, film_id) again.
  Result: the second call raised `AlreadyInWatchlistError`, and a follow-up
  `WatchlistEntry.query...count()` returned **1** — confirming no duplicate row was written.
- Ran the full suite (`pytest tests/ -v`): 4 passed, 0 failed — no regressions.
  (A dedicated dupe test lives with the watchlist tests; see Comment 3 for the test file.)

## Comment 3 — Missing test
**What I did:**
**How I verified:**

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
<!-- Written at the end — feature overview, design decisions, manual testing steps -->
