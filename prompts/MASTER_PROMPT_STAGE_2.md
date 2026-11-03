STAGE 2: LANDING PAGE

Prerequisite: Stage 1 is done. `index.html` (the 67 Speed webcam game) already exists in this folder and works. Do not rewrite the game logic. If the `design-taste-frontend` skill is available, invoke it first and follow its rules and pre-flight check. Then add a marketing landing page that embeds the real game.

Start with this one-line design read: "Reading this as: consumer game landing for a fast-scrolling meme-and-challenge audience, with a bold playful language, leaning toward native CSS plus one mint accent that matches the game." Dials: DESIGN_VARIANCE 8, MOTION_INTENSITY 6, VISUAL_DENSITY 3.

## Files
1. Edit `index.html`: add an embed mode (see below). Touch nothing else in it.
2. Create `landing.html`: one self-contained file, vanilla HTML and CSS, one small inline script. Served at `http://localhost:8067/landing.html` (`python -m http.server 8067`). All "Play now" buttons link to `index.html`.

## Embed mode for index.html
- In `<head>`, before the style block: `<script>if (location.search.includes("embed")) document.documentElement.classList.add("embed");</script>`
- CSS: `.embed body { padding:8px; justify-content:center; }`, `.embed h1, .embed body > p { display:none; }`, `.embed #stage { width:min(100%, calc((100dvh - 90px) * 4 / 3)); }`, `.embed #result { position:fixed; left:50%; top:42%; transform:translate(-50%,-50%); z-index:5; box-shadow:0 20px 60px #000a; }`

## Stack and assets
- Font: Outfit 400, 700, 800 via `@font-face` from `https://cdn.jsdelivr.net/npm/@fontsource/outfit@5.0.8/files/outfit-latin-{400|700|800}-normal.woff2`, `font-display: swap`. Not Inter.
- Icons: Phosphor, `<script src="https://unpkg.com/@phosphor-icons/web@2.1.1"></script>`, used as `<i class="ph ph-camera"></i>`. No hand-drawn SVG icons, no emoji.
- No images are generated. The real product (the embedded game) is the hero visual. No div-based fake screenshots.

## Design tokens (one palette, dark default plus light via prefers-color-scheme)
- Dark: bg `#0d0f12`, surface `#151920`, line `#252b36`, text `#eef1f5`, dim `#9aa3b2`, accent `#7cf5a8`, on-accent `#06210f`, gradient `#7cf5a8` to `#2fbf74` with text `#06210f`.
- Light: bg `#f3f5f8`, surface `#ffffff`, line `#d9dee6`, text `#10151c`, dim `#4f5a6a`, accent `#0b7a43`, on-accent `#ffffff`, gradient `#0b6b3a` to `#0d7c43` with white text.
- No purple, no beige or brass. One accent used identically everywhere. No pure black or white.
- Shape rule: cards and frames 20px radius, buttons and pills fully round. Nothing else.
- Headings weight 800, letter-spacing -0.03em, `text-wrap: balance`. Body 17px, line-height 1.55, paragraphs max 65ch.
- Buttons: pill, accent fill, hover lifts 2px with a soft accent shadow, active scales to 0.98, labels never wrap. Ghost variant has a 1px line border. Visible 3px accent focus ring on keyboard focus.

