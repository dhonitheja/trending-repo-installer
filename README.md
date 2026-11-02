# trending-repo-installer

A Claude skill that finds trending GitHub repositories, flags the risky ones, and installs only the ones you pick, after checking them first.

Ask things like:

- "show me trending AI repos this week"
- "what are the new MCP servers?"
- "trending Rust repos today, just clone the top one"

## What it does

1. **Finds** fresh repos for your topic, language and timeframe (daily, weekly or monthly).
2. **Ranks** them in one table: new repos by stars per day, established repos in a separate section.
3. **Flags** risk on every row: `NEW`, `SPIKE` (suspicious star growth), `NO-LICENSE`, `NO-README`, `ARCHIVED`, `BIG`.
4. **Waits** for you to choose. Nothing is installed before that.
5. **Vets** each chosen repo: it clones shallow into its own folder, reads install scripts, manifests and dependencies without running anything, and gives a verdict of `CLEAN`, `CAUTION` or `STOP`.
6. **Installs** in isolation (virtualenv, `npm ci --ignore-scripts` first), runs a smoke test, and tells you how to run it and which env vars it needs.

## Where the data comes from

`github.com/trending` blocks automated fetching, so the skill does not use it. Instead:

| Source | Used for |
|---|---|
| GitHub search API (via `gh`) | New repos, star counts, license, last push |
| [OSS Insight](https://ossinsight.io/trending/ai) and Trendshift | Cross-checking real star-growth numbers |
| Web search | Fallback if the above fail |

The GitHub API has no "stars gained this week" field, so for new repos the skill ranks by stars divided by age. It says so when it can't get true growth numbers.

## Safety rules

- Never installs before you pick.
- Never uses `sudo`, global installs, or `curl | sh` without your approval.
- Never writes secrets into files; it names the env vars you must set.
- Stays inside the clone folder.
- Stops and tells you if a repo looks suspicious, and offers to delete the clone.

Star counts on brand-new repos can be inflated. The vetting step reduces risk but is not a security audit, so read anything you plan to run with real credentials.

## Options

- `just clone`: clone and vet, skip the install.
- `dry run`: vet only.
- Loose topic words are mapped to GitHub topics (for example "mcp" to `mcp` and `mcp-server`, "agents" to `ai-agents`).

## Install

Copy the skill folder into your Claude skills directory:

```bash
git clone https://github.com/dhonitheja/trending-repo-installer.git
cp -r trending-repo-installer/skills/trending-repo-installer ~/.claude/skills/
```

On claude.ai, you can instead upload `skills/trending-repo-installer/SKILL.md` as a custom skill.

Requirements: `git` and the GitHub CLI (`gh`). Install tools for the repos you pick (Python, Node, Rust, Go) are needed only for those.

## Using it in ChatGPT

The Claude skill relies on tools ChatGPT doesn't have (a shell, `gh`, direct API calls). For ChatGPT, use `chatgpt/instructions.md` instead:

1. Create a Custom GPT (or a Project) and turn on Web Search.
2. Paste the text below the divider in `chatgpt/instructions.md` into the Instructions field.
3. Ask: "show me trending MCP servers this week".

The ChatGPT version does the same finding, ranking, flagging and safety review, but it can't clone or install anything itself. It gives you the commands to run on your own machine.

## Repository layout

```
skills/trending-repo-installer/SKILL.md   # Claude skill
chatgpt/instructions.md                   # ChatGPT version
README.md
LICENSE
```

## License

MIT
