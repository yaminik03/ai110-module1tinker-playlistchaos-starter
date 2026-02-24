## Quick orientation for AI coding agents

This small Streamlit app (root: `app.py`) implements a toy "playlist engine" whose core logic lives in `playlist_logic.py`.
The UI and state live in `app.py` (Streamlit): it keeps three keys in `st.session_state` — `songs`, `profile`, and `history`.

When editing code, focus on these boundaries:
- UI layer: `app.py` — reads/writes `st.session_state`, renders controls and tabs, and calls into `playlist_logic` functions.
- Logic layer: `playlist_logic.py` — normalization, classification, playlist building, merging, searching, stats, lucky-pick, and history summarization.

Key data shape (Song): a dict with keys: `title` (str), `artist` (str, lowercased by normalize), `genre` (str, lowercased), `energy` (int), `tags` (list), and optional `mood` set by classification.

Notable project-specific patterns and quirks (use these explicitly when making edits):
- Default songs in `app.py:default_songs()` are plain dicts (not normalized). `build_playlists()` calls `normalize_song()` again, and `app.py` also normalizes songs when adding via the sidebar.
- `normalize_song()` lowercases `artist` and `genre`. Be consistent when comparing these fields elsewhere (compare lowercased values).
- `classify_song()` mixes substring checks and exact comparisons:
  - It checks if `genre == favorite_genre` OR energy >= `hype_min_energy` OR any hype keyword is in `genre` to return `Hype`.
  - It checks chill keywords incorrectly against the title (`any(k in title for k in chill_keywords)`), not the genre. This is an intentional quirk to pay attention to when fixing bugs.
- `merge_playlists()` assigns lists from `a` into the merged map and extends them with `b`'s lists; this may alias original lists (mutating inputs). If you need immutability, return copies.
- `compute_playlist_stats()` contains aggregation issues to watch:
  - `total` is computed as `len(hype)` (not total songs) and used for `hype_ratio`.
  - `avg_energy` sums energy over `hype` but divides by `len(all_songs)` — likely incorrect denominators.
- `search_songs()` expects substring matching but uses the wrong containment direction: it tests `value in q` instead of `q in value`. Use this to guide fixes and tests.
- `random_choice_or_none()` currently returns `random.choice(songs)` without handling empty lists — this raises if called with an empty list. Fix by checking emptiness and returning `None`.

Concrete places to look first (files & functions):
- `app.py` — `init_state()`, `default_songs()`, `add_song_sidebar()`, `playlist_tabs()`, `lucky_section()`, `history_section()`
- `playlist_logic.py` — `normalize_song()`, `classify_song()`, `build_playlists()`, `merge_playlists()`, `compute_playlist_stats()`, `search_songs()`, `lucky_pick()`, `random_choice_or_none()`

Developer workflow (how to run & test quickly):
- Run the app locally: `streamlit run app.py` from the repo root (macOS zsh).
- Use the sidebar to: change the profile, add songs, reset songs to default, and clear history.
- Reproduce issues by adding songs that exercise edge cases: missing artist/title, non-integer `energy`, `tags` as a string, empty playlists for lucky pick, and favorite genres that differ only by case.

Suggested minimal test/actions to validate fixes (small, manual):
- Fix `search_songs()` and then in the UI open a playlist and search for an artist substring.
- Fix `random_choice_or_none()` and test the "Feeling lucky" button when playlists are empty.
- Fix `compute_playlist_stats()` denominators and verify the metrics in the Stats section change accordingly.
- Fix `classify_song()` chill-keyword bug and validate songs with chill genres are classified as Chill.

When making code changes, add a one-line comment near each fix explaining the reason (the exercise expects small comments documenting fixes).

If you need to run unit tests or add quick ones, create a small test file under `tests/test_playlist_logic.py` that exercises the failing behavior (normalization, search, stats, and lucky pick). There are no tests in repo now — add minimal ones for CI if required.

If you update behavior that affects the UI contract (session keys or song dict shape), update `app.py` usage sites and leave a short note in `README.md` describing the change.

Ask for clarification if any behavior seems intentionally ambiguous (the README notes the app is "intentionally imperfect" and some quirks may be by design).

If you'd like, I can (a) open a PR that fixes the obvious bugs listed above (search, random pick empty check, stats math, chill keyword check), and (b) add minimal unit tests and a short changelog entry. Reply which you'd prefer me to implement.
