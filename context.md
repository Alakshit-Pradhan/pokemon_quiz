# POKÉMON QUIZ — Project Context

> This file is the **source of truth** for the Pokémon Quiz project. When starting a new AI chat to add features or fix bugs, paste this file at the top of the conversation so the assistant immediately understands the project.

---

## Developer

- **Name:** Alakshit Pradhan
- **Age:** 18, independent developer from India
- **Credit line (appears on home + result screens):** `Made by ALAKSHIT PRADHAN`

---

## Project Links

- **Live site:** https://alakshit-pradhan.github.io/pokemon-quiz/ _(update if repo name differs)_
- **GitHub repo:** https://github.com/Alakshit-Pradhan/pokemon-quiz _(update if repo name differs)_
- **Main file:** `index.html` (single-file, self-contained — all HTML, CSS, JS inline)
- **No external assets in repo:** Pokémon images load from a third-party CDN

---

## Project Overview

**POKÉMON QUIZ** is a "Who's That Pokémon?" style guessing game. A Pokémon artwork appears, the player picks the correct name from 4 options within 8 seconds.

**Core gameplay:**
- 3 difficulty modes: **Easy** (Kanto classics + iconic legendaries), **Medium** (Gen 2–4 starter lines, regional favourites, cross-gen popular mons), **Hard** (obscure deep-cuts from every generation)
- Each round = **50 questions**, unique per round (Pokémon shuffled, no duplicates within a round)
- Each question = **1 Pokémon image + 4 name options** (1 correct + 3 random decoys drawn from the same difficulty tier)
- **8-second timer** per question; if it expires, the correct answer is revealed and we auto-advance
- Answer feedback: green + scale-pop for correct, red + shake for wrong
- End screen: final score, percentage, and letter grade (S / A / B / C / D)
- Keyboard shortcuts on desktop: `1`–`4` select options

---

## Tech Stack

- **Single-file HTML** — all HTML, CSS, and JS inline in one `index.html`
- **Vanilla JavaScript** — no frameworks, no build step, no npm
- **Google Fonts:** Fredoka (bubbly title/buttons), Press Start 2P (retro labels/credit), Rubik (body)
- **Pokémon sprites** from PokeAPI's sprite CDN (GitHub-hosted, stable since 2017, no rate limits):
  `https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/other/official-artwork/{id}.png`
  Fallback if official-artwork missing: `.../sprites/pokemon/{id}.png` (regular pixel sprite)
- **No localStorage / no backend / no cookies** — the game is purely stateless per session
- **Deployment:** GitHub Pages, free hosting, auto-deploys on commit to `main` branch

---

## Pokémon Database (hand-tagged)

All Pokémon data is stored inline in `index.html` inside a single `POKEMON` object. Each entry is a `[nationalDexId, displayName]` tuple. IDs match the PokeAPI / National Dex numbering (Bulbasaur = 1, Pikachu = 25, Mewtwo = 150, etc.).

| Mode   | Count | Contents                                                                 |
|--------|-------|--------------------------------------------------------------------------|
| easy   | ~69   | Gen 1 starter families, Pikachu/Raichu, Eevee's 4 Gen-1 evos, legendary birds, Dragonite, Lucario — pop-culture Pokémon even non-fans recognise |
| medium | ~70   | Gen 2–4 starter families, Espeon/Umbreon/Leafeon/Glaceon/Sylveon, Tyranitar, Suicune/Entei/Raikou, Rayquaza, Garchomp, Greninja, Mimikyu |
| hard   | ~71   | Deep cuts — Dunsparce, Shuckle, Luvdisc, Stunfisk, Audino, Reuniclus, Palossand, Togedemaru, Copperajah, etc. |

**Constraint:** Each mode must have **≥53 entries** (50 correct + 3 decoys for the last question). A load-time `sanityCheck()` logs a `console.warn` if any mode is under-populated.

