---
name: trending-repo-installer
description: Find trending GitHub repos (by language, topic, or timeframe) and clone/install the ones the user picks. Use when asked for trending, popular, or hot repos, or to try one out.
---

# Trending Repo Finder & Installer

Find what's trending on GitHub, rank it sensibly, flag risks, then clone and set up only the repos the user picks.

## 1. Clarify (one short question at most)
Defaults: timeframe = weekly, language = any, count = 10, mode = find + install.
If the topic is vague ("AI repos") and the kind matters, ask ONE question: agents / dev tools / models & inference / learning material / anything. Otherwise start immediately.

Map loose words to GitHub topics: ai -> `ai`, `llm`; agents -> `ai-agents`, `agent`; mcp -> `mcp`, `mcp-server`; rag -> `rag`; coding agents -> `claude-code`, `coding-agent`; skills -> `agent-skills`, `claude-skills`.

Users can say "just clone" (no install) or "dry run" (vet only). Respect it.

## 2. Fetch (live data only, never from memory)
github.com/trending blocks automated fetching (robots). Do NOT try it. Use these, in parallel where possible:

A. **GitHub API via `gh`** (preferred; the proxy is preconfigured):
   - New and fast-rising: `gh api -X GET search/repositories -f q="created:>YYYY-MM-DD topic:<topic>" -f sort=stars -f order=desc -f per_page=15` (date = 7 days back for weekly, 1 for daily, 30 for monthly; add `language:<lang>` if requested).
   - Established but active: `q="topic:<topic> pushed:>YYYY-MM-DD stars:>5000"`.
   - Get `full_name, stargazers_count, language, created_at, pushed_at, license.spdx_id, description`.
B. **Growth-ranked sites via WebFetch**: `https://ossinsight.io/trending/ai` (28-day growth) and Trendshift. These give real star-growth numbers; use them to cross-check.
C. If everything fails, WebSearch "github trending <topic> <month year>".

If the shell can't reach a site, do not retry with other shell tricks; use WebFetch/WebSearch.

## 3. Rank and present (one table)
Compute for new repos: `stars_per_day = stars / max(age_days, 1)`. Rank new repos by it. Put established repos in a clearly labeled second section ("Established, recently active"), ranked by growth if a source gave it, else stars.

Columns: #, `owner/repo`, what it is (one line), lang, stars, age or last push, license, source tag (API / OSSInsight / Trendshift), flag.

Flags (be honest, short):
- `NEW` under 14 days old
- `SPIKE` very high stars relative to age with tiny commit history (possible star inflation)
- `NO-LICENSE`, `NO-README`, `ARCHIVED`
- `BIG` very large codebase or heavy deps

Say clearly if you could not get true weekly growth numbers. End with **Sources:** as links. Keep it to one table plus at most two sentences of commentary.

Then ask which to install (numbers or names). NEVER install before the user picks.

## 4. Vet before installing (required for every chosen repo)
Clone shallow into a new, empty directory (default `~/trending/<repo>`, or the user's chosen folder):
`git clone --depth 1 https://github.com/<owner>/<repo>.git <dir>`

Inspect without executing:
- README install section, license, last commit, commit count, contributors, open issue/star sanity
- Manifests: `package.json` (`preinstall`/`postinstall`/`prepare`), `setup.py`/`pyproject.toml`, `Makefile`, `Cargo.toml`, `go.mod`, `requirements.txt` (unusual or typosquatted packages, pinned git URLs)
- `install.sh`, `curl | bash` patterns, base64/obfuscated blobs, hidden network calls, credential or wallet access

Report 2-4 lines with a verdict: CLEAN / CAUTION / STOP, plus reasons. On STOP, do not install; tell the user why and offer to delete the clone.

## 5. Install (skip if the user said just clone or dry run)
Detect the stack and use the documented method, preferring isolation:
- **Python**: `python -m venv .venv && . .venv/bin/activate && pip install -r requirements.txt` (or `pip install -e .`)
- **Node**: `npm ci --ignore-scripts` first; rerun with scripts only if they were reviewed and needed
- **Rust**: `cargo build --release`
- **Go**: `go build ./...`
- **Docker-first projects**: follow the README, don't run privileged containers without asking

Rules:
- Never use `sudo`, global installs (`-g`, `--user` system-wide), or `curl | sh` without explicit user approval.
- Never paste secrets into config; if an API key is required, tell the user which env var to set.
- Don't modify files outside the clone directory.

## 6. Verify, hand off, clean up
Run the project's smoke check (`--help`, `--version`, or its test command) and report pass/fail. Then give: install location, how to run it, required env vars, and any warnings. If install or smoke test failed, offer to delete the clone. Offer one natural next step (try an example, or install the next repo on the list).

## Notes
- If a clone fails (private, renamed, or blocked), say so plainly and offer the next repo on the list.
- Keep the whole flow concise: results table, vet verdict, install outcome.
