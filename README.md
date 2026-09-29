# Familiada — Family Feud for parties

One file, no install, no server. Open `index.html` in Chrome, Edge or Firefox.

1. **Questions screen** (first thing you see): type questions, paste JSON, or click **Import JSON…** to load `questions.json`. Click **Start game ▶**.
2. On the home screen click **Host view** (your laptop) and **Board (TV)**. Drag the board window to the TV/projector and double-click it or press `F` for fullscreen (the first click also enables sounds).
3. When a team answers, click **✓ Correct** next to the matching answer (or press `1`–`8`), or **✗ WRONG** (`X`).
4. At the end of a round, click **Award to …** (the bank × round multiplier goes to that team), then **Reveal remaining** and **Next ▶**.
5. **New game…** on the host view goes back to the questions screen.

Question text format: a blank line between questions, the question on the first line, then one `answer points` per line (up to 8). End the question line with `x2` or `x3` to make that round double or triple points by default (the host can still change it).
JSON format (`questions.json`): `[{ "question": "...", "multiplier": 2, "answers": [{ "text": "...", "points": 30 }] }]` (`multiplier` is optional, default 1). **Export JSON** saves the questions from the box.

The game state is saved in the browser, so refreshing either window is safe.

Hosting online instead? Drop `index.html` on any static host (GitHub Pages, Netlify). Host and board must run in the same browser on the same computer.