**Adding/retagging Pokémon:** Just edit the `POKEMON` object's arrays. Format is strict: `[id, 'Name']`. Escape apostrophes (`'Farfetch\'d'`). Special characters like `♀`, `♂`, `é` work fine in JS string literals.

---

## Timer System

- **Duration:** 8 seconds per question (constant `TIMER_DURATION`, line ~811 in index.html)
- **Starts** when the Pokémon image actually loads (fair for slow connections — no ticking against a blank image)
- **Visual phases** use fractional thresholds (auto-scale if duration changes):
  - **Safe (green):** `TIMER_DURATION * 0.6`s → full (first 40% of time)
  - **Warn (amber):** between `0.3 × DURATION` and `0.6 × DURATION` remaining (middle 30%)
  - **Danger (red, pulsing):** `0 < remaining ≤ 0.3 × DURATION` (last 30%)
- **Timeout behaviour:** `state.locked = true`, all buttons disabled, correct answer revealed in green, 1.3s delay then auto-advance
- **Early selection stops the timer:** `selectOption()` calls `stopTimer()` first thing
- **Timer is cleared on** quit/finish/next-question transitions

To change the timer duration, edit `TIMER_DURATION` only. Threshold fractions (`TIMER_WARN_FRAC`, `TIMER_DANGER_FRAC`) preserve the same visual pacing automatically.

---

## Game State Structure

```js
const state = {
  mode: null,           // 'easy' | 'medium' | 'hard'
  pool: [],             // shuffled 50 [id,name] for the round
  allForMode: [],       // full mode list — used for decoy selection
  current: 0,           // 0-indexed question number
  score: 0,
  locked: false,        // true during feedback delay, prevents double-click
  correctIdx: -1,       // 0-3, which option button holds the correct name
  timerHandle: null,    // setInterval handle, null when not running
  timerStart: 0,        // performance.now() at timer start
  qToken: 0             // monotonic guard against stale async image loads
};
```

The `qToken` guard is important: each `nextQuestion()` bumps it. Any `new Image()` `onload`/`onerror` callback checks `myToken === state.qToken` before touching state. Without this, a slow Gen-1 image arriving AFTER the user has already answered and moved on would restart the timer for the wrong question.

---

## Visual Identity

- **Background:** radial gradients (soft yellow + blue tints on white/pink base) + subtle 5%-opacity Pokéball-pattern overlay
- **Animated Pokéball** as home-screen logo (wobbles slightly on CSS keyframes)
- **Colours (CSS custom properties):**
  - `--poke-red` (#EE1515), `--poke-yellow` (#FFCB05), `--poke-blue` (#3B4CCA)
  - `--poke-white` (#FFFFFF), `--poke-black` (#1F1F1F)
  - Mode accent colours: `--easy` (#2ECC40 green), `--medium` (#F39C12 orange), `--hard` (#8E44AD purple)
  - `--ok` (#2ECC40 green, correct), `--bad` (#E74C3C red, wrong)
- **Typography:**
  - `Fredoka` (weight 500–700) — bubbly headings, mode names, option buttons, title
  - `Press Start 2P` — retro pixel accents (credit, section labels, "WHO'S THAT POKÉMON?")
  - `Rubik` — clean body text, HUD pills
- **Button style:** white card with 3–4px black border, chunky drop shadow `0 4px 0 rgba(0,0,0,0.15), 0 8px 20px rgba(0,0,0,0.12)` — "tactile/tappable" feel, moves on hover/active

---

## Navigation Flow

```
Home Screen (3 mode cards)
    ↓  tap Easy / Medium / Hard
Quiz Screen (Q 1/50 → Q 50/50)
    ├─ Tap correct option → +1 score, 700ms delay, next
    ├─ Tap wrong option   → reveal correct, 1100ms delay, next
    ├─ Timer expires      → reveal correct, 1300ms delay, next
    └─ Quit button        → confirm, back to Home
Result Screen (grade + score + %)
    ├─ Play Again  → new round, same mode
    └─ Home        → back to Home Screen
