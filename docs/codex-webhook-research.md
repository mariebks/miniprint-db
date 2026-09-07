# Codex + GitHub push reactivity (miniprint DB → collector site)

**Research date:** 2026-09-06 (PT)  
**Scope:** OpenAI Codex product (Codex CLI / Codex cloud / ChatGPT Codex), not the 2021 Codex model API.  
**Use case:** Private GitHub repo holds canonical `data/machines.json` + `data/updates.ndjson`; Grok Bot pushes after scrapes; fiancée builds a collector website with Codex that should pick up DB updates soon after, without wasteful polling.

---

## Executive answer

| Question | Answer |
|---|---|
| Can **Codex itself** subscribe to GitHub **webhooks / `push` events** and wake an arbitrary coding task? | **No (Codex-native).** There is no documented Codex feature that installs a listener for “any push to this data repo → start a Codex cloud task.” |
| Partial / related reactivity? | **Partial, but not for this use case.** Codex reacts to **PR/issue-shaped** GitHub activity (`@codex` comments, automatic **code review** when a PR is opened for review) and can be **invoked from CI** via `openai/codex-action`. ChatGPT **event-triggered scheduled tasks** (separate from Codex automations) can watch **pull request activity**, not general branch pushes of JSON files. |
| Should the collector site rely on Codex waking on Bot pushes? | **No.** Treat Codex as the **site builder**; treat **GitHub Actions + host deploy hooks** (or runtime fetch of the JSON) as the **reactive update path**. |

**One-line verdict:** Codex does **not** replace CI for “data repo pushed → site updates.” Use push-triggered GitHub Actions / Vercel–Netlify deploy hooks (or fetch-at-runtime). Use Codex to *build* that pipeline, not to *be* the webhook consumer.

---

## What Codex *can* do with GitHub (2026 docs)

### 1. Codex cloud + GitHub connection
- Connect GitHub, create a cloud **environment** for a repo, start tasks from the web / CLI / integrations.
- Tasks clone the repo, run in an isolated container, return a diff/PR.
- **Trigger model:** human/chat/`@codex`/Slack/Linear — not “subscribe to `push` on a data file.”

