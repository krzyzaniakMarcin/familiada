# Familiada — Family Feud for parties

One file, no install, no server. Open `index.html` in Chrome, Edge or Firefox.

1. **Setup screen** (first thing you see): enter the team names, type questions, paste JSON, or click **Import JSON…** to load `questions.json`. Click **Start game ▶**.
2. The host view opens on your laptop together with a board window (if the popup was blocked, click **📺 Open board**). Drag the board window to the TV/projector and double-click it or press `F` for fullscreen. Sounds play from the host window, so they come out of whatever speakers the computer uses (e.g. the TV over HDMI).
3. When a team answers, click **✓ Correct** next to the matching answer (or press `1`–`8`), or **✗ WRONG** (`X`).
4. At the end of a round, click **Award to …** (the bank × round multiplier goes to that team), then **Reveal remaining** and **Next ▶**.
5. **New game…** on the host view goes back to the questions screen.

Question text format: a blank line between questions, the question on the first line, then one `answer points` per line (up to 8). End the question line with `x2` or `x3` to make that round double or triple points by default (the host can still change it).
JSON format (`questions.json`): `[{ "question": "...", "multiplier": 2, "answers": [{ "text": "...", "points": 30 }] }]` (`multiplier` is optional, default 1). **Export JSON** saves the questions from the box.

**Custom sounds:** put `reveal.mp3`, `strike.mp3` and `award.mp3` in the `sounds/` folder next to `index.html` to replace the built-in sounds.

The game state is saved in the browser, so refreshing either window is safe.

Hosting online instead? Drop `index.html` on any static host (GitHub Pages, Netlify). Host and board must run in the same browser on the same computer.
# familiada