## Page structure (exact copy, use hyphens only, zero em-dashes or en-dashes anywhere)
1. **Nav**: sticky, 64px, one line, blurred bg, bottom hairline. Left wordmark "67 Speed" (the word Speed in accent). Right: links "How it works" (#how), "Tips" (#tips), "FAQ" (#faq), and a small "Play now" button. On screens under 860px hide the three links and keep the button.
2. **Hero**: grid 5fr / 6fr, gap 48px, items centered, top padding about 32px. Left: h1 "How fast is your 67?" with "67" in accent, size `clamp(2.75rem, 5.4vw, 4.75rem)`, max 2 lines. Subtext (max 30ch wide): "Pump your arms at your webcam for 20 seconds. We count every rep right in your browser." Two buttons: "Play now" with a `ph-arrow-right` icon, and a ghost "How it works" linking to #how. Nothing else in the hero (no eyebrow, no tagline, no logos, no scroll cue). Right: a rounded frame with `aspect-ratio: 8 / 7`, `max-height: calc(100dvh - 9rem)`, dark bg `#0b0d12` in both modes, soft shadow, containing `<iframe src="index.html?embed" title="67 Speed live game" allow="camera">` filling it. Under 860px stack the columns and give the frame `height: calc((100vw - 32px) * .75 + 100px)`.
3. **How it works** (id `how`): h2 "Three moves. Twenty seconds." Then a 3-cell bento, exactly three cells, grid `5fr 7fr` with the third cell spanning full width, 16px gaps, each cell padded 28px and with its own surface:
   - "Allow the camera" (`ph-camera`): "Find good light and step back until your shoulders show. Seated works too." Background: dot pattern made with `radial-gradient` on the surface color.
   - "Pump for 20 seconds" (`ph-timer`): "Move your arms up and down as fast as you can. Each wrist direction change is one rep." Background: the accent gradient, with a huge decorative "20" at 22% opacity in the bottom-right corner (`aria-hidden`).
   - "Beat your best" (`ph-trophy`, large, accent): "Your score and reps per second appear at the end. Your best is saved on this device." Background: a soft accent-tinted gradient fading to the surface.
   Under 860px the bento becomes one column.
4. **Statement**: h2 "Your video never leaves this tab." (with "this tab." in accent), size `clamp(2.5rem, 7vw, 6rem)`, max-width 14ch. Paragraph: "Pose tracking runs on your device with Google MediaPipe. Nothing is uploaded and there is no signup."
5. **Tips** (id `tips`): h2 "Tips for a higher score". Below, a horizontal scroll-snap row (`scroll-snap-type: x mandatory`, `tabindex="0"`, aria-label) of five bordered 20px cards, each `flex: 0 0 min(300px, 78vw)`, with an accent icon and a bold sentence:
   - `ph-sun`: "Face a window or lamp so your arms are well lit."
   - `ph-arrows-out`: "Step back until your shoulders show. Hips are optional."
   - `ph-hand-waving`: "Keep both wrists in frame for the whole round."
   - `ph-t-shirt`: "Wear sleeves that contrast with the wall behind you."
   - `ph-armchair`: "Seated is fine. Keep your elbows clear of the desk."
6. **FAQ** (id `faq`): h2 "Questions". Native `<details>` accordion, max-width 820px, hairline between rows, a `ph-plus` icon that rotates 45 degrees when open. Questions and answers:
   - "How are reps counted?" "Each time a wrist reverses direction after travelling a set distance, it counts as one rep. The distance scales with your body size, so it does not depend on how far you stand."
   - "Is my video uploaded?" "No. Tracking runs in your browser. Only the pose model files are downloaded."
   - "Why is my count lower than expected?" "Most webcams capture about 30 frames per second, so very fast pumping can be undercounted. Good light and both wrists in frame help."
   - "Does it work on a phone?" "It should wherever your browser allows camera access. A laptop webcam is the safest choice for frame rate."
   - "Is there a leaderboard?" "Not yet. Your best score is saved on this device."
7. **Final call to action**: a full-width rounded block with the accent gradient and on-gradient text. h2 "20 seconds. Go." (size `clamp(2.5rem, 7vw, 5.5rem)`) and one "Play now" button with `ph-arrow-right`, inverted colors (on-gradient background, gradient-colored text).
8. **Footer**: one line, 15px: "67 Speed is an independent webcam game. Pose tracking by Google MediaPipe."

## Motion (MOTION_INTENSITY 6, every effect motivated)
- Hero entrance: h1, subtext, buttons, game frame fade up 24px in sequence (0, 0.08, 0.16, 0.24s delays), cubic-bezier(.16,1,.3,1). Purpose: hierarchy.
- Scroll reveal: elements with class `reveal` fade up 28px once as they enter the viewport. Use `IntersectionObserver` (threshold 0.15, unobserve after firing). Add `js` class to `<html>` in the head so content stays visible without JavaScript. Bento cells and the FAQ use small staggered `--d` delays.
- Hover and active feedback on buttons, as above.
- Everything animated sits inside `@media (prefers-reduced-motion: no-preference)`. Animate only transform and opacity. No `scroll` event listeners, no marquees, no parallax, no custom cursors.

## Hard rules (design-taste pre-flight, do not break)
- Zero em-dashes or en-dashes. Zero eyebrow labels above headings. No section numbers, no decorative status dots, no pills over images, no version labels or footers, no locale or time strips, no scroll cues.
- No three-equal-cards feature row. No split header (big headline left plus small paragraph right). At most two consecutive image-and-text splits (there are none).
- Hero fits the first viewport on a laptop: headline 2 lines, subtext 20 words or fewer, CTAs visible without scrolling.
- One intent per CTA label: every button that starts the game says "Play now".
- Mobile collapse under 860px is explicit per section. Use `min-height` and `dvh`, never `h-screen`.
- Contrast: all button, body and form text passes WCAG AA in both color modes.
- z-index only on the nav (10) and the embedded result overlay (5).

## Verify before you finish
1. Serve the folder and open `http://localhost:8067/landing.html`. Take screenshots at about 1512 px wide of the hero, the bento, the tips row, the FAQ and the final block, in dark mode. Also check light mode and a 390 px phone width.
2. Confirm the game inside the hero loads, shows "Press Start", and that Start asks for camera permission (the camera must be allowed for `localhost`).
3. Search the file for `—` and `–`. The count must be 0.
4. Report anything you could not check, instead of claiming it works.

Out of scope: leaderboard, 1v1, accounts, generated imagery (offer it at the end instead), a theme toggle, and a build step.
