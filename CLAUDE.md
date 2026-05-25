# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

Offline Czech language learning app for a single user (A1→A2 level). Five static files, no build step, no dependencies, opens directly in a browser via `file://`.

## Files

| File | Purpose |
|------|---------|
| `index.html` | Hub page — links to the three activities |
| `flashcards.html` | Flip-card vocabulary drill |
| `matching.html` | Click-to-match questions and answers |
| `quiz.html` | 20-question multiple-choice with weighted repetition |
| `data.js` | **Single source of truth** — loaded by all three activity pages via `<script src="data.js">` |
| `RecentCzechLessons.pdf` | Scanned handwritten lesson notes — reference for content |

## Data structures (data.js)

```js
VOCAB          // array of { czech, english, category }
MATCHING_SETS  // array of { title, pairs: [{ q, a }] }
CONJUGATIONS   // array of { verb, english, forms: [{ pronoun, form, english }] }
```

**Categories in VOCAB:** `body`, `health`, `food`, `countries`, `verbs`, `phrases`, `time`, `plurals`, `past_tense`, `cases`, `adjectives`, `aspect`, `connectors`, `negation`

Adding new content means adding to `data.js` only — the activity pages read it dynamically.

## Key behaviour notes

- **Quiz** — 20-question sessions (`SESSION_TARGET = 20`). Weighted random selection: wrong answers add +3 to an item's weight, correct subtracts −1 (floor 1). Auto-advances 1600 ms after a correct answer; Next button only appears on wrong answers. End-of-session shows a summary of missed items.
- **Flashcards** — `EMOJI_MAP` (~150 entries) maps specific Czech words to emoji; `CAT_EMOJI` provides per-category fallbacks. `CAT_BG` gives each category a distinct background colour on the visual box. No network calls.
- **Matching** — two flex columns (`#q-col`, `#a-col`). Correct match identified by exact string comparison `ans.text === correctPair.a`, so answer strings in `MATCHING_SETS` must be verbatim identical to what appears in the answer column.

## Constraints

- Vanilla JS/HTML/CSS only — no frameworks, no npm, no bundler.
- Must work from `file://` (no server required), so no ES modules with `import`.
- All pages share `data.js` via a plain `<script>` tag; globals (`VOCAB`, `MATCHING_SETS`, `CONJUGATIONS`) are accessed directly.
