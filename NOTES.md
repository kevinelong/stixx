# Developer / Agent Handoff Notes — `stixx`

> Purpose: let a fresh agent (or human) take over this repo cold, with no prior
> chat context. Everything project-critical is committed; this file captures the
> "why" and the operational gotchas that aren't obvious from the code.

Last updated: 2026-08-27.

---

## 1. What this repo is

A single self-contained static web page — a **catalog brochure for "Stixx Stand"
billiard products by Bob Grable** (Tualatin, OR) — published as a GitHub Pages
site.

- **Live URL:** https://kevinelong.github.io/stixx/
- **GitHub:** https://github.com/kevinelong/stixx
- **Default branch:** `main`

### Tracked files (the whole project)

| File | Role |
|------|------|
| `index.html` | The entire site. Self-contained: inline CSS, no external images/fonts/JS libs. One tiny inline `<script>` sets the footer year. |
| `README.md` | Placeholder (`# stixx`). |
| `LICENSE` | License. |
| `.gitignore` | Stock Node-style ignores (not otherwise meaningful — no build tooling here). |
| `NOTES.md` | This file. |

There are **no build steps, no dependencies, no asset directory.** What's in the
repo is the whole deliverable. Editing `index.html` and pushing to `main` updates
the live site.

---

## 2. How it's deployed (READ THIS before touching Pages)

Pages is configured as **"Deploy from a branch" → `main` / root** (set by the repo
owner in Settings → Pages). GitHub's built-in "pages build and deployment" job runs
automatically on every push to `main` and serves `index.html` from the repo root.

### Important history / decision

- An earlier attempt used a **GitHub Actions workflow** (`.github/workflows/pages.yml`
  with `actions/deploy-pages`). It was **removed** (PR #2) because the owner chose
  branch-based deploy instead. The two Pages "source" modes are mutually exclusive:
  a `deploy-pages` Actions workflow **fails** when the source is "Deploy from a branch."
- The auto-enable trick (`actions/configure-pages` with `enablement: true`) **does not
  work** here: the workflow's `GITHUB_TOKEN` returns `Resource not accessible by
  integration` when trying to create the Pages site. Enabling Pages the first time
  **requires a human** in Settings → Pages. Don't re-add that workflow expecting it to
  self-enable.

**If you need to switch to Actions-based deploy** (e.g. to add a build step): the owner
must change Settings → Pages source to "GitHub Actions" *first*, then you add the
workflow. Doing it in the other order just produces red failed runs.

---

## 3. How to make common changes

- **Edit page content/design:** edit `index.html`, commit, get it onto `main`. The
  branch build redeploys within ~1–2 min (CDN can cache the old page briefly).
- **Add an image or other asset:** because the page currently has zero external
  assets, you can either inline it (e.g. a `data:` URI) or add a real file to the repo
  and reference it by relative path. Keep paths relative — the site is served under the
  `/stixx/` subpath, so absolute `/foo.png` paths will 404.
- **Verify a deploy:** check the latest "pages build and deployment" run concluded
  `success` (see tooling notes below), then load the live URL.

---

## 4. Environment constraints I hit (so you don't waste time)

This repo is often worked on from a sandboxed remote agent environment. Two limits
matter:

1. **`*.github.io` is blocked by egress policy.** You **cannot** `curl`/fetch the live
   `https://kevinelong.github.io/stixx/` page from inside the agent environment (403 at
   the proxy). To confirm the site renders, ask the human to open it, or rely on the
   Pages build run's `success` conclusion as the proxy for "it deployed."
2. **The GitHub Pages settings API path is blocked, and the GitHub MCP server has no
   Pages-config tool.** So an agent **cannot read or change** Settings → Pages
   (source, custom domain, enablement). That's a human-only step. You *can* see whether
   deploys ran/succeeded via the Actions API.

Everything else (repo contents, commits, PRs, Actions runs/logs) is reachable via the
GitHub MCP tools.

---

## 5. Git / workflow conventions in force

- Development happens on branch **`claude/stixx-repo-github-pages-kii5zg`**, then a PR
  into `main`, squash-merged. (If that PR is already merged, restart the branch from
  latest `main` — don't stack new work on merged history.)
- Never force-push `main`. Force-with-lease on the dev branch is fine when it only
  carries already-merged history plus the new commit.
- Commit messages / PR bodies are the durable "notes." Keep them descriptive.

---

## 6. Current state (as of last update)

- `main` is clean and live. Catalog page published successfully via branch-based Pages.
- No open work items. No `.github/workflows/` directory (intentionally removed).
- Only branches: `main` and the `claude/…` dev branch. A stale
  `copilot/create-billiard-products-catalog` branch may still exist on the remote — its
  content (the original `index.html` + an Actions workflow) is already fully reflected on
  `main`, so it can be deleted if you want to tidy up; nothing depends on it.

---

## 7. Open questions / things a human should confirm

- **Contact details in `index.html`** (phone `503-332-9056`, email
  `stixxstand@gmail.com`) came from the original Copilot-authored draft. Confirm they're
  correct/current before treating the page as production marketing material.
- **Product copy** (chalk holders, template rack holders, the "Stixx Stand" flagship)
  is likewise from that draft — verify against what Bob Grable actually sells.
