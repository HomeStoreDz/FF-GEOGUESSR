# FF-GEOGUESSR

Arabic GeoGuessr-style game on the Free Fire Bermuda map. Solo mode works offline-first; Room mode (2 players) uses Supabase.

## Files
| File | Purpose |
|---|---|
| `index.html` | The whole game (design, solo mode, multiplayer). |
| `supabase.sql` | Database setup. Run once in Supabase. |
| `bermuda_map.webp` + location `.jpg` files | Your existing images (keep them next to `index.html`). |

## Setup (in this order)
1. **Supabase:** open your project → **SQL Editor** → **New query** → paste all of `supabase.sql` → **Run**. You should see "Success". It is safe to run again.
2. **GitHub:** open your repo → **Add file → Upload files** → upload `index.html` and `README.md` (replace the old `index.html`) → **Commit**. Keep all images in the same folder.
3. **Pages:** repo → **Settings → Pages** → Source: `main` branch, `/ (root)`. Wait ~1 minute, open https://homestoredz.github.io/FF-GEOGUESSR/ .
4. Hard-refresh the page (Ctrl+F5, or clear the site's cache on Android).

## What the SQL changes
- **Reused:** table `public.rooms` and all its data and columns.
- **Added (only if missing):** `host_token`, `guest_token`, `game_state`, `is_private`, `host_seen`, `guest_seen`.
- **Removed on purpose:** every old policy on `rooms`, old functions named `create_game_room`, `join_game_room`, `get_game_room`, `update_game_state`, and rooms older than 1 day (cleaned when a new room is created). `update_game_state` is gone because letting the browser write game state would let anyone cheat.
- **Realtime:** `rooms` is removed from the `supabase_realtime` publication, because streaming rows would leak the player tokens. Instead, a trigger sends an empty "changed" ping and the browser re-reads its own room through a token-checked function. Polling every 2.5 s is the backup.
- **Created:** 9 functions the browser may call: `create_game_room`, `list_open_rooms`, `join_game_room`, `get_game_room`, `start_game`, `submit_guess`, `tick_game`, `next_round`, `leave_game_room` (plus private helpers named `ff_*`).

## How rooms stay secure without a password
- The room code is 6 random characters (about 1 billion combinations), made by the database.
- Each player gets a long random token. It is stored only in that player's browser and checked by the database on every action.
- Tokens are never returned by any list or read function. The table has no policies and no direct access.
- **Public rooms** appear in the search list (anyone can join them, like a public lobby). Tick **"Private room"** when creating, and only people you give the code to can join.
- Scores are calculated by the database from your guess, not by the browser.

## Test solo
Open the site → type a name → **Solo** → pick difficulty → play. Your best score is still saved on the device.

## Test multiplayer (two browsers or two phones)
1. Device A: name "A" → **Room** → **إنشاء غرفة**. Note the code.
2. Device B: name "B" → **Room** → **البحث عن غرفة** → tap the room (or type the code).
3. A sees B join. A presses **ابدأ اللعب**.
4. Both guess; each sees "waiting for opponent", then both pins and scores. Press **Next** on both.
5. After round 15 you see the final result. The host can start a new match.
6. Try refreshing one browser mid-match: it should return to the same room.

## Debugging
In `index.html`, change `const DEBUG=false;` to `true` to see technical details in the browser console (F12). Set it back to `false` before sharing. Tokens are never logged.

## Known limitations
- I could not run the SQL or the game against your live Supabase from my environment, so **you must test** (checklist in the chat answer).
- The location coordinates are inside `index.html` (needed for solo mode), so a technical player can read them. The server stops *score tampering*, not *reading the answers*.
- No rate limiting. Someone could spam-create rooms. Fine for a small game; Supabase has its own API limits.
- Two players per room only.
- If a player closes the tab, the other sees "⚠️ غير متصل" after 30 s; missed rounds score 0, and the other can leave (a mid-match leaver forfeits).
- Unfinished rooms older than 1 day are deleted automatically.

## Copyright and monetization
The map and screenshots come from Free Fire (Garena). Unless you have permission from the rights holder, treat them as **not yours**. Keep the game free, non-commercial and clearly "unofficial"; ads, selling access or donations can create legal risk. For a public release, ask Garena or use your own artwork.
