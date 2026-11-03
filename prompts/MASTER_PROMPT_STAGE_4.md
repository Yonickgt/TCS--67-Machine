STAGE 4: ZOOM INTO PLAY MODE

Prerequisite: Stages 1 to 3 are done. `game.html` (game, `?embed` mode, name prompt, leaderboard) and `index.html` (landing page with the hero game iframe and a Leaderboard section) exist and work. This stage makes the landing page zoom the game into a full "play mode" with the leaderboard beside it and a Back button, whenever the player starts. Do not change the rep counting or the storage format (`67name`, `67board`).

## Behaviour
1. The hero game stays in the page as before. When the player presses Start inside it, OR presses any "Play now" button on the landing page (nav, hero, final call to action), the page zooms into play mode and the name prompt opens.
2. Play mode layout (desktop): a fixed top bar 64px high with the "67 Speed" wordmark on the left and a Back button at the top right. Under it, the game fills the left side (everything except a right panel of `min(380px, 32vw)`), and the right panel shows the top-10 leaderboard with a "Leaderboard" heading. The page behind does not scroll.
3. The player stays in play mode until they press Back (or Escape). Back zooms the frame back into the hero. A round that is already running keeps running in the hero.
4. The side panel updates live when a round ends, from the same `67board` data. No mock data, empty state "No scores yet. Play a round and you will show up here."
5. Phone layout (under 860px): the game on top, the leaderboard panel below it taking the bottom 42dvh, full width, with a top border instead of a left border.

## game.html (two small additions at the end of the name-prompt code)
- In the Start button's click handler, first send the parent a message: `if (parent !== window) parent.postMessage({ type: "67:start" }, location.origin);` then open the name modal as before.
- Add a listener so the landing page can open the prompt remotely: `addEventListener("message", e => { if (e.origin === location.origin && e.data && e.data.type === "67:open") $("start").click(); });`. Always check `e.origin`.

## index.html
1. **Markup**, placed between `</nav>` and `<main>`:
   - `<div id="playbar"><a class="brand" href="#top">67 <b>Speed</b></a><button class="btn ghost sm" id="back" type="button"><i class="ph ph-arrow-left"></i> Back</button></div>`
   - `<aside id="playpanel"><h2>Leaderboard</h2><ol class="lb" id="pb"></ol></aside>`
2. **CSS** (document the z-index scale in a comment: nav 10, play overlay 30, play bar 40):
   - `button.btn { font-family:inherit; cursor:pointer; }`
   - `.game { view-transition-name:game; }` and `::view-transition-group(game) { animation-duration:.55s; animation-timing-function:cubic-bezier(.16,1,.3,1); }`
   - `#playbar, #playpanel { display:none; }`, `body.play { overflow:hidden; }`
   - `body.play #playbar`: flex, `position:fixed` top, full width, 64px, `z-index:40`, items centered, space-between, 16px side padding, page background, bottom hairline.
   - `body.play .game`: `position:fixed; z-index:30; top:64px; left:0; bottom:0; right:min(380px,32vw); height:auto; max-height:none; aspect-ratio:auto; border-radius:0; border:0; box-shadow:none;`
   - `body.play #playpanel`: `display:block; position:fixed; z-index:30; top:64px; right:0; bottom:0; width:min(380px,32vw); padding:24px 20px;` page background, left hairline, `overflow:auto`. Inside it the heading is 1.6rem, and the `.lb` rows are tighter (`padding:12px 14px`, name 1.05rem, rank and score 1.25rem, `max-width:none`).
   - In the `max-width: 860px` block: `body.play .game { right:0; bottom:42dvh; }` and `body.play #playpanel { top:auto; left:0; width:100%; height:42dvh; border-left:0; border-top:1px solid var(--line); }`
   - Do not put any transform, filter or `contain` on ancestors of `.game`, or `position:fixed` breaks.
3. **Script**:
   - `renderBoard()` now fills both lists: loop over `["lb", "pb"]`, each built with `createElement` and `textContent`.
   - `setPlay(on)`: return early if `document.body.classList.contains("play") === on`. Otherwise toggle the `play` class on `<body>` inside `document.startViewTransition(...)` when that API exists and the user does not prefer reduced motion, else toggle directly. The View Transition is what gives the zoom morph.
   - `window` `message` listener: if `e.origin === location.origin && e.data && e.data.type === "67:start"`, call `setPlay(true)`.
   - `#back` click calls `setPlay(false)`. A `keydown` listener on Escape also calls `setPlay(false)`.
   - Every anchor `a[href="game.html"]` gets a click handler: ignore modified clicks (`metaKey`, `ctrlKey`, `shiftKey`, `button`), otherwise `preventDefault()`, `setPlay(true)`, then `document.querySelector(".game iframe").contentWindow.postMessage({ type: "67:open" }, location.origin)`. Keep the `href` as the fallback for no JavaScript and middle-click.

## Hard rules (unchanged)
Zero em-dashes or en-dashes in visible text, one accent color, same radii, motion gated behind reduced-motion, no `scroll` listeners, AA contrast in dark and light mode. The standalone `game.html` page keeps working on its own.

## Verify before you finish
1. Serve the folder (`python -m http.server 8067`), open `http://localhost:8067/`. Press Start in the hero game: the frame zooms to fill the screen, the leaderboard panel appears on the right, Back is at the top right, and the name prompt is open.
2. Press Back (and separately Escape): you return to the hero and the page scrolls normally again.
3. Press the nav "Play now", the hero "Play now" and the final "Play now": each enters play mode and opens the name prompt. Ctrl-click one: it opens `game.html` in a new tab instead.
4. Finish a round in play mode: the side panel shows the new score without a reload.
5. Check 390px width (panel below the game) and light mode, and that a user with reduced motion gets no morph but the same layout.
6. Search both files for `—` and `–` in visible text. Count must be 0. Report anything you could not check instead of claiming it works.

Out of scope: a shared online leaderboard, fullscreen API, sound, 1v1 modes.
