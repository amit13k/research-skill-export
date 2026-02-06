# Research & Web Search Rules

## Core Rule
**The `research` skill is the default for ANY web search or info gathering.** Never use WebSearch or WebFetch directly — always use the research skill, even for simple "search for X" requests.

## Always Use Specialized Skills for Sub-Searches
- Reddit → use `reddit-search` skill via `Skill` tool
- Hacker News → use `hackernews-search` skill via `Skill` tool
- GitHub → use `github-search` skill via `Skill` tool
- **NEVER** spawn a generic Task agent to manually do `site:reddit.com` searches — use the dedicated skill instead

## Never Use WebFetch
- `browser-fetch` skill is the universal replacement for WebFetch
- All skills should use `playwright-logged-in` (or `playwright-fresh`) for URL reading

## DFS Link-Following
- Research must follow internal links on pages, not just read Google results at one level
- When a page links to deeper docs/reports/feature pages, follow those links (max 2 levels deep)
- Subagents should extract FOLLOW_LINKS from pages they read
- Total budget: 8 page reads (5 initial + 3 follow-up)

## Playwright MCP Configuration

Two Playwright MCP instances are configured:

### 1. `playwright-logged-in`
- Uses dedicated Chrome profile (maintains cookies, logins, extensions, and browser state)
- Use for: Testing logged-in experiences, accessing authenticated content, workflows requiring persistence
- This is the **default** for all browsing

### 2. `playwright-fresh`
- Starts with a completely fresh browser instance (no profile)
- No cookies, extensions, or saved state
- Use for: Testing new user experiences, incognito testing, avoiding cached data
