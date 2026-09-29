# Family Feud Party App Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** A browser app for playing Family Feud at parties: a public scoreboard window for the TV and a host admin window with the answers, where the host marks answers correct (reveal) or wrong (strike).

**Architecture:** Static files, no build, no server, no dependencies. `game.js` holds pure state-transition functions (shared by both pages and the tests). The host page owns the state and writes it to `localStorage`; the board page renders it and re-renders on the `storage` event (fires in other windows of the same origin). Transient effects (ding / buzzer / big X) are signalled with `state.event = {type, n}` where `n` increments on every event.

**Tech Stack:** Vanilla HTML/CSS/JS (classic scripts so `file://` works), Node 26 built-in test runner (`node --test`).

**Spec:** The user goal: "create a web-app to play Family Feud for parties. The app will be run by host displaying the scoring board on the public screen. The host will have the admin view with answers, and they will click correct/wrong answer after the teams answer."

## Global Constraints

- No npm dependencies, no build step. Opening `index.html` directly or via `python3 -m http.server` must work.
- `game.js` must load both as a browser classic `<script>` (exposes `window.Feud`) and via `require('./game.js')` in Node.
- All state functions are pure: they never mutate the input state, they return a new one.
- Board shows at most 8 answer slots.

## Review Focus

1. Clicking reveal twice on the same answer → points must be banked once only.
2. Revealing leftover answers after the round was awarded → no points move (bank stays 0, scores unchanged).
3. Question text with a trailing number in the answer (e.g. `Route 66 12`) → answer text `Route 66`, points `12`.
4. More than 3 strike clicks → strikes capped at 3.
5. Malformed question text (answer line with no points, question with no answers, >8 answers) → clear error, the old question set stays in place.

---

### Task 1: Game logic (`game.js`) + tests

**Files:**
- Create: `game.js`
- Test: `game.test.js`

**Interfaces:**
- Produces (global `Feud` in the browser, `module.exports` in Node):
  - `parseQuestions(text: string) -> Array<{q: string, answers: Array<{text: string, points: number}>}>` — blocks separated by blank lines; first line is the question; each other line is `answer text <points>`; answers sorted by points descending; throws `Error` with a human-readable message on bad input.
  - `SAMPLE_QUESTIONS: string` — default question text (5+ questions).
  - `newGame(questions, teamNames = ['Team A', 'Team B']) -> State`
  - `startRound(state, round: number) -> State` — resets `revealed`, `strikes`, `bank`, `awarded`, `multiplier = 1`.
  - `reveal(state, i) -> State` — marks answer i revealed; if not already revealed and not `awarded`, `bank += points`; event `reveal`.
  - `strike(state) -> State` — `strikes = min(3, strikes + 1)`; event `strike`.
  - `clearStrikes(state) -> State`
  - `award(state, team: 0|1) -> State` — `teams[team].score += bank * multiplier`, `bank = 0`, `awarded = true`; event `award`. No-op if already awarded.
  - `revealAll(state) -> State` — reveals all without banking.
  - `setMultiplier(state, m: 1|2|3) -> State`
  - `adjustScore(state, team, delta) -> State`
  - `renameTeam(state, team, name) -> State`
  - `toggleQuestion(state) -> State` — flips `showQuestion` (whether the board shows the question text).
  - `STORAGE_KEY = 'familiada-state'`
  - State: `{teams: [{name, score}, {name, score}], questions, round, revealed: boolean[], strikes, bank, multiplier, awarded, showQuestion, event: {type: string|null, n: number}}`

- [ ] **Step 1: Write failing tests** in `game.test.js` using `node:test` + `node:assert/strict` covering: parse happy path (sorted, trailing number `Route 66 12`), parse errors (no points, no answers, 9 answers), reveal banks once, reveal after award banks nothing, strike cap at 3, award applies multiplier and resets bank, award twice is a no-op, startRound resets, functions don't mutate input, event counter increments, SAMPLE_QUESTIONS parses.
- [ ] **Step 2: Run** `node --test` → FAIL (module missing).
- [ ] **Step 3: Implement `game.js`** wrapped as `(function (root) { ...; if (typeof module !== 'undefined') module.exports = api; else root.Feud = api; })(this);`. Use `structuredClone` to copy state.
- [ ] **Step 4: Run** `node --test` → PASS.
- [ ] **Step 5: Commit** `feat: game logic and question parser`.

### Task 2: Pages — landing, host admin, public board

**Files:**
- Create: `index.html`, `host.html`, `board.html`, `style.css`, `README.md`

**Interfaces:**
- Consumes: everything from Task 1 via `<script src="game.js"></script>`.

Behaviour:
- `index.html`: title + two big links: "Host view" (`host.html`) and "Board (TV)" (`board.html`), with 3-line instructions.
- `host.html` (host's private screen):
  - On load: state = `JSON.parse(localStorage[STORAGE_KEY])` if present and valid, else `newGame(parseQuestions(SAMPLE_QUESTIONS))`. Every change: `state = fn(state, ...)`, save to localStorage, re-render.
  - Header: "Open board window" button (`window.open('board.html', 'feud-board')`), round selector (`<select>` of questions, "Round N: question"), prev/next buttons, multiplier ×1/×2/×3 buttons, "Show question on board" toggle.
  - Answers list: each row shows number, text, points, and a big button — "✓ Correct" (calls `reveal`) — disabled/marked when revealed.
  - Big red "✗ Wrong" button (`strike`), strike counter shown as X X X, "Clear strikes".
  - Bank display (`bank × multiplier`), "Award to <Team A>" / "Award to <Team B>" buttons, "Reveal remaining" (`revealAll`).
  - Teams panel: name inputs (`renameTeam`), score, −5/+5 buttons (`adjustScore`).
  - Keyboard shortcuts (ignored while typing in an input/textarea): `1`–`8` reveal, `x` strike.
  - "Questions" `<details>` section: textarea prefilled with current questions serialized back to the text format, "Load questions (resets game)" button → `parseQuestions`; on error `alert(err.message)` and keep the old game; on success `newGame(parsed, current team names)`. Also "Reset scores / new game" button with `confirm()`.
- `board.html` (TV): dark blue/gold Family Feud look, large type.
  - Top: team A name+score (left), round bank with `×M` if M > 1 (centre), team B (right).
  - Optional question text (when `showQuestion`).
  - 2-column grid of answer slots (count = number of answers, max 8): hidden slot shows the number in a circle; revealed shows text + points with a flip animation.
  - Strike overlay: on event `strike` show `strikes` big red X's for ~1.2 s and play a buzzer (WebAudio, square wave ~150 Hz, 0.6 s). On `reveal` play a ding (sine, two quick notes). On `award` a short rising arpeggio.
  - Renders on load and on `window.addEventListener('storage', ...)` for `STORAGE_KEY`; effects fire only when `event.n` differs from the last seen value (not on first load).
  - "Click for fullscreen & sound" button overlay → `document.documentElement.requestFullscreen()` + create/resume `AudioContext`; hide the button afterwards. Key `f` toggles fullscreen.
- `style.css`: shared styles; board uses CSS vars; responsive so the board fills a 16:9 TV and the host view works on a laptop.
- `README.md`: how to run (open `index.html`, or `python3 -m http.server` then http://localhost:8000), how to play, question format, `node --test`.

- [ ] **Step 1:** Write the files.
- [ ] **Step 2:** Verify in a browser: open host + board, reveal, strike, award, see board update; `node --test` still passes.
- [ ] **Step 3: Commit** `feat: host admin view and public board`.
