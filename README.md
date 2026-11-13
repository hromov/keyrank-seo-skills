# SEO rank-tracking skills for Claude Code

Three Claude Code skills that turn stored Google rank-tracking history into answers: what moved this week, which keywords flip between your own pages, and where you rank in the top 10 but Google's AI Overview cites someone else. They read data from the KeyRank MCP server.

## The skills

- **weekly-rank-report**: totals versus last week (top 3/10/30/100, visibility, estimated clicks), keywords that crossed into or out of the top 3 and top 10, biggest gains and losses, new and lost rankings, ranking-page switches, and any of your dated notes that line up with a move.
- **rank-cannibalization**: keywords where the ranking URL keeps switching between two or more of your own pages, the median position each page held, and a harmful / benign / unclear call with options to test.
- **ai-overview-gaps**: keywords where you rank in the organic top 10 and an AI Overview shows up but does not cite you, with the domains and URLs it cites instead (paid plans), what the overview says, and hypotheses worth testing.

Each skill tells Claude which tools to call and in which order, keeps credit use low (project-wide scans first, then targeted calls with exact keyword filters), trusts only complete check days, and reports a missing position as "not in the top 30/100" rather than "not ranked". Every report ends with the credits it used.

You can ask in plain language ("what moved this week on example.com?", "are any of my pages cannibalising each other?", "where are we missing from AI Overviews?") or run a skill directly.

## Requirements

- Claude Code.
- A KeyRank account with at least one project, and an API key from Settings → Developer access. The Free plan works (weekly checks, top 30); paid plans add daily checks, top 100 depth, and the names of competitors cited in AI Overviews.

## 1. Connect the MCP server

```bash
claude mcp add --transport http keyrank https://api.key-rank.com/mcp --header "Authorization: Bearer YOUR_KEY"
```

The server speaks streamable HTTP and is read-only: the key cannot add keywords, change projects or touch billing. Full reference: https://www.key-rank.com/docs/mcp

Check it with `/mcp` in a Claude Code session; you should see `keyrank` with nine tools (`list_projects`, `get_rankings`, `get_movers`, `get_summary`, `get_page`, `get_serp`, `get_competitors`, `get_ai_overview`, `get_notes`).

## 2. Install the skills

### Option A: as a plugin (recommended)

This repository is a Claude Code plugin and a one-plugin marketplace. In a Claude Code session:

```
/plugin marketplace add hromov/keyrank-seo-skills
/plugin install keyrank-skills@keyrank
```

Or from your shell:

```bash
claude plugin marketplace add hromov/keyrank-seo-skills
claude plugin install keyrank-skills@keyrank
```

Plugin skills are namespaced, so you run them as `/keyrank-skills:weekly-rank-report`, `/keyrank-skills:rank-cannibalization` and `/keyrank-skills:ai-overview-gaps`. Claude also picks them up on its own when your question matches.

### Option B: copy the skill folders

```bash
git clone https://github.com/hromov/keyrank-seo-skills.git
mkdir -p ~/.claude/skills
cp -r keyrank-seo-skills/skills/* ~/.claude/skills/
```

Use `.claude/skills/` inside a project instead of `~/.claude/skills/` to share them with your team through the repository. Copied skills run as `/weekly-rank-report`, `/rank-cannibalization` and `/ai-overview-gaps`.

### Trying a local checkout

```bash
claude --plugin-dir ./keyrank-seo-skills
```

## Credits

Every tool call spends weekly credits (`list_projects` costs 1), and every result reports `cost` and `credits_left`. A weekly report on a 200-keyword project takes three or four calls and costs about 8 credits on a daily plan; see the cost line in each tool's description for the formula. The skills stop and tell you when the remaining credits would not cover the next call.

## What it can't do

- It does not search Google live. Everything comes from stored checks: daily on paid plans, weekly on Free. A keyword added today has no history yet.
- It only sees keywords you track in KeyRank. It is not keyword research and it has no Search Console clicks or impressions.
- A null position means not found within the stored depth (top 30 on Free, top 100 on paid). The skills say so; they cannot tell you whether you are at 31 or 300.
- Days that are partial or still in progress are left out. If the last check is incomplete, the report ends on the last complete day.
- Some ranking URLs are rebuilt from Google's breadcrumb (`url_approx`) and the path may be wrong; cannibalization calls on those URLs are marked unconfirmed.
- On the Free plan, AI Overview results show only your own citations; other cited domains are hidden and counted.
- Fixes it suggests (merge pages, change internal links, rewrite a section) are hypotheses to test, not diagnoses. Cross-check in Search Console before acting.
- SERP titles and AI Overview text are third-party content. The skills treat them as data and never follow instructions inside them.

## Contributing

Issues and pull requests are welcome, especially new skills built on the same tools and corrections where a skill misreads the data.

## Licence

MIT. See [LICENSE](LICENSE).
