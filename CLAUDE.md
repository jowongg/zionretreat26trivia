# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running

No build, no dependencies, no server. Open `trivia_game.html` directly in a browser. The repo is also served as a static site via GitHub Pages at `https://jowongg.github.io/zionretreat26trivia/trivia_game.html` (Pages is configured to deploy from the `main` branch root).

## Architecture

This is a single-file static web app. All HTML, CSS, JavaScript, and question data live in `trivia_game.html`. There is no framework, no bundler, and no external runtime asset.

**Two sources of truth for the questions — keep them in sync.** Questions exist twice:
- `trivia_50_final.md` — human-readable source with all 50 questions, options, correct answers, and explanations (Cantonese Chinese).
- The `QUESTIONS` array literal inside `<script>` in `trivia_game.html` — what the game actually reads at runtime.

When a question changes, both files must be updated. The markdown does not feed the HTML at runtime; nothing parses it.

**Three-page state machine.** The app has three `<div class="page">` sections — `page-main`, `page-question`, `page-result` — and `switchPage(id)` toggles a single `.active` class to show one at a time. Flow: main → question → result. On a correct answer the result page reveals the right option + explanation; on a wrong answer it does not (the user goes home to retry).

**Question shape.** Each `QUESTIONS` entry is `{id, cat, q, a:[4 strings], correct: 0–3, exp}`. `cat` is matched as a string against `CATEGORIES[i].key` to group questions on the main page — these strings must stay byte-identical (e.g. `"地理 / 雜項"` with the exact spacing).

**Difficulty is hardcoded, not stored on the question.** Two `Set`s near the top of the script — `EASY` and `MEDIUM` — list question IDs; everything else falls through to `hard` in `getDifficulty()`. To rebalance difficulty, edit these sets.

**Progress is persisted to `localStorage`** under key `trivia_progress` as `{questionId: "correct" | "wrong"}`. `resetProgress()` clears it after a `confirm()`. Only "answered at all" (not correctness) drives the ✓ mark on the main grid; the correct/total stats use the value.

**Question IDs are 1–50 sequential and assumed contiguous.** `nextQuestion()` wraps from 50 back to 1. If you add or remove questions, renumber so IDs stay 1..N contiguous and update `nextQuestion()`'s upper bound.
