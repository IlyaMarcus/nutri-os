# CLAUDE.md — Project Configuration
# Project: NutriOS (Web App)
# Created: 2026-03-24 | Version: 1.0

---

## Project Overview

Personal nutrition logger for Ilya — a mobile-first single-page web app (HTML/CSS/JS) that functions as a full nutrition operating system. Features: food logging via voice input, manual text entry, or camera/photo upload; AI-powered food recognition and weight estimation from photos; automatic nutritional value lookup; daily macro/calorie tracking; and log history.

The app runs entirely in the browser (localStorage persistence) and is served as a single HTML file. No backend, no build step. Photos are processed in-memory and never stored on any server.

---

## Repository

- **GitHub repo**: TBD
- **Live URL**: TBD
- **Deploys**: automatically on every push to `main`
- **Branch strategy**: `main` = stable/released, `dev` = active work
- **Each meaningful change = one commit**, with a clear message
- **No push to `main` without explicit instruction** — work on `dev` or feature branches

---

## File Structure

```
NutriOS/
  CLAUDE.md           ← This file
  index.html          ← The app (single file, all JS/CSS inline)
  CHANGELOG.md        ← Human-readable version history
```

---

## Working Style & Rules

### Transparency (REQUIRED)
- Every action is explained **before execution** — what, why, how
- For file changes: name the file and what is changing
- When in doubt: **always ask**, never guess

### Traceability
- The top of `index.html` carries a version header comment:
  ```html
  <!-- NutriOS | Version: X.Y | Updated: YYYY-MM-DD -->
  ```
- Increment version on every meaningful change before saving

### Versioning
- Format: `v0.1` (initial), `v0.2` (minor/fix), `v1.0` (first stable release), `v2.0` (major restructure)
- Every version gets an entry in `CHANGELOG.md`
- Git commit message mirrors the changelog entry
- GitHub is the source of truth — in-file version must match the latest commit

### Changelog format (`CHANGELOG.md`)
```
## v0.2 — 2026-03-24
- Added: camera food recognition screen

## v0.1 — 2026-03-24
- Initial build: voice logging, manual entry, photo analysis, daily summary
```

### Documentation
- Explain non-obvious decisions in comments inside the code
- Define any new data structures at the point of introduction
- Goal: the code is readable cold, weeks later

### Destructive Action Protocol (REQUIRED)
Before suggesting anything irreversible — deleting data, clearing storage, removing the PWA, resetting the app, or any action that could cause data loss — Claude MUST:
1. State exactly what will be lost
2. Confirm the user understands and accepts the risk
3. Offer a safer alternative first if one exists

### Security Action Protocol (REQUIRED)
Before suggesting anything with a security risk — creating tokens, sharing credentials, pasting API keys, exposing secrets, storing sensitive data — Claude MUST triple-check:
1. Is there a safer way to do this?
2. Is the user about to expose something in an unsafe place (chat, public file, etc.)?
3. Warn explicitly before proceeding, and build the safe input mechanism BEFORE asking for the sensitive value

### Forbidden
- No silent delete or overwrite
- No commits or pushes without explicit instruction
- No over-engineering — only solve what was asked
- No adding features beyond what was requested in the conversation
- No refactoring surrounding code unless it directly blocks the task

---

## Communication

- Respond in English
- Short and direct — no filler, no preamble
- Use Markdown formatting
- When showing code changes, show only the changed section with enough context to locate it

### Screenshot / Image Rule (STRICT)
When the user sends a screenshot or image **without accompanying text instructions**, do nothing. Do not analyze it, do not write code, do not spend tokens. Just wait for the user to explain what they want.

---

## Tech Stack & Constraints

- **Single HTML file** — all CSS and JS inline, no external dependencies except Google Fonts and the Anthropic API
- **No build tools** — no npm, webpack, bundlers, etc.
- **Storage**: `localStorage` only — key: `nutri_os_v1`
- **API**: Anthropic Claude API called directly from the browser (model: `claude-opus-4-5` or latest available)
  - Used for: food photo vision analysis, weight/portion estimation, nutritional value lookup, voice transcript parsing
- **Fonts**: Google Fonts — Bebas Neue, DM Mono, DM Sans
- **Target device**: iPhone (Safari) — mobile-first, touch-friendly, no hover states required

### Input Modes
1. **Voice** — browser Web Speech API captures speech → transcript sent to Claude API → parsed into food + quantity → nutrition lookup
2. **Manual text** — user types e.g. "2 eggs, 1 slice bread" → sent to Claude API → nutrition lookup
3. **Camera / Upload** — user takes photo or uploads image → image converted to base64 in-memory → sent to Claude vision API → food identified, weight/grams estimated, nutrition returned
   - Photos are **never persisted** — processed in memory only, base64 blob discarded after API response

### Nutrition Data
- All nutritional values (calories, protein, carbs, fat, fiber) are derived via Claude API — no separate nutrition database needed
- Claude returns structured JSON with per-item and total breakdown

### CSS conventions
- CSS variables defined in `:root` — use them, never hardcode colors
- All spacing via existing utility classes before adding new ones
- Dark theme only — `--black: #0a0a0a` base

### JS conventions
- State lives in the single `S` object — never create parallel state
- Persist via `save()` / `load()` functions only
- No external libraries — vanilla JS only
- Keep all render functions idempotent (safe to call multiple times)

---

## Data Structure

```js
// localStorage key: nutri_os_v1
S = {
  apiKey: "",           // user-entered Anthropic API key
  log: [                // array of daily logs
    {
      date: "2026-03-24",
      entries: [
        {
          id: "uuid",
          time: "08:30",
          input_type: "voice" | "manual" | "camera",
          raw_input: "two scrambled eggs",   // original text or voice transcript
          items: [
            {
              name: "Scrambled eggs (2 large)",
              grams: 94,
              calories: 182,
              protein: 13,
              carbs: 2,
              fat: 14,
              fiber: 0
            }
          ],
          totals: { calories: 182, protein: 13, carbs: 2, fat: 14, fiber: 0 }
        }
      ],
      day_totals: { calories: 0, protein: 0, carbs: 0, fat: 0, fiber: 0 }
    }
  ]
}
```

---

## Integration Plan

NutriOS is built as a **standalone app first**, but is explicitly designed to be merged into **TrainingOS** at a later stage.

### Rules that follow from this

- **All nutrition logic lives in clearly named, self-contained functions** — prefixed `nutri_` so they never clash with TrainingOS function names
- **State is isolated** — the `S` object and `nutri_os_v1` localStorage key are independent; when merging, the nutrition state will fold into TrainingOS's state object as a `nutrition` namespace
- **No assumptions about the shell** — NutriOS renders into a `#app` div; when integrated, that becomes a screen inside TrainingOS's nav system
- **CSS variables match TrainingOS** — same dark theme, same fonts, same spacing conventions — so the UI drops in without visual conflicts
- **No feature coupling** — NutriOS never references TrainingOS concepts (sets, sessions, programs). It is nutrition-only.

---

## Known Limitations / Deferred

| Item | Status |
|---|---|
| API key exposed in client-side JS | Accepted for personal-use app — not for public deployment |
| No offline support (API calls need network) | Deferred |
| Photo weight estimation is AI-estimated, not scale-accurate | By design — user can manually correct |
| No barcode scanning | Deferred |

---

## Changelog

| Date | Version | Change |
|---|---|---|
| 2026-03-24 | 1.0 | CLAUDE.md created for NutriOS project |
