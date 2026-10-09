# SevenX Icebreaker

A live, anonymous voting game about what SevenX stands for. One screen on the projector, everyone votes from their phone.

- **Game screen (projector):** https://friesen6.github.io/sevenx-icebreaker/
- **Voting page (what the QR code opens):** https://friesen6.github.io/sevenx-icebreaker/play.html
- **Repo (published pages only):** https://github.com/friesen6/sevenx-icebreaker

## Running a session

1. Open the game screen on the projector laptop. The opening screen shows a QR code. Click the QR code to enlarge it, click again to shrink it.
2. Click **Host controls** (bottom left) and enter the passcode, which is in the local `.pass` file. The browser remembers it.
3. Press **Start**. Every question starts its timer on its own. Votes are anonymous and final. The result appears when the timer ends or when you press Reveal.
4. Before the real session, run through it once, then press **Reset game** (two clicks) to clear test votes.

| Key | Action |
|---|---|
| Space | Pause or resume the timer on an open question. On other slides, next. |
| → (or N, Enter) | Reveal the answer on an open question, then next |
| ← | Back |
| S | Skip the open question |
| P | Pause or resume (same as Space) |
| Esc | Close the enlarged QR code |

Buttons: Back, Next or Reveal now, Skip question, Pause timer, Restart question, Reset game.

What people see:
- **Big screen:** the question, the options, how many votes are in (never the split while voting), a timer. At the reveal it shows the correct option and the percentage per option.
- **Phones:** four letter buttons (A to D, or A and B in round 3). After the reveal: right or wrong, the answer text and the explanation. At the end: every question with its answer and the person's own choice, in small text.
- Round 1 and 2 are scored (a private score on each phone). Round 3 has no score and shows the split.

## How it is built

- Two static pages built from one source file: `template.html` becomes `index.html` (host) and `play.html` (phone). They are hosted on GitHub Pages from the `main` branch.
- The game state lives in Supabase, project `sxoqtrphoyjmgsyftbfj` ("Get it done"), in a separate `icebreaker` schema that the API does not expose. The pages call `public.ib_*` functions with the public anon key. Everything that must stay secret (the answer key, the host passcode) stays in the database. The page source contains no answers.
- Each phone is identified by a random id stored in its browser. There are no names or accounts.
- The pages poll the server once a second.

## Files

| File | What it is | In the public repo? |
|---|---|---|
| `template.html` | Source for both pages (HTML, CSS, JS) | no |
| `deck.json` | Rounds, questions, options, correct answers, explanations, timers | no (contains the answers) |
| `build.js` | `node build.js` builds `index.html`, `play.html` and `seed.sql` | no |
| `qr.min.js` | QR code library (qrcode-generator 1.4.4, MIT), inlined into the pages | no |
| `schema.sql` | Database tables, view and functions | no |
| `seed.sql` | Generated: loads the answer key and passcode into the database | no |
| `.pass` | The host passcode | no |
| `index.html`, `play.html` | Generated pages that GitHub Pages serves | yes |
| `archive/sevenx-game.html` | The first version: a host-only game, no phones | no |

## Common tasks

**Change a question, option, answer or timer**
1. Edit `deck.json`.
2. `node build.js` (rebuilds the pages and `seed.sql`).
3. Run `seed.sql` in the Supabase SQL editor (project `sxoqtrphoyjmgsyftbfj`).
4. Publish the pages: `git add index.html play.html && git commit -m "..." && git push`. GitHub Pages updates in about a minute. Refresh the page if you see the old version.

**Change the host passcode**
```sql
update icebreaker.config set value = 'NEW-CODE' where key = 'pass';
```
Then put the same value in `.pass`. Browsers that saved the old code ask again.

**Set up the database from scratch** (new Supabase project): run `schema.sql`, then `seed.sql`. Put the project URL and publishable key at the top of `template.html` (`SB_URL`, `SB_KEY`), then rebuild.

**Take it all down after the event**: see the bottom of `schema.sql` for the drop statements, and delete the GitHub repo or turn off Pages.

## Things to know

- Votes are limited to one per device per question, and the first vote is final. Clearing browser data lets someone vote again.
- The vote split is hidden at the server until the reveal, and the answer recap is held back until the host reaches the last slide.
- 20 phones need Wi-Fi or mobile data.
- The source files are intentionally not in the public repo. This folder is the only copy, so keep it backed up.
