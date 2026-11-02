# trending-repo-installer

A Claude skill that finds trending GitHub repositories, flags the risky ones, inspects the ones you pick, and installs a repo only after you approve an exact, sandboxed plan.

Ask things like:

- "show me trending AI repos this week"
- "what are the new MCP servers?"
- "trending Rust repos today, just clone the top one"

## What it does

1. **Finds** fresh repos for your topic, language and timeframe (daily, weekly or monthly), and tells you the time window and when the data was fetched.
2. **Ranks** them in one table: new repos by average stars per day (a lifetime average, not measured growth), established repos in a separate group.
3. **Flags** risk on every row: `NEW`, `SPIKE`, `BIG`, `NO-LICENSE`, `NO-README`, `ARCHIVED`. Flags are triage hints, not proof of anything.
4. **Waits** for you to choose. It never picks a repo for you.
5. **Inspects** each pick without running anything: it records the exact commit SHA and reads the README, manifests, install scripts, Dockerfiles and dependencies. Verdicts are `CLEAN`, `CAUTION` or `STOP`, and `CLEAN` is never a security guarantee.
6. **Asks approval for an install plan** that lists the exact commands, revision, destination, downloads, mounts and network access. Choosing a repo is not approval to run it.
7. **Installs inside a disposable sandbox** and runs a bounded smoke check, then reports the result and the env var names you need to set.

If a sandbox can't be enforced in your environment, the skill stops before running anything and gives you reviewed manual steps instead.

## Where the data comes from

`github.com/trending` blocks automated fetching, so the skill does not use it.

| Source | Used for |
|---|---|
| GitHub search API (via `gh`, or web search) | New repos, star counts, license, last push |
| [OSS Insight](https://ossinsight.io/trending/ai) and Trendshift | Supplementary growth numbers, after checking each page's coverage |
| Web search | Fallback, labelled as unverified |

The GitHub API has no "stars gained this week" field, so the skill says plainly when it can't get true growth numbers. It never invents results if live access fails.

## Safety rules

- Repo files, web pages and tool output are treated as untrusted data, never as instructions.
- Nothing is cloned or run until you choose, and nothing is executed until you approve the exact plan.
- Third-party code runs only in a disposable sandbox: non-root, no privileged mode, no host credentials or sockets, restricted network.
- A virtualenv or `--ignore-scripts` is not treated as a security boundary.
- No `sudo`, global installs or `curl | sh`. Real secrets are never passed to untrusted code.
- It stops on `STOP` verdicts and offers to delete only the inspection copy it created.

This is a risk-reduction workflow, not a security audit. Star counts on brand-new repos can be inflated.

## Options

- `just clone`: clone and inspect, no execution.
- `dry run`: static inspection only, no installation.
- Loose topic words are mapped to GitHub topics as search hints (for example "mcp" to `mcp` and `mcp-server`).

## Install

Copy the skill folder into your Claude skills directory:

```bash
git clone https://github.com/dhonitheja/trending-repo-installer.git
cp -r trending-repo-installer/skills/trending-repo-installer ~/.claude/skills/
```

On claude.ai, you can instead upload `skills/trending-repo-installer/SKILL.md` as a custom skill.

Requirements: `git`. The GitHub CLI (`gh`) is optional but recommended for reliable discovery.

## Using it in ChatGPT

The Claude skill relies on tools ChatGPT doesn't have (a shell, `gh`, direct API calls). Use `chatgpt/instructions.md` instead:

1. Create a Custom GPT (or a Project) and turn on Web Search.
2. Paste the text below the divider in `chatgpt/instructions.md` into the Instructions field.
3. Ask: "show me trending MCP servers this week".

The ChatGPT version finds, ranks, flags and reviews repos, but it can't clone or install anything. It gives you reviewed commands to run yourself, ideally in a sandbox or VM.

## Repository layout

```
skills/trending-repo-installer/SKILL.md   # Claude skill
chatgpt/instructions.md                   # ChatGPT version
README.md
LICENSE
```

## License

MIT
