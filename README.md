# IT PMO Project Board

[![CI](https://github.com/MTRYTECH/it-pmo-kanban/actions/workflows/ci.yml/badge.svg)](https://github.com/MTRYTECH/it-pmo-kanban/actions/workflows/ci.yml)

A single-file Kanban board for an IT PMO, built as an internal demo/training tool for a fictitious bank.

**Live demo:** https://MTRYTECH.github.io/it-pmo-kanban/

## Features
- Four-column board with a summary strip counting all tasks
- Drag and drop between columns, plus a keyboard-friendly "Move" select on each card
- Add-task dialog with inline validation; task IDs follow the `UOB-ITPM-####` format
- Inline "Delete? Yes/No" confirmation (no browser pop-ups)
- Filters by project, assignee and priority
- Seed data with due dates relative to today, so there are always overdue and upcoming items
- Toast notifications for feedback

## Run locally
No build, no package manager and no server. Open `index.html` directly in a browser.

## Constraints
- Vanilla HTML, CSS and JS only: no frameworks, CDN scripts, web fonts or image files
- No persistence: nothing is stored, so a refresh resets the board to the seeded data
- The only network call is the optional new-task email notification

## Email notification (FormSubmit)
Adding a task sends a notification through [FormSubmit](https://formsubmit.co). Set the address in the `FORMSUBMIT_ENDPOINT` constant at the top of the script in `index.html`, replacing `YOUR_EMAIL@example.com`. FormSubmit requires a one-time email activation for a new address. Until it is configured, adding a task shows an "email notification failed" warning toast; the board keeps working. Do not commit a real address to a public repo.

## Project structure
| Path | Purpose |
| --- | --- |
| `index.html` | All markup, styles and script |
| `CLAUDE.md` | Guidance for Claude Code when editing the project |
| `.github/workflows/ci.yml` | Constraint checks and GitHub Pages deploy |
| `.claude/commands/publish-github.md` | Custom command that publishes the project to GitHub |

## CI/CD
GitHub Actions checks the project's constraints on every push and pull request (no browser storage, no `!important`, no `alert`/`confirm`, no external scripts or styles) and deploys `index.html` to GitHub Pages on pushes to `main`.

This is a demo with fictitious data. It has no affiliation with any real bank.
