# Science Lab

An animated, touch-driven science experiment web app for two children: a 1st grader and a 5th grader. Both are native English speakers. One uses an iPad (Safari), the other an Android tablet (Chrome).

- **Live site:** https://ctogus.github.io/science-lab/ (GitHub Pages, served from `main`, repo root)
- **Repo:** https://github.com/ctogus/science-lab (public)
- **Plan and status:** `ROADMAP.md`

## Working with the owner

- The owner writes in Turkish; reply in Turkish. All text inside the app is English (US spelling).
- The owner is not a programmer. Explain in plain words, skip code details unless asked.
- Updates are requested by group number from `ROADMAP.md`, in phrases like "1'i yap", "birinci grubu yükle", "2'yi ekle", "3. güncellemeyi yap". Each means: build that group's experiments, test them, publish, tick the group in `ROADMAP.md`.
- The owner is on a usage-limited plan. Work one group at a time, publish when a group is done, and never leave the live site half-finished. After each group, report plan usage (the `get_usage` tool) and say whether there is room for the next group.
- Publishing an agreed group to `main` is pre-approved. Ask before anything else that is outward-facing (new repos, changing visibility, settings).

## Files

| File | Purpose |
|---|---|
| `index.html` | The whole app: styles, shell, voice, and every experiment |
| `sw.js` | Offline support: network first, saved copy when offline |
| `manifest.webmanifest`, `icon-*.png`, `apple-touch-icon.png` | Home-screen install |
| `ROADMAP.md` | Numbered groups of experiments and their status |

No build step and no dependencies. Keep it that way: the app must work offline from a handful of static files.

## Adding an experiment

Add one `Lab.register({...})` block in section 5 of `index.html`. The comment at the top of the script lists every field. The shell provides the menu, the six steps (Question, Predict, Experiment, Results, Why?, Quiz), narration, goals, stars and the quiz.

If you add a new file, also add it to `FILES` in `sw.js`.

### Content rules

- Every experiment has two levels, and both must be written:
  - `explorer` (age 6-7): short sentences, common words, no numbers or units, drag / tap / watch.
  - `scientist` (age 10-11): proper terms, numbers and units, at least one variable to change (slider or buttons) and a cause-and-effect goal.
- Each step explains itself: what we are doing, what happened, and why. Notes during the experiment (`ctx.note`) describe what the child just did and saw.
- Goals must be reachable by a child alone; the Next button only enables when all goals are done.
- Science must be correct at both levels. A simplification is fine; a wrong statement is not. If the big question raises a puzzle (for example "why does a ship float?"), the Why step must answer it.
- Text is read aloud sentence by sentence. Write full sentences that end in `.`, `!` or `?`. Avoid abbreviations. Units and symbols that the voice reads badly go in `Voice.clean`.
- Emoji are used as the artwork. Avoid very new emoji that old tablets cannot show.
- Home activity: something safe with household items. Add a `safety` line when there is any risk.
- Quiz: 2 questions for explorer, 3 for scientist; `answer` is the index of the correct option (options are shuffled on screen).

### Technical rules

- Scenes are SVG in an 800 × 600 view box. Use the helpers `S`, `label`, `emojiItem`, `drag`, `seg`, and `scene.loop` / `scene.after` (cleaned up automatically).
- Touch targets at least about 44 px on screen. Dragging uses pointer events through `drag()`; do not attach touch or mouse events directly.
- Must run on iPad Safari and Android Chrome: no regex lookbehind, no features newer than about 2022.
- Namespace any SVG `id` with the experiment id.
- Generated sentences (built from lists of what the child tested) must read correctly: capital letter at the start, "the" before item names, no repeated words.

## Testing

1. Serve the repo folder with `py -m http.server 8792 --directory <repo folder>` (run it in the background) and open `http://localhost:8792/index.html` in the Browser pane. `preview_start` has been unreliable here: it served a different folder.
2. The Browser pane often does not animate while hidden (`requestAnimationFrame` stalls and screenshots time out). For logic tests, replace it in the page: `window.requestAnimationFrame = f => setTimeout(() => f(performance.now()), 16)` and re-render with `App.go(step)`.
3. Drive the app from JavaScript: set a test profile in `Store.data`, then `App.open(Lab.list[i])`, `App.go(2)`. Simulate drags by dispatching `pointerdown` on the element and `pointermove` / `pointerup` on `window`. Read `.note`, `.goals li` and `App.data` to check results.
4. For every new experiment check both levels: all goals can be completed, Results and Why text read correctly, the quiz ends with a star, and the console has no errors.
5. Remove the test profile afterwards (`localStorage.removeItem('sciencelab.v1')`).
6. `.claude/test-helpers.js` (local only, not in the repo) holds ready-made helpers as `window.T`: load it with a script tag, write the group's test as a second script that fills `T.out`, start it without awaiting (JavaScript calls time out after 45 s), then read `T.out` later. If the file is missing, recreate it from this description.

A second opinion from a subagent on science accuracy and reading level is worth it for each new group.

## Publishing

- GitHub CLI: `C:\Program Files\GitHub CLI\gh.exe` (logged in as `ctogus`; it may not be on PATH).
- Commit identity is set in the repo config: `ctogus <355767971+ctogus@users.noreply.github.com>`. Keep the owner's personal email out of commits.
- Commit, then `git push origin main`. Pages redeploys in about a minute; confirm with `curl` that the live page contains the new experiment.
- Tablets get the new version the next time the app is opened online. `sw.js` is network-first, so no cache version bump is needed for content changes.

## Not verified yet

The app has only been tested in a desktop browser. Voice works on the owner's computer. On real tablets, dragging feel, voice quality and portrait / landscape layout are still unchecked; ask for feedback when the children have tried it.
