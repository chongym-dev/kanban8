# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

A single-file IT PMO Kanban demo/training app for a fictitious bank: `index.html` holds all markup, a `<style>` block and a `<script>` block. There is no build, lint, or test tooling and no package manager. Run it by opening `index.html` directly in a browser (`file://`, no server).

## Hard constraints (from the original spec)

- Vanilla HTML/CSS/JS only: no frameworks, bundler, npm, CDN scripts, web fonts, or image files (system font stack, inline SVG/Unicode glyphs).
- No persistence of any kind: no localStorage, sessionStorage, IndexedDB, or cookies. A refresh resets the board to the seeded demo data, by design (the header says so).
- The only network call is the FormSubmit AJAX endpoint (`FORMSUBMIT_ENDPOINT`). A failure there must never break the board; the card stays and a warning toast shows.
- No real UOB logo or branding. Use the neutral "IT PMO" text wordmark and the blue palette. The `UOB-ITPM-` task ID prefix and `[UOB IT PMO]` email subject tag are required by the spec and live in config constants.
- CSS: custom properties for palette and spacing, no `!important`. Escape all user strings with `escapeHtml()` before building HTML.

## Architecture

- `state = { tasks, filters, confirmingId, movingId }` is the single source of truth. UI-only transient state (which card shows "Delete? Yes/No" or the "Move ▸" menu) lives in `state` too, so it survives re-renders.
- `renderBoard()` rebuilds the whole board (columns, cards via `renderCard()`, count badges, header summary) from `state` using `innerHTML`. Do not mutate card DOM elsewhere. To keep keyboard focus across a re-render, set `pendingFocus` to a selector before calling `renderBoard()`.
- Because the board is rebuilt on every change, all board events are delegated on `#board`: `click` uses `data-action` attributes (`delete`, `delete-yes`, `delete-no`, `move`, `move-to`), and drag and drop uses `dragstart`/`dragover`/`dragleave`/`drop`/`dragend`. Both drag and the keyboard "Move ▸" menu end in `moveTask()`.
- Mutations: `addTask`, `moveTask`, `deleteTask`. `applyFilters()` is a pure filter over `state.tasks` used only at render time.
- Add Task flow (`handleSubmit`): `validateForm` → optimistic `addTask` and `resetForm` → `notifyNewTask` (async, try/catch/finally) → success or warning toast. Task IDs come from `generateTaskId()` using the module-level `nextIdNumber` counter, which the seed data also consumes (the first user-added task is `UOB-ITPM-0009`).
- Seed due dates are computed relative to today with `offsetDate()`, so the demo always has overdue items.

## Testing changes

There are no automated tests. Verify in a browser: open the file, then check drag between columns, the Move ▸ menu, delete confirm, filters, form validation, the toast on the FormSubmit failure path (the placeholder email makes it fail), and that columns stack below 768px width.
