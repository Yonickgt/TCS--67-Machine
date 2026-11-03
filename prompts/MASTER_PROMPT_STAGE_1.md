Build a single-file website, `game.html`, that replicates "67 Speed" (67speedgames.com): a webcam arm-pumping game that counts reps with computer vision. No build step, no backend, no model training.

## Stack
- One `game.html`, vanilla JS in `<script type="module">`, inline CSS. No frameworks.
- Detection: Google MediaPipe Pose Landmarker via the pretrained model, running fully in the browser (video never leaves the device).
  - Import: `import { PoseLandmarker, FilesetResolver, DrawingUtils } from "https://cdn.jsdelivr.net/npm/@mediapipe/tasks-vision@0.10.14/vision_bundle.mjs"`
  - WASM fileset: `https://cdn.jsdelivr.net/npm/@mediapipe/tasks-vision@0.10.14/wasm`
  - Model: `https://storage.googleapis.com/mediapipe-models/pose_landmarker/pose_landmarker_lite/float16/1/pose_landmarker_lite.task`
  - Options: `delegate: "GPU"`, `runningMode: "VIDEO"`, `numPoses: 1`.
- Camera: `getUserMedia({ video: { width: 640, height: 480, frameRate: 60 } })`. Must be served over HTTPS or localhost.

## Game flow
1. Page shows title, a short instruction paragraph, a 4:3 stage (max 720px wide) with `<video>` plus an overlay `<canvas>` (both mirrored with `transform: scaleX(-1)`, `object-fit: cover`), and a Start button.
2. HUD over the stage: top-left timer ("20.0s"), top-right personal best ("Best N", hidden if none), a huge centered accent-colored rep number, and a small message pill at the bottom.
3. On Start: disable the button, lazy-load the model and camera (message "Loading model…" then "Starting camera…"; on error show the error message and re-enable the button), then a 3-2-1 countdown (1 s each, message "Get ready, arms up!"), then a 20-second round (reset the big number to 0, message "GO!").
4. At 20 s: stop counting, show a result card ("Score N reps · X.X reps/s" plus "New personal best!" or "Best: N"), change the button to "Play again". Save the best in `localStorage` (wrap every access in try/catch).
5. The skeleton (`POSE_CONNECTIONS`, green lines; white landmark dots) draws on the canvas every frame, including during the countdown and before Start once the camera is on.

## Rep counting (the core logic)
- Use wrist landmarks 15 and 16. Convert the normalized y to pixels (`y * canvas.height`).
- Ignore a wrist whose `visibility <= 0.5`.
- Distance-independent threshold: `thr = 0.15 * scale`, where `scale` is:
  - the average of the left and right shoulder-to-hip pixel distances (landmarks 11-23 and 12-24) when all four are visible;
  - otherwise `1.5 * shoulder width` (11-12) when only the shoulders are visible, so seated users at a laptop can play;
  - otherwise show "Step back: show your shoulders" and skip the frame.
- One counter per wrist. Smooth y with a 3-frame moving average. Zigzag reversal detection: track an extreme `ext` and a direction `dir` (0 unknown, 1 down, -1 up). While `dir >= 0` (moving down or unknown): if `v > ext` set `ext = v` (lowest point so far); else if `ext - v > thr` the wrist has reversed upward, so set `dir = -1`, `ext = v`, `count++`. While `dir <= 0` (moving up or unknown): if `v < ext` set `ext = v` (highest point so far); else if `v - ext > thr` set `dir = 1`, `ext = v`, `count++`. (Remember y grows downward in image coordinates.) Total reps = left count + right count; every direction change beyond the threshold is one rep.
- Keep the counter as two small pure functions, `makeCounter()` and `feed(state, y, thr)`, with no DOM access, so they can be tested in Node.

## Robustness requirements (these were real bugs)
- The round timer, the countdown display and the end-of-round check MUST run every animation frame, independent of whether a body is detected. Never put them behind the "body visible" early-return, or the round freezes when the user is too close.
- Skip pose detection when `video.currentTime` has not advanced; pass `performance.now()`-style increasing timestamps to `detectForVideo`.
- Show "No body found, step back" when no pose is detected during a round, and clear it ("GO!") once the body is visible again.

## Self-check
Add a tiny `console.assert` on page load: feed a synthetic 5 Hz sine (amplitude 100px, 30 fps, 60 frames, thr 20) to the counter and assert the count is between 15 and 21. Also verify in Node that a 2 Hz sine gives about 8 reps in 2 s and pure random jitter (±3px) gives 0.

## Style
Dark theme (CSS variables: bg `#0b0d12`, text `#f2f4f8`, dim `#8b93a7`, accent `#7cf5a8`, card `#151922`), system font, rounded 16px stage, accent-colored pill button, mobile-friendly with 16px side gutters.

## Out of scope (do not build)
Leaderboard, 1v1, friends battles, accounts, audio, other mini-games.

Deliver the complete file, then say how to run it: `python -m http.server 8067` and open `http://localhost:8067/game.html`.
