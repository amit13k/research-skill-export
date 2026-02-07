# Claude Code Research Skill System

A comprehensive multi-source research system for Claude Code that orchestrates Google, Reddit, Hacker News, GitHub, and official forums to deliver deep, sourced research on any topic.

## What This Does

When you say "research X" or "find out about X", Claude Code will:

1. **Clarify intent** — asks a targeted question to lock scope before searching (for ambiguous queries)
2. **Classify** the topic (tech, product, news, opinion, how-to, troubleshooting)
3. **Generate diverse queries** using vocabulary expansion (3 different framings)
4. **Search in parallel** across Google, Reddit, HN, GitHub, and official forums
5. **Read pages intelligently** using Best-First Search with a Promise Score heuristic
6. **Follow links** into deeper content (docs, papers, changelogs) when promising — no fixed depth limit
7. **Synthesize** findings into a sourced summary with community sentiment

## Architecture

```
research (orchestrator)
├── google-search-browser    — Google search via Playwright browser
├── reddit-search            — Reddit search + comment extraction
├── hackernews-search        — HN Algolia search + discussion reading
├── github-search            — GitHub issues/discussions via Google + MCP tools
└── browser-fetch            — Universal page reader (replaces WebFetch)
```

The `research` skill is the entry point. It dynamically loads sub-skills as needed based on topic classification.

## Prerequisites

