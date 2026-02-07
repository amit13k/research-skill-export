# Research & Web Search Rules

## Core Rule
**The `research` skill is the default for ANY web search or info gathering.** Never use WebSearch or WebFetch directly — always use the research skill, even for simple "search for X" requests.

## Search Before Guessing Rule
**If you are not CONFIDENT in your answer, use the `research` skill BEFORE responding.** This applies to:
- Any question about events, releases, or changes after your knowledge cutoff
- Current pricing, availability, status, or compatibility questions
- Topics where you'd hedge with "I think", "I believe", "probably", "might"
- Niche tools, libraries, configs, or options you don't know deeply
- "Does X support Y?", "Can X do Y?", "What's the right setting for Z?" — correctness-critical questions
- Version-specific behavior, API changes, or deprecated features
- Any question where saying "I'm not sure" or "I don't have info on that" would be the alternative

**NEVER say "I don't know" or "I'm not sure" without searching first.** The research skill exists precisely for these moments.

## Always Use Specialized Skills for Sub-Searches
- Reddit → use `reddit-search` skill via `Skill` tool
- Hacker News → use `hackernews-search` skill via `Skill` tool
- GitHub → use `github-search` skill via `Skill` tool
- **NEVER** spawn a generic Task agent to manually do `site:reddit.com` searches — use the dedicated skill instead

## Never Use WebFetch
- `browser-fetch` skill is the universal replacement for WebFetch
- All skills should use `playwright-logged-in` (or `playwright-fresh`) for URL reading

## Best-First Search Link-Following
- Research uses Best-First Search with a Promise Score heuristic — NOT simple depth-first or breadth-first
- Every discovered URL is scored (relevance, credibility, novelty, context) and the highest-scored unread URL is always read next
- No fixed depth limit — depth emerges from scoring and diminishing returns detection
- Subagents should extract DISCOVERED_LINKS from pages they read
- Stopping conditions: diminishing returns, queue starved, confident answer (verified via Answer-Completeness Gate), or saturation

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

### Context Management: ALWAYS Use Subagents for Browser Operations

**ABSOLUTE RULE — NO EXCEPTIONS: NEVER call ANY Playwright MCP tool directly in the main context.**

This includes `browser_navigate`, `browser_snapshot`, `browser_click`, `browser_fill_form`, and ALL other `playwright-logged-in` / `playwright-fresh` tools. Every single one returns huge accessibility snapshots (~10-15k tokens) that bloat the main conversation window.

**What to do instead:** ALWAYS delegate to a Task subagent:
1. Spawn a Task agent with the browser instructions
2. The subagent calls the Playwright tools, processes the snapshot internally
3. The subagent returns ONLY a compact summary (relevant text, links, confirmation)
4. The huge snapshot stays in the subagent's context, never touches main
