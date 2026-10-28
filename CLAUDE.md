# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

A single-file IT PMO Kanban board (fictitious bank, internal demo/training tool): `index.html` contains all markup, a `<style>` block and a `<script>` block. There is no build, bundler, package manager, linter or test suite. Run it by opening `index.html` directly in a browser (`file://`) — no server needed.

## Hard constraints (keep when editing)

- Vanilla HTML/CSS/JS only; no frameworks, CDN scripts, web fonts or image files (system font stack, Unicode/inline SVG icons).
- No persistence: no `localStorage`, `sessionStorage`, IndexedDB or cookies. A refresh must reset the board to the seeded data (the UI says so).
- No `!important`; no native `alert()`/`confirm()` (delete confirmation is inline, validation errors are inline, feedback is via toasts).
- No UOB logo/branding or imitation of official UOB systems — neutral "IT PMO" text wordmark and blue palette only. (The `UOB-ITPM-####` task ID format is a deliberate spec requirement.)
- The only network call is the FormSubmit AJAX POST in `notifyNewTask()`. The notification email address lives solely in the `FORMSUBMIT_ENDPOINT` constant at the top of the script and must not be sent anywhere else. FormSubmit needs one-time email activation; the failure path must never break the board.

## Architecture

Data lives in `state = { tasks, filters: {project, assignee, priority} }`. Related UI state is in module-level `let`s, not in `state`: `nextIdNumber` (the `UOB-ITPM-####` counter), `draggedId`, `confirmDeleteId` (which card shows the inline "Delete? Yes/No"), and `focusAfterRender`.

Rendering is one-directional: mutate `state`, then call `renderBoard()`, which rebuilds the summary strip and all four columns via `innerHTML`. Card contents are only ever written in `renderCard()`/`renderBoard()`. Because rendering destroys the DOM, event handling is delegated from `#board` in `initEvents()` (drag-and-drop, the "Move ▸" select, delete buttons) rather than bound per card, and `focusAfterRender` restores keyboard focus to a card's Move control after it re-renders.

Mutations go through `addTask()`, `moveTask()`, `deleteTask()`. Filters apply through `applyFilters()` inside `renderBoard()`; the summary strip always counts the full unfiltered array. Every user-supplied string must pass through `escapeHtml()` before entering HTML (toasts use `textContent`).

Add-task flow (`handleSubmit()`) is optimistic: `validateForm()` → `addTask()` (card appears immediately) → `notifyNewTask()` in try/catch → success or warning toast. The form is a native `<dialog>` that stays open showing "Sending…" and closes in `finally`. The toast region is `popover="manual"` and is re-shown in `showToast()` so it stacks above the open modal.

While `FORMSUBMIT_ENDPOINT` still contains `YOUR_EMAIL@example.com`, `notifyNewTask()` throws before sending anything, so the "email notification failed" warning toast is the expected unconfigured behaviour. Real requests abort after 15 s.

Seed due dates are computed relative to today (`seedTasks()` via `offsetYmd()`), so the demo always has overdue and upcoming items. Date checks compare local `yyyy-mm-dd` strings (`todayYmd()`, `dateToYmd()`), never `toISOString()`, to avoid UTC off-by-one.

## Verifying changes

There are no automated tests. Open `index.html` in a browser and exercise: drag between columns, Move ▸ select, inline delete confirm, form validation errors, filters, and the warning toast on submit with the placeholder endpoint.