- [Claude Code CLI](https://docs.anthropic.com/en/docs/claude-code) installed
- Node.js (for npx/Playwright)
- A GitHub PAT token (for GitHub MCP server)
- Chrome/Chromium (Playwright will use it)

## Installation

### 1. Install MCP Servers

Add these to your `~/.claude.json` under the `mcpServers` key (or use `claude mcp add`):

```jsonc
// See config/mcp-servers.json for the full config
{
  "mcpServers": {
    "github": {
      "type": "http",
      "url": "https://api.githubcopilot.com/mcp",
      "headers": {
        "Authorization": "Bearer <YOUR_GITHUB_PAT_TOKEN>"
      }
    },
    "playwright-logged-in": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@playwright/mcp@latest", "--user-data-dir=<PATH_TO_CHROME_PROFILE>"]
    },
    "playwright-fresh": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@playwright/mcp@latest"]
    }
  }
}
```

**Replace:**
- `<YOUR_GITHUB_PAT_TOKEN>` with your GitHub Personal Access Token
- `<PATH_TO_CHROME_PROFILE>` with a path to a dedicated Chrome profile directory (e.g., `~/.config/claude-chrome-profile`)

### 2. Install Skills

Copy the skill directories into your Claude Code skills folder:

```bash
cp -r skills/* ~/.claude/skills/
```

This installs:
- `~/.claude/skills/research/` (main orchestrator)
- `~/.claude/skills/google-search-browser/`
- `~/.claude/skills/reddit-search/`
- `~/.claude/skills/hackernews-search/`
- `~/.claude/skills/github-search/`
- `~/.claude/skills/browser-fetch/`

### 3. Configure Settings

Copy the settings files:

```bash
# Main settings — denies WebFetch (forces browser usage), enables thinking
cp config/settings.json ~/.claude/settings.json

# Local settings — auto-allows MCP tools, sets up hooks for auto-approval
cp config/settings.local.json ~/.claude/settings.local.json
```

### 4. Install the Auto-Allow Hook

The hook auto-approves MCP tool calls so you don't get prompted for every browser/GitHub action:

```bash
mkdir -p ~/.claude/hooks
cp hooks/allow.sh ~/.claude/hooks/allow.sh
chmod +x ~/.claude/hooks/allow.sh
```

### 5. Add CLAUDE.md Instructions

Append the research rules to your `~/.claude/CLAUDE.md`:

```bash
cat config/CLAUDE.md >> ~/.claude/CLAUDE.md
```

Or manually copy the relevant sections from `config/CLAUDE.md` into your existing `~/.claude/CLAUDE.md`.

## Usage

Once installed, just tell Claude Code to research anything:

```
research latest developments in AI agents
find out about Ghostty terminal emulator
what does reddit think about Cursor vs Claude Code
compare Bun vs Deno for a new project
is SvelteKit good for production apps
```

The research skill triggers on natural language — "research", "find out about", "look into", "compare", "search for", "what is", etc.

## How It Works

### Intent Clarification

Before searching, the skill asks a targeted clarifying question when the query is ambiguous (e.g., provider-specific products, usage/billing questions, "easiest way" requests). This prevents wasted research on the wrong scope.

### Best-First Search with Promise Scoring

The research skill doesn't just read the first Google results. It maintains a priority queue of all discovered URLs and scores each one:

| Factor | Points | Description |
|--------|--------|-------------|
| Relevance | 0-30 | Does the title/snippet address the question? |
| Credibility | 0-25 | Official source (25) > Expert blog (20) > Forum (15) > Publication (12) > Random blog (8) > SEO spam (0) |
| Novelty | 0-25 | Would this add NEW information vs what's already known? |
| Context | 0-20 | Linked from authoritative page? Recent? High engagement? |

Depth decays the score gently (x0.9 per level), but a high-value deep link can outrank a mediocre shallow one. There is no fixed depth limit — depth is governed by scoring and diminishing returns.

### Stopping Conditions

Research stops when any of:
- Diminishing returns (last 2 reads confirmed existing knowledge)
- Queue starved (no URLs with score >= 20)
- Confident answer reached (verified via Answer-Completeness Gate)
- Saturation (same facts keep appearing across independent sources)

### Official Source First

The skill always reads the official source before any third-party coverage. If Anthropic announced something, it reads anthropic.com first, not a TechCrunch summary.

### Ecosystem & Practicality Search

When asked for the "easiest" or "most common" approach, the skill doesn't stop at official docs — it also searches for popular integrations, plugins, wrappers, and community-standard tooling that users actually deploy.

## File Structure

```
research-skill-export/
├── README.md                          ← You are here
├── skills/
│   ├── research/
│   │   ├── SKILL.md                   ← Main orchestrator skill
│   │   └── memory/
│   │       └── _general.md            ← Cross-topic research strategies
│   ├── google-search-browser/
│   │   └── SKILL.md                   ← Google search via Playwright
│   ├── reddit-search/
│   │   └── SKILL.md                   ← Reddit search + extraction
│   ├── hackernews-search/
│   │   └── SKILL.md                   ← HN Algolia search
│   ├── github-search/
│   │   └── SKILL.md                   ← GitHub issues/discussions search
│   └── browser-fetch/
│       └── SKILL.md                   ← Universal page reader
├── config/
│   ├── mcp-servers.json               ← MCP server configs for Claude Code (add to ~/.claude.json)
│   ├── opencode.json                  ← MCP server configs for OpenCode (merge into opencode.json)
│   ├── settings.json                  ← Claude Code main settings (→ ~/.claude/settings.json)
│   ├── settings.local.json            ← Claude Code local settings with hooks (→ ~/.claude/settings.local.json)
│   └── CLAUDE.md                      ← Research rules (append to CLAUDE.md or AGENTS.md)
└── hooks/
    └── allow.sh                       ← Auto-approve hook for MCP tools
```

## Customization

### Chrome Profile

The `playwright-logged-in` instance uses a dedicated Chrome profile. This means:
- Your logins persist between sessions (Reddit, Google, etc.)
- You avoid CAPTCHAs and rate limits
- Extensions work

Create the profile directory and log into sites you use frequently:
```bash
mkdir -p ~/.config/claude-chrome-profile
```

### Research Memory

The research skill learns over time. After each research session, it records what worked:
- Which sources gave the best results for a topic
- Which query patterns worked
- Which sites to avoid
- Navigation tips for specific sites

Memory files are stored in `~/.claude/skills/research/memory/`. The `_general.md` file contains cross-topic strategies. Topic-specific files (e.g., `ai-ml.md`, `linux-ubuntu.md`) are created automatically as you research.

---

## Installing in OpenCode

[OpenCode](https://opencode.ai) is an alternative AI coding CLI. It has native support for MCP servers, skills, and can even read Claude Code's `CLAUDE.md` and `~/.claude/skills/` as fallbacks. Here's how to set up the research system in OpenCode.

### Compatibility Notes

| Feature | Claude Code | OpenCode | Compatibility |
|---------|-------------|----------|---------------|
| Skills (`SKILL.md`) | `~/.claude/skills/` | `~/.config/opencode/skills/` | OpenCode reads both locations as fallback |
| System prompt | `CLAUDE.md` | `AGENTS.md` | OpenCode reads `CLAUDE.md` as fallback |
| MCP servers | `~/.claude.json` (`mcpServers`) | `opencode.json` (`mcp`) | Different format — needs translation |
| Hooks (auto-allow) | `~/.claude/hooks/` | Not supported | Use `permission` config instead |
| Settings (deny WebFetch) | `settings.json` | Not applicable | OpenCode doesn't have WebFetch |
| GitHub MCP | HTTP via Copilot endpoint | HTTP via Copilot endpoint | Same endpoint, different config format |

### Step 1: Install MCP Servers

Add or merge this into your `~/.config/opencode/opencode.json` (or project-level `opencode.json`):

```jsonc
// See config/opencode.json for the full config
{
  "$schema": "https://opencode.ai/config.json",
  "mcp": {
    "playwright-logged-in": {
      "type": "local",
      "command": [
        "npx", "-y", "@playwright/mcp@latest",
        "--user-data-dir=<PATH_TO_YOUR_CHROME_PROFILE>",
        "--isolated=false"
      ],
      "enabled": true
    },
    "playwright-fresh": {
      "type": "local",
      "command": ["npx", "-y", "@playwright/mcp@latest"],
      "enabled": true
    },
    "github": {
      "type": "http",
      "url": "https://api.githubcopilot.com/mcp/",
      "headers": {
        "Authorization": "Bearer <YOUR_GITHUB_PAT_TOKEN>"
      },
      "enabled": true
    }
  }
}
```

**Key differences from Claude Code:**
- `"type": "local"` instead of `"type": "stdio"` for Playwright (stdio commands)
- `"type": "http"` works the same way for the GitHub MCP endpoint
- `"command"` is an **array** (not a string + separate `"args"`)
- `--isolated=false` on `playwright-logged-in` ensures it reuses your Chrome profile properly

**Set your GitHub token:**
```bash
export GITHUB_PERSONAL_ACCESS_TOKEN="ghp_your_token_here"
# Add to ~/.bashrc or ~/.zshrc to persist
```

> **Note:** Older OpenCode guides may reference `@anthropic-ai/github-mcp-server` as a stdio-based GitHub MCP. This npm package can fail with 404 errors. The HTTP endpoint (`https://api.githubcopilot.com/mcp/`) is the recommended approach — it works for both Claude Code and OpenCode.

### Step 2: Install Skills

OpenCode reads skills from `~/.config/opencode/skills/` **and** falls back to `~/.claude/skills/`. You can either:

**Option A: Copy to OpenCode's native location (recommended)**
```bash
cp -r skills/* ~/.config/opencode/skills/
```

**Option B: Copy to Claude Code's location (works for both tools)**
```bash
cp -r skills/* ~/.claude/skills/
```

If you use both Claude Code and OpenCode, Option B lets both tools share the same skills.

### Step 3: Add System Instructions

OpenCode uses `AGENTS.md` for system instructions (but reads `CLAUDE.md` as fallback).

**Option A: Create an AGENTS.md (recommended for OpenCode-only setups)**
```bash
cat config/CLAUDE.md >> ~/.config/opencode/AGENTS.md
```

**Option B: Use CLAUDE.md (works for both tools)**
```bash
cat config/CLAUDE.md >> ~/.claude/CLAUDE.md
```

OpenCode reads `~/.claude/CLAUDE.md` as a fallback when `~/.config/opencode/AGENTS.md` doesn't exist. If you use both tools, Option B avoids maintaining two files.

### Step 4: Auto-Allow Skills (Replace Hooks)

OpenCode doesn't have Claude Code's hook system, but you can auto-allow skills via the `permission` config in `opencode.json`:

```json
{
  "permission": {
    "skill": {
      "*": "allow"
    }
  }
}
```

For MCP tools, OpenCode will prompt on first use. There's no hook-based auto-approval, but after you allow a tool once it remembers the permission for the session.

### Step 5: Create Chrome Profile

Same as Claude Code — create a dedicated Chrome profile directory:

```bash
mkdir -p ~/.config/opencode-chrome-profile
```

Then update the `--user-data-dir` path in your `opencode.json` to point to it.

### OpenCode-Specific Gotchas

1. **No WebFetch to block** — OpenCode doesn't have a built-in WebFetch tool, so the `deny: ["WebFetch"]` setting is irrelevant. The skills already use Playwright for all page reading.

2. **GitHub MCP uses same HTTP endpoint** — Both Claude Code and OpenCode now use the Copilot HTTP endpoint (`https://api.githubcopilot.com/mcp/`). The MCP tools exposed are the same (`mcp__github__search_issues`, etc.), so the skills work unchanged.

3. **Skill invocation** — In Claude Code you invoke skills with `/research` or via the `Skill` tool. In OpenCode, skills are loaded via the native `skill` tool the same way. The research skill's sub-skill loading (calling `Skill` tool for `google-search-browser`, `reddit-search`, etc.) works identically.

4. **Agent teams** — The research skill uses `Task` subagents for parallel page reads. OpenCode supports subagents/tasks, so this should work, but the exact behavior depends on your OpenCode version and model configuration.

5. **Fallback chain** — If you already have Claude Code installed with these skills, OpenCode will find them automatically via its fallback paths (`~/.claude/skills/`, `~/.claude/CLAUDE.md`). You may not need to copy anything.

---

## Notes

- **WebFetch is blocked** (Claude Code) — all page reading goes through Playwright browsers for reliability
- **GitHub CLI (`gh`) is avoided** — GitHub MCP tools are used instead for structured API access
- The auto-allow hook (Claude Code) means MCP tool calls won't prompt you for permission every time
- The research skill dynamically scales depth based on topic complexity — simple questions get quick answers, complex topics get deeper investigation
