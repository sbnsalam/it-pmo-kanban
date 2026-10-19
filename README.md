# IT PMO Kanban Board

A single-file Kanban board for a fictitious bank's IT Project Management Office. It's built for demos and training: log a task, move it across the board, and get an email notification when a new task is added.

Everything lives in `index.html`. There's no build step, no dependencies and no server.

> This is a demo. It doesn't use any real bank's branding and isn't connected to any real system.

## Getting started

Double-click `index.html`, or open it in any modern browser. It works straight from `file://`.

The board starts with seed tasks. Their due dates are set relative to today, so a few are always overdue.

**Nothing is saved.** Refreshing the page resets the board to the seed data.

## Features

- **Four columns:** Backlog, In Progress, Blocked, Done. Each column header shows a count of its visible tasks.
- **Add Task form** with inline validation. Title, Assignee and Due Date are required. You can also set Description, Project / Workstream, Category, Priority and Status.
- **Task IDs** are generated as `UOB-ITPM-0001`, `UOB-ITPM-0002`, and so on.
- **Moving tasks:** drag a card to another column, or use the card's **Move ▸** dropdown (keyboard friendly).
- **Delete** with an inline "Delete? Yes / No" confirmation.
- **Filters** by project, assignee (text match) and priority.
- **Overdue highlighting:** a task is overdue when its due date has passed and it isn't Done.
- **Header summary** of totals across all tasks, ignoring filters.
- **Responsive:** the layout stacks into one column below 768px.
- **Email notification** for each new task, sent through [FormSubmit](https://formsubmit.co).

### Reference data

| Field | Values |
|---|---|
| Project / Workstream | Core Banking Upgrade, Digital Channels, Cybersecurity Uplift, Data & Analytics, Infrastructure & Cloud, Vendor Management |
| Category | Application Development, Infrastructure, Cybersecurity, Data, Compliance, Vendor |
| Priority | Critical, High, Medium, Low |

These lists are the `PROJECTS`, `CATEGORIES`, `PRIORITIES` and `STATUSES` arrays near the top of the script. To add a column, add an entry to `STATUSES`.

## Email notifications

When a task is added, the page posts its details to FormSubmit's AJAX endpoint, which emails them as a table. This is the only network request the page makes.

1. Open `index.html` and find this line near the top of the `<script>` block:
   ```js
   const FORMSUBMIT_ENDPOINT = "https://formsubmit.co/ajax/YOUR_EMAIL@example.com";
   ```
2. Replace `YOUR_EMAIL@example.com` with the address that should get the notifications.
3. Add one task. FormSubmit sends an **activation email** to that address. Nothing is delivered until you click the link in it.
4. After activation, every new task sends an email.

The board updates straight away whether or not the email goes through. If sending fails (including before activation), the card stays and a warning toast appears.

## Configuration

These constants sit at the top of the script:

| Constant | Default | Purpose |
|---|---|---|
| `FORMSUBMIT_ENDPOINT` | `https://formsubmit.co/ajax/YOUR_EMAIL@example.com` | Where notification emails are sent |
| `TASK_ID_PREFIX` | `UOB-ITPM-` | Prefix for generated task IDs |
| `TOAST_MS` | `4500` | How long toast messages stay up, in milliseconds |

## Design constraints

- One file: markup, one `<style>` block and one `<script>` block in `index.html`.
- Vanilla HTML, CSS and JavaScript. No frameworks, CDNs, web fonts or image files. Icons are Unicode or inline SVG.
- No browser storage (localStorage, sessionStorage, IndexedDB or cookies).
- No `alert()` / `confirm()` dialogs, and no `!important` in CSS.

See `CLAUDE.md` for architecture notes: the state model, render loop, event delegation and escaping rules.

## Quick checks

Syntax-check the script (needs Node):

```bash
sed -n '/<script>/,/<\/script>/p' index.html | sed '1d;$d' > /tmp/k.js && node --check /tmp/k.js
```

Make sure no storage, dialogs, `!important` or extra URLs have crept in. The only match should be the `FORMSUBMIT_ENDPOINT` line:

```bash
grep -nE 'localStorage|sessionStorage|indexedDB|document\.cookie|alert\(|confirm\(|!important|https?://' index.html
```
