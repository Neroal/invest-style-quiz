# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

投資風格測驗 — a 10-question Traditional Chinese quiz that recommends an investment style (定存 / 基金 / ETF / 個股). The whole app is a single static file, `index.html` (HTML + CSS + one inline `<script>` IIFE, vanilla JS, no dependencies). `README.md` is the user-facing description (in Chinese). All user-visible copy is Traditional Chinese; keep that.

## Commands

There is no build, lint, or test tooling. Open `index.html` directly in a browser to run it. Deployment: pushing to `main` runs `.github/workflows/deploy.yml`, which publishes only `index.html` to GitHub Pages (repo Settings → Pages → Source must be "GitHub Actions"). Any new runtime file must be added to the "Prepare site" step.

## Architecture (the `<script>` block at the bottom of `index.html`)

- `QUESTIONS`: each has an `id`, a `dim`, and 4 options whose score is the option's index (0–3). Dimensions: `r` = risk tolerance (5 questions), `i` = involvement/effort (3), `c` = financial situation (2).
- `compute()` is the core logic:
  1. Normalizes `r` and `i` to 0–1 and splits at 0.5 into the four `TYPES` keys: `safe`, `trust`, `passive`, `active`. The `c` dimension does **not** affect the type.
  2. Starts from the type's base `alloc`, then shifts allocation toward deposit based on the *specific answers* `answers.fund` and `answers.horizon` (removing from stock → etf → fund). It reads those answer ids directly, so renaming those question ids breaks it.
  3. Builds a leverage warning when `answers.lever >= 2` combined with weak `fund`/`horizon`/`know` answers.
  4. Returns `pos` (clamped 0.06–0.94) used to place the dot on the 2×2 result map.
- `CELLS` fixes the quadrant order (top-left, top-right, bottom-left, bottom-right) used by `buildMap`; it must stay consistent with the `compute()` threshold layout (y = risk, x = involvement).
- `ASSETS` / `TYPES` hold copy and allocation percentages. Each `alloc` should sum to 100; asset colors come from CSS variables `--c-deposit/fund/etf/stock`.
- Rendering: `show()` toggles the `start` / `quiz` / `result` sections; `renderQuestion`/`pick` drive the quiz, `renderResult` draws the allocation bar, map, and notes.

When adding or removing questions, keep each dimension's max (3 per question) consistent — `compute()` derives it from `QUESTIONS` automatically — and update the question counts in `README.md`.
