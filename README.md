# Throughline: IT PMO Kanban Board

Throughline is a single-file Kanban board for a fictitious bank's IT Project Management Office. It's built for demos and training: log a task, move it across the board, and get an email notification when a new task is added.

Everything lives in `index.html`. There's no build step, no dependencies and no server.

> This is a demo. It doesn't use any real bank's branding and isn't connected to any real system.

## Live demo

- **v2 (Throughline):** https://sbnsalam.github.io/it-pmo-kanban/v2/
- **v1 (original):** https://sbnsalam.github.io/it-pmo-kanban/ (frozen at tag `v1.0.0`)

The Pages workflow serves `index.html` from the `v1.0.0` tag at the site root and the current `main` under `/v2/`.

## Getting started

Double-click `index.html`, or open it in any modern browser. It works straight from `file://`.

The board starts with seed tasks. Their due dates are set relative to today, so a few are always overdue.

**Nothing is saved.** Refreshing the page resets the board to the seed data.

## Features

- **Four columns:** Backlog, In Progress, Blocked, Done. Each column header shows a count of its visible tasks.
- **Add Task panel** slides in from the right when you click the **+** button at the top right, with inline validation. Title, Assignee and Due Date are required. You can also set Description, Project / Workstream, Category, Priority and Status.
- **Task IDs** are generated as `ITPM-0001`, `ITPM-0002`, and so on.
- **Moving tasks:** drag a card to another column, or use the card's **Move ▸** dropdown (keyboard friendly).
- **Delete** with an inline "Delete? Yes / No" confirmation.
- **Filters** by project, assignee (text match) and priority.
- **Overdue highlighting:** a task is overdue when its due date has passed and it isn't Done.
- **Header summary** of totals across all tasks, ignoring filters.
- **Cards** show a priority pill with signal bars, a two-line description, project and category, an assignee avatar, and a due chip that reads "Due tomorrow" / "3 days overdue" (amber within 3 days, red when overdue).
- **Full-screen board:** on screens 1024px and wider the board fills the window under the header and each column scrolls on its own.
- **Icon buttons:** Add Task is an icon-only **+** button; Help, FAQ and About are icon-only too (question mark, speech bubble, info). All have accessible names and tooltips.
- **Burn-down view**, the default when the page opens (switch with **Burn-down / Board** next to the filters): open tasks vs. an ideal line over a 21-days-back / 14-days-ahead period, tasks completed per week, and stat tiles for pace and projected finish (On track / Behind). Charts follow the filters, have hover and keyboard tooltips, and a data-table view. Opening the view animates the charts (lines draw left to right, bars grow); reduced-motion settings turn this off.
- **Light and dark themes:** follows the system setting; the sun / moon button in the header switches it until the next refresh.
- **Help, FAQ and About** in one modal guide opened from the header. Press `?` to open Help.
- **Responsive:** the board goes two-up below 1200px and stacks into one column below 768px.
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
| `APP_NAME` | `Throughline` | Name used in the email subject |
| `APP_VERSION` | `2.0` | Shown on the About tab |
| `TASK_ID_PREFIX` | `ITPM-` | Prefix for generated task IDs |
| `TOAST_MS` | `4500` | How long toast messages stay up, in milliseconds |
| `BURNDOWN_PAST_DAYS` / `BURNDOWN_AHEAD_DAYS` | `21` / `14` | Burn-down period: start before today, end (ideal hits zero) after today |
| `VELOCITY_DAYS` | `14` | Completions in this window set the pace for the projected finish |
| `WEEKS_SHOWN` | `5` | Bars in the completed-per-week chart |
| `DUE_SOON_DAYS` | `3` | Open tasks due within this many days get an amber chip |

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
