# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single-file "IT PMO" Kanban board (`index.html`) for a fictitious bank's internal IT PMO. It is a demo/training tool. Keep the neutral "IT PMO" text wordmark; do not add a real bank logo or imitate an official system.

There is no build, lint, test or package manager. Open `index.html` directly in a browser (double-click, `file://`). No server is needed. To syntax-check the script, copy the contents of the `<script>` block into a scratch `.js` file and run `node --check` on it.

## Hard constraints (from the original spec)

- Vanilla HTML/CSS/JS only, all in the one `index.html`. No frameworks, build step, npm, or external resources (system font stack, inline SVG or Unicode glyphs only).
- No persistence: never use localStorage, sessionStorage, IndexedDB or cookies. Board state is an in-memory array that is reseeded on refresh, and the header note says so.
- The only network call is the FormSubmit AJAX JSON endpoint (`FORMSUBMIT_ENDPOINT`, a config constant at the top of the script, placeholder `YOUR_EMAIL@example.com`). The email address must never be sent anywhere else. FormSubmit needs one-time activation via the confirmation email sent on the first submission.
- CSS uses custom properties in `:root` and no `!important`. No `alert()` or `confirm()`: errors are inline, delete confirmation is an inline Yes/No.

## Architecture

All script lives in one IIFE at the bottom of the file, with commented sections: constants and state, helpers, render, board mutations, toast, FormSubmit, add-task form, filters and drag-and-drop, init.

- **State:** a single `state` object (`tasks`, `filters`, `nextSeq`, `confirmDeleteId`). `SEED` holds the 8 sample tasks, and `seedTasks()` copies them in on load. Seed due dates are fixed calendar dates (not relative to today), so the Overdue badges depend on when the file is opened.
- **Rendering:** `renderBoard()` is the only writer of card DOM. It re-renders every column from `applyFilters()` output via `innerHTML` and updates the count badges, header summary and filter note. `renderCard()` builds one card's HTML string, and everything interpolated goes through `escapeHtml()`. Any state change (`addTask`, `moveTask`, `deleteTask`, filter change) mutates `state` and calls `renderBoard()`. Because the DOM is rebuilt, handlers that need focus afterward re-query and focus it explicitly (see `askDelete`, `cancelDelete`, `deleteTask`, and the move branch of `onBoardClick`).
- **Events:** one delegated click handler on `#board` (`onBoardClick`, dispatching on `data-action`), plus delegated drag handlers (`dragstart/over/leave/drop/end`) on `#board`. Cards are `draggable`, and `onPointerDown`/`onPointerUp` turn dragging off while the pointer is on a button or the Move `<summary>`. The keyboard alternative to drag is the `<details class="move">` "Move ▸" menu on each card.
- **Add-task modal:** a `div#modal` with `role="dialog"` (not a native `<dialog>`), with a hand-written focus trap and Escape handling in `onDocumentKeyDown`. Field ids are `field-<name>` with matching `error-<name>` elements, and `FORM_FIELDS` ties validation, error display and focus together. `validateForm` includes the "due date not in the past" rule.
- **Submit flow (`onTaskSubmit`):** validate, then `addTask` (optimistic: the card appears at once), `resetForm`, and `notifyNewTask` (FormSubmit) with the submit button disabled and showing "Sending…". On failure, show the warning toast "Card added locally — email notification failed". A `finally` block re-enables the buttons and closes the modal. A network failure must never break the board.
- **IDs:** `UOB-ITPM-####`, generated from `state.nextSeq` (starts at 9 after the 8 seeds).

When adding a status, project or category, update the matching constants (`COLUMNS`, `PROJECTS`, `CATEGORIES`), the `<option>` lists in both the filter bar and the form, and (for statuses) the column markup, `SUMMARY_IDS` and `COLUMN_HEADINGS`.