```

---

## Key Design Decisions & Gotchas

1. **Single file, zero build step** — developer edits `index.html` via GitHub's web UI pencil-icon workflow. No local git, no npm.
2. **Images from CDN, not repo** — keeps the repo under 50 KB. Downside: requires internet. Fine for a GitHub-Pages-hosted quiz.
3. **Image preloading** — the next question's image is preloaded in the background (`new Image().src = imgURL(nextId)`) so transitions feel instant.
4. **Stale async guard** — `qToken` pattern prevents callbacks from old questions touching the new question's state.
5. **Timer starts only after image loads** — this is a fairness choice. On slow connections we don't want the clock running against a blank screen.
6. **Decoys are same-tier** — correct Pokémon's 3 decoys come from the **same difficulty list** so Hard mode doesn't get "Pikachu" as a decoy against "Stunfisk". Makes the quiz feel more challenging at each tier.
7. **50 unique Pokémon per round** — `shuffleInPlace(pool).slice(0, 50)` guarantees no duplicate questions within a single round.
8. **Grading thresholds:** S ≥ 90%, A ≥ 75%, B ≥ 60%, C ≥ 40%, D < 40%.
9. **Mobile breakpoints:** 600px (single-column mode cards + options), 360px (tighter title/button sizing).
10. **No localStorage** — intentionally stateless. Each session starts fresh. Could add persistent high scores later if wanted.

---

## Communication Preferences (for future AI assistants)

- Reply in **English + Hindi mix** (Hinglish) — casual, direct, beginner-friendly tone
- Deliver **complete paste-ready files** — no truncation, no "rest of the code here" placeholders
- Prefer **targeted edits** when modifying existing code — preserve all working features, visual identity, and the Pokémon database exactly
- Use **analogies and step-by-step explanations** for technical concepts — the developer is learning
- **Production-quality code** even when teaching — don't write bad code for simplicity's sake
- When deploying changes: instructions should cover GitHub web UI (edit via pencil icon) since no local Git setup is assumed
- **Never silently remove or change** existing features when adding new ones — confirm if uncertain
- Verify fixes with actual testing where possible — don't ship speculative edits

---

## Deployment Workflow (no local setup)

1. Open `github.com/Alakshit-Pradhan/pokemon-quiz` in browser
2. Click `index.html` → ✏️ pencil (edit)
3. `Ctrl+A` → delete → paste new full file
4. Scroll down → **Commit changes**
5. Wait 1–2 minutes for GitHub Pages to redeploy
6. Hard refresh live site (`Ctrl+Shift+R`) or open in incognito/private tab to bypass browser cache

---

## Future Feature Ideas (roadmap — not yet built)

- [ ] **Silhouette mode** — classic "Who's That Pokémon?" black silhouette that reveals full colour on answer
- [ ] **Time-attack mode** — 60 seconds total, answer as many as you can
- [ ] **Type guessing** — show a Pokémon, guess its type(s) instead of its name
- [ ] **Sound on** — play the Pokémon's cry (PokeAPI has cry URLs) on reveal
- [ ] **Persistent high scores** per mode (localStorage)
- [ ] **Shareable result cards** — "I got 42/50 on Hard mode! Can you beat me?" with a screenshot
- [ ] **More Pokémon** — Gen 9 / Paldea additions (Koraidon, Miraidon, Iron Valiant, etc.)
- [ ] **Daily challenge** — fixed seed of the day, share scores
- [ ] **Streak counter** — consecutive correct answers visible during gameplay
- [ ] **Difficulty mixer** — custom round with % split from each tier
- [ ] **Multi-language names** — Japanese, French, German Pokémon names option

When adding any of these, **read the existing code carefully first** and preserve the established visual identity and gameplay feel.

---

## Credits

- **Code, design, direction:** Alakshit Pradhan
- **Assistance:** Iterative collaboration with Claude (Anthropic)
- **Pokémon artwork source:** [PokéAPI Sprites](https://github.com/PokeAPI/sprites) (community-maintained, freely available)
- **Fonts:** Google Fonts (Fredoka, Press Start 2P, Rubik)
- **Legal note:** Pokémon and all related names, artwork, and trademarks are property of Nintendo / Game Freak / The Pokémon Company. This is a non-commercial fan project for educational/personal use.

---

*Last updated: when you change the project, update this file too.*
