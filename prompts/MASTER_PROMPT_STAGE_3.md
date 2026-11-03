STAGE 3: NAME PROMPT AND LEADERBOARD

Prerequisite: Stages 1 and 2 are done. `game.html` (the game, with `?embed` mode) and `index.html` (the landing page that embeds it) both exist and work. This stage adds a name prompt before every round and a leaderboard, in both files. Do not change the rep-counting logic. Stage 1 listed the leaderboard as out of scope; this stage overrides that.

## Hard requirement: no mock data
The leaderboard shows ONLY real rounds played in this browser. No seed rows, no placeholder names, no fake scores, no backend. When nothing has been played, show an empty state. All data lives in `localStorage`.

## Storage contract (shared by both files, same origin)
- `67name`: the last name the player typed (string), used to pre-fill the prompt.
- `67board`: JSON array of entries `{ name: string, score: number, t: number }` where `t` is `Date.now()` at the end of the round. Sorted by `score` descending, ties broken by earlier `t` first. Keep at most 50 entries, display the top 10.
- Wrap every `localStorage` and `JSON.parse` access in try/catch and fall back to an empty array or null. The page must work with storage blocked.
- `67best` (personal best) from Stage 1 stays as is.

## game.html changes
1. **Name prompt on every Start.** Pressing Start no longer calls `run()` directly. It opens a modal:
   - Markup (add after the leaderboard section): `<div id="modal" hidden><form id="nameForm"><label for="name">What is your name?</label><input id="name" maxlength="16" required autocomplete="off"><button>Start round</button></form></div>`
   - Label above the input, no placeholder-as-label. The input is pre-filled with `67name` and focused when the modal opens.
   - On submit: `preventDefault`, trim the value and cut it to 16 characters, ignore if empty, save it to `67name`, hide the modal, store it in `playerName`, then call `run()`. The round never starts without a name.
   - Escape closes the modal without starting.
   - Style: fixed full-screen overlay (`inset:0`, `background:#000a`, centered, `z-index:20`), card using the existing `--card` color with a 1px border and 16px radius, 360px max width. The input uses the page bg, a visible border and a 2px accent focus outline. Add `#modal[hidden]{display:none}` because the overlay sets `display:flex`.
2. **Save the score.** In `finish(total)`, after the personal-best logic: build `entry = { name: playerName, score: total, t: Date.now() }`, write `[...getBoard(), entry]` sorted as above and sliced to 50, then call `renderBoard(entry.t)`.
3. **Leaderboard section** under the result card: `<section id="board"><h2>Leaderboard</h2><ol id="rows"></ol></section>`, width `min(100%, 720px)`, the `--card` background, 16px radius.
   - Rows are `<li>` built with `createElement` and `textContent` (never `innerHTML`, names are untrusted). Each row has a rank (via a CSS counter), the name (ellipsis on overflow) and the score (bold, accent color), separated by a thin top border.
   - The entry just saved gets class `me` (accent-tinted background, rounded) by matching its `t`.
   - Empty state: a single row "No scores yet. Be the first."
   - Call `renderBoard()` once on load.
4. **Embed mode:** add `.embed #board { display:none; }` so the hero iframe stays compact. The name modal still works inside the iframe.
5. Mark the storage helpers with a comment: `// ponytail: leaderboard lives in localStorage, so it is per browser. Swap getBoard/setBoard for fetch calls to a backend to share it.`

## index.html changes
1. **Nav:** add a "Leaderboard" link (`#leaderboard`) between "How it works" and "Tips". The nav must still fit on one line at desktop.
2. **New section** directly under the hero and before "How it works": `<section id="leaderboard"><div class="wrap"><h2 class="reveal">Leaderboard</h2><ol class="lb reveal" id="lb" style="--d:.1s"></ol></div></section>`
3. **Styling:** the list is max 820px wide, no bullets, rank numbers from a CSS counter (large, weight 800, dim; the top 3 ranks in the accent color). Rows are 20px-radius with padding `16px 20px`, with the odd rows tinted to the `--surface` color (no hairline borders, because ten bordered rows is the banned long-list pattern). Name is bold 1.25rem with ellipsis, score is weight 800, 1.6rem, accent. Empty state row: "No scores yet. Play a round and you will show up here."
4. **Script:** `renderBoard()` reads `67board` (try/catch), shows the top 10 using `textContent`, runs on load, and re-runs on `window` `storage` events where `e.key === "67board"`. The hero iframe is same-origin, so the board updates live when a round ends inside it.

## Hard rules (unchanged from Stage 2)
Zero em-dashes or en-dashes in visible text. One accent color, same radii rules, hero still fits the first viewport, motion stays gated behind `prefers-reduced-motion: no-preference`, no `scroll` listeners, no decorative dots, all contrast passes AA in dark and light mode.

## Verify before you finish
1. Serve the folder (`python -m http.server 8067`) and open `http://localhost:8067/game.html` with empty storage. The leaderboard shows only the empty state. Press Start: the name modal appears with a focused input. Escape closes it. Submitting an empty name does nothing.
2. Play (or, if no camera is available, call the save path from the console) so at least two entries with different scores exist. Confirm the order, the `me` highlight, and that a name like `<b>x</b>` is displayed as literal text.
3. Open `http://localhost:8067/`: the Leaderboard section shows the same entries, and playing a round in the hero iframe updates it without a reload.
4. Reload with storage cleared and confirm both pages show the empty state and nothing throws.
5. Search both files for `—` and `–` in visible text. Count must be 0.
6. Report anything you could not check instead of claiming it works.

Out of scope: a shared or online leaderboard (needs a backend), accounts, deleting or editing entries, anti-cheat, 1v1 modes.
