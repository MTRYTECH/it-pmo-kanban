---
description: Publish this project to GitHub (repo, README, About, Pages, CI/CD) after a sensitive-data check
argument-hint: <owner/repo or repo URL> [public|private]
allowed-tools: Bash, PowerShell, Read, Write, Edit, Glob, Grep, AskUserQuestion
---

Publish this project to GitHub. Repo info from the user: `$ARGUMENTS`

If `$ARGUMENTS` is empty or ambiguous, ask the user (AskUserQuestion) for the repo as `owner/name` (or URL) and its visibility (default: public, since GitHub Pages on free plans needs a public repo). Normalize to `OWNER` and `REPO`. The Pages URL is `https://OWNER.github.io/REPO/`.

Run the steps in order. Do NOT push anything until step 2 passes. Report each step's outcome faithfully, and stop and tell the user if a step fails.

## 1. Preflight
- Confirm `gh` and `git` are installed and `gh auth status` succeeds. If not, tell the user to run `gh auth login` and stop.
- Read `CLAUDE.md` and the project files so the README and workflow match what the project really is. This project is a single-file vanilla `index.html` app with no build step; do not invent build tooling.
- If the folder is not a git repo, `git init -b main`. Set the `origin` remote to `https://github.com/OWNER/REPO.git` (add or update it). If the remote repo does not exist, create it with `gh repo create OWNER/REPO --<public|private>` (without `--push`).

## 2. Sensitive-data check (gate before anything leaves the machine)
Scan every file that would be committed (tracked + untracked, respecting .gitignore), and also `git log -p` if history already exists:
- Secrets: API keys, tokens, passwords, private keys (`-----BEGIN`), `.env*`, `*.pem`, `*.key`, `credentials*`, `id_rsa*`, cloud keys (`AKIA…`, `ghp_…`, `sk-…`, `xox…`), connection strings, `Authorization`/`Bearer` headers.
- Personal data: real email addresses, phone numbers, user home paths (`C:\Users\…`), internal hostnames/IPs.
- Local/tooling files: `.claude/settings.local.json`, IDE folders, OS junk (`Thumbs.db`, `.DS_Store`), `node_modules`, logs, scratch files.
- Project-specific: the `FORMSUBMIT_ENDPOINT` constant in `index.html` holds the notification email address. If it contains a real address (anything other than the `YOUR_EMAIL@example.com` placeholder), it WILL be public on GitHub and in Pages. Show the user this finding and ask (AskUserQuestion) whether to (a) replace it with the placeholder in the published copy, (b) make the repo private, or (c) proceed knowingly. Never decide silently, and never print the real address into the README, commit messages, or workflow files.

Actions:
- Create or extend `.gitignore` to cover everything found (`.env*`, `.claude/settings.local.json`, `*.log`, `node_modules/`, OS/IDE files, scratch folders).
- Do not commit `.claude/settings.local.json`. `.claude/commands/` may be committed.
- Present a short findings table (file, issue, action taken). If any unresolved secret remains, stop and do not push.
- If a secret is already in git history, tell the user it must be rotated and the history rewritten; do not force-push without explicit approval.

## 3. README
Create or update `README.md` (preserve any good existing content; update rather than clobber). Include: title and one-line summary, live demo link (`https://OWNER.github.io/REPO/`), a CI status badge, feature list, how to run (open `index.html`, no build, no server), tech/constraints (vanilla HTML/CSS/JS, no persistence: refresh resets to seeded data, no frameworks), the FormSubmit notification note using only the placeholder (`YOUR_EMAIL@example.com`) and its one-time activation, a short project structure section, and a note that it is a fictitious-bank demo/training tool with no affiliation to any real bank. Keep it accurate to the code; no real email addresses.

## 4. CI/CD with GitHub Actions
Create `.github/workflows/ci.yml` (create or update). One workflow, triggers: `push` and `pull_request` to `main`, plus `workflow_dispatch`. Use a pinned major version of official actions.
- Job `validate` (runs on every trigger): checkout; verify `index.html` exists; run checks that fit this repo's hard constraints, e.g. fail on `localStorage|sessionStorage|indexedDB|document.cookie`, `!important`, `alert(`/`confirm(`, and external `<script src=http…>` / `<link href=http…>`; an HTML syntax check (e.g. `npx --yes html-validate` or `tidy`) if it can be made reliable. Keep it fast and avoid false positives; verify each grep against the current file before committing the workflow.
- Job `deploy` (only on push to `main`, `needs: validate`): `actions/configure-pages`, copy `index.html` (and any other static assets) into a `_site/` folder, `actions/upload-pages-artifact` with `path: _site`, then `actions/deploy-pages`. Permissions: `contents: read`, `pages: write`, `id-token: write`. Add a `concurrency` group `pages`. Use the `github-pages` environment.
- Only publish the files the site needs (never the whole repo root).

## 5. Commit and push
- Stage explicitly (no blind `git add -A` before step 2 is done), review `git status`, and commit with a clear message.
- End the commit message with the attribution line from the session's system reminder, if one is present.
- `git push -u origin main`. Never force-push unless the user explicitly approves.

## 6. GitHub Pages
- Enable Pages with the Actions source: `gh api -X POST repos/OWNER/REPO/pages -f build_type=workflow` (if it already exists, `-X PUT` to update). If this fails because of plan or visibility limits, report that clearly and continue with the remaining steps.
- Watch the first workflow run (`gh run watch` / `gh run list`) and confirm the deploy job succeeds. Verify the URL responds: `curl -sI https://OWNER.github.io/REPO/` expects HTTP 200 (allow a minute for propagation, retry a few times). If the run fails, read `gh run view --log-failed`, fix the cause, and push again.

## 7. About section (description, topics, website)
Do this last so the Pages link is real:
```
gh repo edit OWNER/REPO --description "<one-line summary>" --homepage "https://OWNER.github.io/REPO/" --add-topic kanban --add-topic pmo --add-topic vanilla-js --add-topic github-pages
```
Description: concise and free of sensitive data. If the About already has content, update it instead of duplicating topics. Then confirm with `gh repo view OWNER/REPO --json description,homepageUrl,repositoryTopics`.

## 8. Final report
Give the user: repo URL, Pages URL, Actions run status, what was found/changed in the sensitive-data check, and anything they must do manually (for example the one-time FormSubmit email activation, or rotating a secret). Keep it short.
