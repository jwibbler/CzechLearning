# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

Offline Czech language learning app for a single user (A1→A2 level). Five static files plus a shared stylesheet, no build step, no dependencies — opens directly in a browser via `file://` or served from GitHub Pages at `https://jwibbler.github.io/CzechLearning/`.

## Files

| File | Purpose |
|------|---------|
| `index.html` | Hub page — links to the three activities |
| `flashcards.html` | Flip-card vocabulary drill |
| `matching.html` | Click-to-match questions and answers |
| `quiz.html` | 20-question multiple-choice with weighted repetition + conjugation drill |
| `style.css` | **Shared stylesheet** — CSS variables, nav, buttons, chips, progress bar |
| `data.js` | **Single source of truth** — loaded by all three activity pages via `<script src="data.js">` |
| `Czech lesson images/` | Scanned lesson PDFs — used as source for adding new vocabulary |

## Data structures (data.js)

```js
VOCAB          // array of { czech, english, category }
MATCHING_SETS  // array of { title, pairs: [{ q, a }] }
CONJUGATIONS   // array of { verb, english, forms: [{ pronoun, form, english }] }
```

**Categories in VOCAB:** `body`, `health`, `food`, `countries`, `verbs`, `phrases`, `time`, `plurals`, `past_tense`, `cases`, `adjectives`, `aspect`, `connectors`, `negation`

Adding new content means adding to `data.js` only — the activity pages read it dynamically.

All Czech text must use correct diacritics (háčeks and čárkas): e.g. `říct`, `žít`, `šťastný`.

## Theming (style.css)

The site uses CSS custom properties for light/dark theming. Dark mode is the default. The user's preference is persisted in `localStorage` under the key `theme`.

```css
[data-theme="dark"] { --bg: #0f172a; --surface: #1e293b; ... }
```

Every page sets `data-theme="dark"` on `<html>` and runs this script block to apply the saved preference:

```js
(function() {
  const saved = localStorage.getItem('theme') || 'dark';
  document.documentElement.dataset.theme = saved;
  document.getElementById('theme-btn').textContent = saved === 'dark' ? '☀️' : '🌙';
})();
```

Each page links to `style.css` for shared styles, then has a minimal inline `<style>` for page-specific rules only.

## Key behaviour notes

- **Quiz** — 20-question sessions (`SESSION_TARGET = 20`). Weighted random selection: wrong answers add +3 to an item's weight, correct subtracts −1 (floor 1). Auto-advances 1600 ms after a correct answer; Next button only appears on wrong answers. End-of-session shows a summary of missed items.
- **Flashcards** — Filter chips by category; shuffle button; Czech→English or English→Czech toggle. No images or audio (removed — browser TTS quality was too poor for Czech).
- **Matching** — Two flex columns (`#q-col`, `#a-col`). Correct match identified by exact string comparison `ans.text === correctPair.a`, so answer strings in `MATCHING_SETS` must be verbatim identical to what appears in the answer column.

## Constraints

- Vanilla JS/HTML/CSS only — no frameworks, no npm, no bundler.
- Must work from `file://` (no server required), so no ES modules with `import`.
- All pages share `data.js` via a plain `<script>` tag; globals (`VOCAB`, `MATCHING_SETS`, `CONJUGATIONS`) are accessed directly.

## Deployment

- GitHub repo: `https://github.com/jwibbler/CzechLearning`
- Live site: `https://jwibbler.github.io/CzechLearning/` (GitHub Pages, public repo)
- After changes: `git add <files> && git commit -m "..." && git push` — Pages rebuilds automatically within ~1 minute.
- Always wait for user to confirm local testing before pushing.

## Adding new vocabulary from lesson scans

When the user shares a scanned lesson PDF or image:
1. Read the image and extract Czech words, English translations, and verb conjugations.
2. Compare against existing `VOCAB`, `MATCHING_SETS`, and `CONJUGATIONS` in `data.js`.
3. Add only entries that are genuinely missing.
4. Use correct Czech diacritics throughout.
5. Assign the most appropriate existing category — do not invent new categories.
6. Save and push only after user confirms.