Sources: [Codex cloud](https://developers.openai.com/codex/cloud), [Use Codex with GitHub / code review](https://learn.chatgpt.com/docs/third-party/github)

### 2. GitHub PR integration (`@codex`, automatic reviews)
Documented triggers:
- Comment `@codex review` (or `@codex …` for a cloud task with PR context).
- Optional **Automatic reviews** when a PR is opened for review.
- Follow-ups like `@codex fix the P1 issue`.

**Not documented:** waking a Codex task solely because `main` received a direct push of `machines.json` with no PR.

Source: [Review GitHub pull requests with Codex](https://learn.chatgpt.com/docs/third-party/github)

### 3. `openai/codex-action` (Codex *inside* GitHub Actions)
- Official Action runs `codex exec` on a runner when **your workflow’s** `on:` fires (`pull_request`, `push`, `workflow_dispatch`, `repository_dispatch`, etc.).
- This is **CI calling Codex**, not Codex owning a webhook endpoint.
- Useful if you want an agent to rewrite site code after a data change; overkill if the site only needs fresh JSON.

Sources: [Codex GitHub Action](https://developers.openai.com/codex/github-action), [openai/codex-action](https://github.com/openai/codex-action)

### 4. Scheduled tasks / Codex automations (clock-driven)
- Recurring background tasks (RRULE / daily / minute follow-ups).
- Desktop/local automations need the machine/app running; cloud-oriented patterns vary by surface.
- Docs explicitly distinguish: **ChatGPT event-triggered tasks** vs **Codex separate automations**.
- A schedule that `git pull`s is **polling by cadence**, not event-driven.

Sources: [Scheduled tasks](https://learn.chatgpt.com/docs/automations), [Help: Tasks in ChatGPT](https://help.openai.com/en/articles/10291617-tasks-inchatgpt)

### 5. ChatGPT event-triggered tasks (adjacent, not Codex)
- Eligible plans; web/mobile; Help Center calls them **“webhook-based”** internally.
- Supported GitHub trigger: **pull request activity** (filters: PR, author, title, label; reviews / comments / commit updates / merges).
- **Not** documented as: “any `push` to a private data repo with JSON commits.”
- Event triggers unavailable in Codex CLI / IDE extension; Help Center: *“Codex uses separate automations.”*

Sources: [Scheduled tasks → Trigger tasks from app events](https://learn.chatgpt.com/docs/automations), [Help Center](https://help.openai.com/en/articles/10291617-tasks-inchatgpt)

### 6. GitLab contrast (do not confuse with GitHub)
- GitLab setup docs describe Codex installing **project/group webhooks** for MR/comment/issue delivery, and review policies including **On every push** (on MRs).
- That is **GitLab-specific** integration plumbing. It does **not** imply Codex exposes a user-configurable GitHub push→task webhook API for arbitrary repos.

Source: [Use Codex with GitLab (Beta)](https://learn.chatgpt.com/docs/third-party/gitlab)

---

## Practical compatible patterns (2026)

Prefer patterns that are **push-driven** and **deterministic** for a JSON DB + static/SSR collector site.

### A. Best default: host deploy on GitHub `push` (no Codex in the hot path)
1. Bot pushes to **data repo** (`data/machines.json`, `data/updates.ndjson`).
2. **Website repo** either:
   - vendors/copies data at build time (submodule, sparse checkout, or Actions download), **or**
   - lives in the **same** monorepo as the data (simplest).
3. Vercel / Netlify / Cloudflare Pages / GitHub Pages connected to the site repo → deploy on push.
4. If data is a **separate** private repo: data-repo Action on `push` paths `data/**` calls the host **Deploy Hook** or `repository_dispatch` / `workflow_dispatch` on the site repo.

### B. Runtime fetch (often simplest for “DB updates”)
1. Site reads JSON at request time (or short-TTL cache) from:
   - GitHub Contents API / raw URL (token for private), or
   - object storage / CDN that Bot also updates.
2. No rebuild; no Codex; no polling of Codex.
3. Caveat: private GitHub rate limits, token handling, and cache headers.

### C. GitHub Actions rebuild without calling Codex
```yaml
# conceptual — data repo
on:
  push:
    paths: ['data/**']
jobs:
  notify-site:
    runs-on: ubuntu-latest
    steps:
      - name: Trigger site rebuild
        run: curl -X POST "$DEPLOY_HOOK_URL"
```
Or checkout both repos, build, deploy artifact.

### D. Actions → Codex only when *code* must change
Use `openai/codex-action` when a data schema change needs generated UI/types/pages. Do **not** burn Codex credits on every scrape push if the site already consumes JSON.

### E. Codex scheduled `git pull` (acceptable fallback, not ideal)
- Cadence-based pull/rebuild from desktop automation or a cron Action.
- Works, but is **polling**; lag and wasted runs vs push-triggered deploy.

### F. Cursor Cloud Agents
- Same class of tool as Codex for *authoring*; not a GitHub webhook bus for production data updates. Prefer Actions/deploy hooks for the live pipeline.

---

## Recommended handoff architecture (miniprint)

**Goal:** Bot pushes JSON → website reflects updates soon after, **without** wasteful polling and **without** depending on Codex waking.

1. **Keep the private GitHub data repo as canonical** (`data/machines.json`, `data/updates.ndjson`). Bot continues to push after scrapes.
2. **Build the collector site so data is a build-time or runtime input**, not something Codex must re-implement on every push.
3. **Primary path (recommended):**
   - Put site + data in one repo **or** wire **data-repo `push` → Deploy Hook / `repository_dispatch` → site rebuild**.
   - Host on Vercel/Netlify/Cloudflare/GitHub Pages with GitHub integration.
4. **Optional runtime path:** site fetches latest JSON (authenticated) with short cache if you want zero rebuild latency.
5. **Use Codex (fiancée’s setup) to implement (2)–(4)** once: Actions YAML, deploy hook, fetch client, types for `machines.json`. Then Codex leaves the hot path.
6. **Only if** scrape payloads require new site code: add a path-filtered Action that optionally runs `openai/codex-action` or opens an issue/`@codex` task — rare, not per-scrape.
7. **Do not** design around “Codex listens for webhooks on the data repo” or ChatGPT PR event tasks for direct JSON pushes.

### Numbered ops checklist for Kyle ↔ fiancée
1. Decide: **same repo** vs **data repo + site repo**.
2. If split: create a Deploy Hook (or site workflow `repository_dispatch`) and an Action on data `push` paths `data/**`.
3. Grant the site/host a read token only if runtime/private fetch is used; prefer build-time checkout with `actions/checkout` + deploy credentials.
4. Document expected lag (usually seconds–minutes for Actions+host, not hours).
5. Keep Codex for feature work; keep CI/host for refresh.

---

## Codex-specific limits (summary)

| Capability | Status (per official docs, Sep 2026) |
|---|---|
| Native GitHub webhook listener for arbitrary `push` → coding task | **Not documented / effectively no** |
| `@codex` on PR/issue comments → cloud task / review | **Yes** |
| Automatic PR code review | **Yes** (settings toggle) |
| Clock-based scheduled tasks / automations | **Yes** (surface-dependent; local needs app on) |
| ChatGPT GitHub **PR activity** event tasks | **Yes** (ChatGPT Work; **not** Codex automations; not general push) |
| Run Codex from GitHub Actions on `push` | **Yes** (`openai/codex-action` — CI owns the webhook) |
| Codex CLI as always-on webhook server | **No** (CLI is local/CI agent, not a public event bus) |

**Ambiguity call-out:** OpenAI does not publish a public “Codex inbound webhook URL you register on a repo for push events.” GitLab docs mention webhooks Codex *installs* for MR flows; GitHub docs emphasize app connection + PR review/`@codex`. If a private beta exists, it is not in the public developer docs reviewed here — treat as **unavailable**.

---

## Caveats / what Kyle should tell his fiancée’s Codex setup

1. **Don’t ask Codex to “subscribe to the data repo webhook.”** Ask it to scaffold **GitHub Actions + deploy hook** (or runtime JSON fetch).
2. Distinguish products: **Codex coding agent** ≠ **ChatGPT event-triggered tasks** ≠ **GitHub Actions**. Mixing them causes false expectations.
3. **PR-only event tasks** won’t fire on Bot’s typical direct pushes to `main` unless you change the Bot to open PRs (usually unnecessary for a data feed).
4. Private repo: host/build needs a **read credential**; don’t embed long-lived tokens in the client bundle — use server/build-time secrets.
5. Scrapes may be frequent: prefer **path filters** (`data/**`) and host deploy hooks over invoking Codex every push.
6. If the site is static and only displays JSON, **Codex is the carpenter, not the conveyor belt.**
7. Re-check docs before launch; Codex surfaces evolve quickly in 2026 — especially automations vs ChatGPT Scheduled.

---

## Sources / links used

### Official
- [Codex cloud](https://developers.openai.com/codex/cloud) (also `.md`)
- [Codex GitHub Action](https://developers.openai.com/codex/github-action) / [learn.chatgpt.com GitHub Action](https://learn.chatgpt.com/docs/github-action)
- [openai/codex-action](https://github.com/openai/codex-action)
- [Scheduled tasks / automations](https://learn.chatgpt.com/docs/automations)
- [Review GitHub PRs with Codex](https://learn.chatgpt.com/docs/third-party/github)
- [Use Codex with GitLab (Beta)](https://learn.chatgpt.com/docs/third-party/gitlab) — webhook contrast only
- [Help Center: Tasks in ChatGPT](https://help.openai.com/en/articles/10291617-tasks-inchatgpt)
- [Non-interactive / CI patterns](https://developers.openai.com/codex/non-interactive-mode)
- [Cookbook: autofix with GitHub Actions](https://developers.openai.com/cookbook/examples/codex/autofix-github-actions)

### Secondary (context only; not used as primary proof)
- OpenAI Developer Community threads on Codex cloud workflow triggers
- Third-party writeups comparing Actions vs Codex automations

---

## Bottom line for miniprint

**Codex-native webhooks for data-repo pushes: no.**  
**Compatible reactive pattern: GitHub `push` → Actions / Deploy Hook → site update (or runtime fetch).**  
**Codex’s job: build that site and that CI once; stay out of the per-scrape loop.**
