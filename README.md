# 67 Speed

A webcam arm-speed game. Pump your arms in front of your camera for 20 seconds and the page counts your reps with computer vision. Everything runs in your browser: no backend, no build step, no model training, and your video is never uploaded.

## Run it locally

You need a webcam and a recent Chrome or Edge. Clone the repo, then serve the folder with any static server (the camera only works on `localhost` or HTTPS, not from a double-clicked file).

```bash
git clone <this-repo-url>
cd 67-speed

# pick one
python -m http.server 8067
npx serve -l 8067
```

Open **http://localhost:8067/** for the landing page (press Play now to zoom into the game), or **http://localhost:8067/game.html** for the game on its own. Allow the camera when asked.

The first load downloads the pose model and fonts from CDNs (jsdelivr, unpkg, Google storage), so you need an internet connection.

## How it works

- **Detection:** [MediaPipe Pose Landmarker](https://ai.google.dev/edge/mediapipe/solutions/vision/pose_landmarker) (pretrained "lite" model, WebAssembly and GPU in the browser).
- **Counting:** each wrist is tracked on its own. A rep is counted every time the wrist reverses direction after moving more than 15% of your shoulder-to-hip distance (or 1.5 times shoulder width when your hips are out of frame). The threshold scales with your body, so camera distance does not matter much.
- **Rate limit:** webcams run at about 30 fps, so pumping faster than roughly 8 to 10 reps per second gets undercounted.
- **Tuning:** if the count feels high or low, change `THRESH` near the top of the script in `game.html` (lower is more sensitive).

## Files

| File | What it is |
|---|---|
| `game.html` | The game: camera, counter, name prompt, leaderboard. Also runs in `?embed` mode inside the landing page. |
| `index.html` | Marketing page. Embeds the game, zooms into a full play view with the leaderboard beside it, has a Back button. |
| `prompts/` | Four master prompts (stages 1 to 4) that rebuild the project step by step with Claude Code. |

## Your data

Names and scores are saved in your browser's `localStorage` (`67name`, `67board`, `67best`). There is no shared leaderboard, so scores stay on the device and browser that played them. To reset, clear site data for `localhost:8067`, or run `localStorage.clear()` in the browser console.

## Credits

Pose tracking by Google MediaPipe. Icons by Phosphor. Font: Outfit. This is an independent project, inspired by 67speedgames.com and not affiliated with it.
