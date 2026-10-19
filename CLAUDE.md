# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single-file IT PMO Kanban board (`index.html`) for a fictitious bank's internal PMO. It's a demo and training tool. The page must not use real bank logos or branding, or imitate an official system. The only identity is the neutral "Throughline" name, its inline-SVG logo (three Kanban lanes plus an amber arrow) and a corporate blue palette.

## Hard constraints (keep these when editing)

- **One file:** all markup, one `<style>` block and one `<script>` block live in `index.html`. Vanilla HTML/CSS/JS only: no frameworks, build step, npm, CDN scripts, web fonts or image files. Icons are Unicode or inline SVG; fonts are a system stack.
- **Opens by double-click** (`file://`), so no server is needed.
- **No persistence.** No localStorage, sessionStorage, IndexedDB or cookies. A refresh resets the board to seed data; the header note says so.
- **One network call:** `notifyNewTask()` posts JSON to FormSubmit's AJAX endpoint. Nothing else leaves the page.
- No `alert()`/`confirm()` (delete uses an inline "Delete? Yes / No" toggle), and no `!important`.

## Commands

There's no build or test tooling. Open `index.html` in a browser. Quick checks:

```bash
sed -n '/<script>/,/<\/script>/p' index.html | sed '1d;$d' > /tmp/k.js && node --check /tmp/k.js
```

```bash
grep -nE 'localStorage|sessionStorage|indexedDB|document\.cookie|alert\(|confirm\(|!important|https?://' index.html
```

The grep should only match the `FORMSUBMIT_ENDPOINT` line.

## Architecture

- **Config** is at the top of the script: `FORMSUBMIT_ENDPOINT` (the notification email address, changed in one place), `APP_NAME` / `APP_VERSION`, `TASK_ID_PREFIX` (`ITPM-`, zero-padded to 4 digits from the `nextId` counter) and `TOAST_MS`. The HTML comment above the script explains FormSubmit's one-time activation: the first submission sends a confirmation email, and nothing is delivered until its link is clicked.
- **State:** `state = { tasks, filters, theme }` is the only source of truth (`theme` is `null` to follow the OS). `STATUSES`, `PROJECTS`, `CATEGORIES` and `PRIORITIES` drive both the select options and the board columns. Adding a column means adding it to `STATUSES`.
- **Render loop:** mutations (`addTask`, `moveTask`, `deleteTask`, `setPendingDelete`) change `state` and then call `renderBoard()`. `renderBoard()` runs `applyFilters()` → `renderCard()` and rebuilds the board's `innerHTML`, then calls `renderSummary()`. Never edit card DOM directly; even the delete-confirm UI is state (`task.pendingDelete`). Column badges count filtered tasks; the header summary counts all tasks.
- **Escaping:** every interpolated value goes through `escapeHtml()`. Keep that when adding fields.
- **Events** use delegation on `#board`, since cards are re-rendered: click (`data-action` = delete / confirm-delete / cancel-delete), change (the per-card "Move ▸" select, the keyboard fallback for drag), and native HTML5 drag events (`.drop-target` highlights the hovered column). After a re-render, focus is restored with `focusInCard()`.
- **Form submit is optimistic:** `validateForm()` → inline errors with `aria-invalid` → `addTask()` → reset → success toast, then `notifyNewTask()` in try/catch. While it's in flight the button is disabled and shows "Sending…". On failure (including FormSubmit returning 200 with `success: "false"`) the card stays and a warning toast appears.
- **Dates** are compared as local `YYYY-MM-DD` strings (`toIsoDate`, `todayIso`). Seed due dates are relative to today (`daysFromToday`), so some seed tasks are always overdue. Overdue = due date before today and status ≠ Done.
- **Layout:** the main area (filters + board) takes the full width; the board is 4-up, 2-up below 1024px and 1-up below 768px. The Add Task form lives in a right-hand drawer, a modal `<dialog>` (`#task-drawer`) opened by `openTaskDrawer()` from "+ Add Task" and closed by `closeTaskDrawer()` after a successful add, the × button, Esc or a backdrop click. Colours and spacing are tokens on `:root`; card left borders and priority pills are keyed by `data-priority` / `.pill-<Priority>`.
- **Theming:** every colour token uses `light-dark()`, driven by `color-scheme`. `applyTheme()` sets `data-theme` on `<html>` only after the user toggles; a refresh goes back to the OS setting. Add new colours as `light-dark(light, dark)` tokens, never raw hex in rules.
- **Help / FAQ / About** is one native `<dialog>` (`#info-dialog`) with an ARIA tablist; `openInfo(tab)` opens it, `?` opens Help, and a backdrop click or Esc closes it. Inline SVGs omit `xmlns` so the URL grep stays clean.
