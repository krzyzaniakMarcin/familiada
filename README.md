# Familiada — Family Feud for parties

One file, no install, no server. Open `index.html` in Chrome, Edge or Firefox.

1. Click **Host view** on your laptop.
2. Click **📺 Open board window**, drag it to the TV/projector, click it once (fullscreen + sounds; `F` toggles fullscreen).
3. When a team answers, click **✓ Correct** next to the matching answer (or press `1`–`8`), or **✗ WRONG** (`X`).
4. At the end of a round, click **Award to …** (the bank × round multiplier goes to that team), then **Reveal remaining** and **Next ▶**.

Edit questions under **Edit questions** on the host view: a blank line between questions, the question on the first line, then one `answer points` per line (up to 8).
**Question bank as JSON:** edit `questions.json` (format: `[{ "question": "...", "answers": [{ "text": "...", "points": 30 }] }]`, 1–8 answers each) and load it with **Import JSON…** on the host view. **Export JSON** saves the current questions. Importing starts a new game (team names are kept).

The game state is saved in the browser, so refreshing either window is safe.

Hosting online instead? Drop `index.html` on any static host (GitHub Pages, Netlify). Host and board must run in the same browser on the same computer.
