---
name: github-search
description: Intelligent multi-step GitHub issue and discussion search. Strategically uses Google search first (often one-shots the right issue), then uses GitHub MCP server tools for structured searching of issues, discussions, and repositories with sorting and filtering. Combines Google site:github.com with MCP tools (search_issues, list_discussions, search_repositories) to find relevant issues, PRs, and discussions. Asks user for clarification when queries are unclear.
---

## What I do
- **Strategically use Google search first** to find relevant GitHub issues/discussions (often one-shots the exact issue)
- **Use GitHub MCP server tools** for structured searching with sorting, filtering, and date ranges
- **Search GitHub Discussions** via MCP `list_discussions` and `get_discussion` tools (no browser needed)
- **Find the right repository** via MCP `search_repositories` when the repo is unknown
- **Analyze query clarity** and refine keywords when ambiguous
- **Ask user for clarification** when queries are completely unclear
- **Combine Google + MCP tools** to find the most relevant issues, PRs, and discussions
- **Extract key information** from issues: status, labels, workarounds, root cause, linked PRs
- Return concise findings with issue numbers, status, and actionable information

## When to use me
Use this skill when the user asks to:
- Search GitHub for issues about a bug or feature
- Find if a problem has been reported on a repo
- Look up GitHub discussions about a topic
- Check the status of a known issue
- Find workarounds for a bug from GitHub issues
- Search across multiple repos for a common problem
- Any query like "search GitHub for...", "is there an issue for...", "has anyone reported..."

## Available Tools

### Primary: Google Search (via WebSearch)
Best for: Finding the exact issue quickly, searching comments/body text, discovering which repo has the issue.

### GitHub MCP Server Tools (via `mcp__github__*`)
The GitHub MCP server provides these key tools:

**Search & Discovery:**
- `mcp__github__search_issues` — Search issues with `query` (GitHub search syntax), `sort`, `order`, `owner`, `repo`
- `mcp__github__search_repositories` — Find repos with `query` (supports `stars:>1000`, `topic:react`, etc.), `sort`, `order`
- `mcp__github__search_code` — Search code across repos

**Issues:**
- `mcp__github__get_issue` — Get a specific issue by number (owner, repo, issue_number)
- `mcp__github__list_issues` — List issues in a repo with filtering

**Discussions (the key advantage over gh CLI):**
- `mcp__github__list_discussions` — List discussions with `owner`, `repo`, `category`, `orderBy`, `direction`
- `mcp__github__get_discussion` — Get a specific discussion by number
- `mcp__github__get_discussion_comments` — Get comments on a discussion
- `mcp__github__list_discussion_categories` — List available categories in a repo

**Pull Requests:**
- `mcp__github__get_pull_request` — Get PR details

### Fallback: `gh` CLI
Use `gh` CLI only when MCP tools don't support a specific query pattern (e.g., complex date range filters).

## Quick Decision Tree

```
User Query
    |
Is the repo known?
    |
YES --> Is it a specific bug/error?
    |       |
    |   YES --> Google: "error message" site:github.com/org/repo
    |       |
    |   NO  --> Google: keywords site:github.com/org/repo
    |
NO  --> Google: keywords site:github.com
    |   + MCP: search_repositories to identify the right repo
    |
Google found the answer?
    |
YES --> Verify with MCP get_issue/get_discussion if needed, STOP
    |
NO  --> MCP: search_issues + list_discussions (parallel)
    |
Found now?
    |
YES --> MCP: get_issue/get_discussion for details, STOP
    |
NO  --> Refine keywords, try once more, then give up
```

## Search Strategy: Google First, MCP Second

**Google search is the PRIMARY discovery tool.** It frequently one-shots the exact issue. MCP tools are the PRIMARY structured search and data extraction tools.

### Why Google First?
- Google indexes issue comments, discussion replies, and linked content
- Google understands synonyms and related terms better
- A single Google query often finds the exact issue immediately
- Google surfaces closed/resolved issues that GitHub search buries

### Why MCP Second (not gh CLI)?
- MCP `list_discussions` can search and sort discussions (gh CLI cannot)
- MCP `get_discussion_comments` reads discussion replies without a browser
- MCP `search_issues` supports GitHub search syntax with sort/order
- MCP `search_repositories` finds the right repo when unknown
- No browser overhead — fast structured API calls

## Step-by-Step Search Process

### Step 0: Query Analysis

**Assess the query before searching:**

1. **Is a specific repo known?** (e.g., "ghostty ctrl+c issue" = ghostty-org/ghostty)
2. **Is there an error message?** (exact error strings are gold for Google)
3. **Is the query about a bug, feature request, or general question?**
4. **Is the repo unknown?** Use MCP `search_repositories` to find it

**Query Clarity Levels:**

| Level | Example | Action |
|-------|---------|--------|
| Clear + repo known | "ghostty ctrl+c fish shell bug" | Google site:github.com/org/repo directly |
| Clear + repo unknown | "neovim treesitter highlight broken" | Google site:github.com + MCP search_repositories |
| Ambiguous | "terminal freeze linux" | Google broadly, then narrow to repos |
| Completely unclear | "it's broken" | Ask user for clarification |

### Step 0.5: Find the Repository (when unknown)

**If the user mentions a project name but not the org/repo:**

Use MCP to find and verify the correct repository:
```
mcp__github__search_repositories(query="ghostty terminal emulator", sort="stars", order="desc")
```

This returns repo metadata including stars, description, and owner — use it to verify you have the right open source repo before searching issues.

**Verification signals:**
- High star count = likely the official repo
- Description matches the project
- Owner matches known org (e.g., ghostty-org, facebook, vercel)
- Recent activity (not archived/abandoned)

### Step 1: Google Search (Primary - Often Sufficient)

**This is your most powerful discovery tool. Use it first, always.**

#### URL Patterns

**When repo is known:**
```
https://www.google.com/search?q={keywords}+site:github.com/{org}/{repo}&tbs=qdr:{time}
```

**When repo is unknown:**
```
https://www.google.com/search?q={keywords}+site:github.com&tbs=qdr:{time}
```

**For exact error messages:**
```
https://www.google.com/search?q="{exact error message}"+site:github.com/{org}/{repo}
```

#### Google Search Tips for GitHub
- **Quote exact error messages** - this is the #1 way to find the right issue
- **Include "issue" or "discussion"** to bias toward those pages
- **Add the year** for recent issues: "2025" or "2026"
- **Use OR** for alternate terms: "ctrl+c OR ctrl-c OR SIGINT"
- **Use minus** to exclude noise: "crash -bot -stale"
- Time filters: `qdr:w` (week), `qdr:m` (month), `qdr:y` (year)

#### Google Search Variations (try in order, stop when found)
1. **Exact match:** `"{error message}" site:github.com/org/repo`
2. **Keywords + repo:** `keyword1 keyword2 site:github.com/org/repo`
3. **Keywords + broader:** `keyword1 keyword2 site:github.com`
4. **With issue/discussion hint:** `keyword1 keyword2 issue OR discussion site:github.com/org/repo`

**Early Termination:**
- If Google returns an issue/discussion that matches, verify with MCP `get_issue`/`get_discussion`, then STOP
- If the top 3 Google results are relevant, extract info and STOP
- Only proceed to Step 2 if Google yields nothing useful

### Step 2: MCP Structured Search (When Google Isn't Enough)

**Use MCP tools in parallel for maximum efficiency.**

#### Search Issues
```
mcp__github__search_issues(
  query="ctrl+c fish shell",
  owner="ghostty-org",
  repo="ghostty",
  sort="created",
  order="desc"
)
```

**GitHub search syntax in the `query` parameter:**
- `is:issue` or `is:pr` — filter type
- `is:open` or `is:closed` — filter state
- `label:bug` — filter by label
- `in:title` or `in:body` — where to search
- `created:>2025-01-01` — date filters
- `reactions:>5` — engagement filters
- `comments:>10` — discussion depth

**Sort options:** `created`, `updated`, `comments`, `reactions-+1`, `interactions`, `best-match`
**Order options:** `asc`, `desc`

#### Search Discussions (MCP advantage — gh CLI can't do this)
```
mcp__github__list_discussions(
  owner="ghostty-org",
  repo="ghostty",
  orderBy="CREATED_AT",
  direction="DESC"
)
```

Then filter results by scanning titles for keyword matches.

**To search a specific category:**
1. First get categories: `mcp__github__list_discussion_categories(owner, repo)`
2. Then filter: `mcp__github__list_discussions(owner, repo, category="CATEGORY_ID")`

#### Search Repositories (when repo is unknown)
```
mcp__github__search_repositories(
  query="ghostty terminal emulator",
  sort="stars",
  order="desc"
)
```

### Step 3: Deep Dive (Only When Needed)

**Only do this for the 1-3 most relevant issues/discussions found.**

#### For Issues
```
mcp__github__get_issue(owner="ghostty-org", repo="ghostty", issue_number=10544)
```

#### For Discussions
```
mcp__github__get_discussion(owner="ghostty-org", repo="ghostty", discussionNumber=10541)
```

#### For Discussion Comments (workarounds often live here)
```
mcp__github__get_discussion_comments(owner="ghostty-org", repo="ghostty", discussionNumber=10541)
```

#### For linked PRs
```
mcp__github__get_pull_request(owner="ghostty-org", repo="ghostty", pullNumber=10455)
```

**Extract:**
- Root cause (if identified)
- Workaround (if any)
- Fix status (PR merged? Release version?)
- Related issues

### Step 4: Fallback to `gh` CLI (Rare)

Only use `gh` CLI when MCP tools don't support the query pattern:

```bash
# Complex date range + label + sort combination
gh search issues "keywords" --repo org/repo --label "bug" --created ">2025-01-01" --sort reactions-+1 --order desc

# View issue with full comment thread (if MCP truncates)
gh issue view NUMBER --repo org/repo --comments
```

## Quality Filters

### Default Issue Quality Filter
```
MINIMUM: Reactions >= 1 OR Comments >= 2
PREFERRED: Reactions >= 5 OR Comments >= 5
HIGH QUALITY: Reactions >= 10 OR Comments >= 10
```

- Skip bot-generated issues (dependabot, renovate, stale bot)
- Skip issues with only "me too" comments
- Prefer issues with maintainer responses
- Prefer issues with linked PRs (indicates a fix exists)

### Relevance Signals (Strongest to Weakest)
1. Exact error message match in issue title/body
2. Issue has a "bug" or "confirmed" label
3. Maintainer has commented or labeled the issue
4. Issue has a linked PR (fix in progress or merged)
5. High reaction count (many users affected)
6. Recent activity (updated within last month)
7. Multiple users reporting same symptoms in comments

## Output Format

### Quick Answer (when Google one-shots it)
```
## Found: [Issue/Discussion Title] (#NUMBER)
**Status:** Open/Closed | **Labels:** bug, confirmed
**Repo:** org/repo

**Summary:** [1-2 sentence summary of the issue]

**Workaround:** [if any]
**Fix:** [PR #NUMBER merged/pending, or "no fix yet"]

**Link:** https://github.com/org/repo/issues/NUMBER
```

### Multiple Results
```
## Results for "[query]"

### Most Relevant
1. **[Title]** - org/repo#NUMBER (Open, 15 reactions, 23 comments)
   - [1 sentence summary]
   - Workaround: [if any]

2. **[Title]** - org/repo#NUMBER (Closed, fixed in v1.2.3)
   - [1 sentence summary]

### Related
3. **[Title]** - org/repo#NUMBER (Open)
   - [1 sentence summary]
```

### No Results Found
```
## No matching issues found for "[query]"

**Searched:**
- Google: site:github.com/org/repo
- MCP: search_issues, list_discussions
- gh CLI: (if used)

**Suggestions:**
- Try different keywords: [suggestions]
- Check if the project uses a different issue tracker
- Consider opening a new issue
```

## Keyword Intelligence

### GitHub-Specific Keyword Tips
- **Use exact error messages** in quotes - most effective search strategy
- **Include component names** - "shell-integration", "kitty-protocol", "wayland"
- **Include version numbers** - "v1.3.0", "1.2.1", "tip"
- **Use GitHub search qualifiers in MCP query** - "is:issue is:open label:bug"
- **OS-specific terms** - "linux", "macos", "windows", "wayland", "x11"

### Keyword Refinement During Search
If results are poor:
1. **Too broad:** "crash" -> "crash wayland resize" (add context)
2. **Too specific:** "SIGINT CSI-u 99;5u fish 4.0.2 ghostty tip" -> "ctrl+c fish ghostty" (simplify)
3. **Wrong terminology:** "terminal freeze" -> "terminal hang OR unresponsive" (use OR for synonyms)
4. **Missing context:** "not working" -> "ctrl+c not working fish shell" (add specifics)

## Anti-Patterns to Avoid

- **Don't** skip Google search and go straight to MCP/gh CLI
  - **Do** always start with Google - it's the most effective discovery tool for GitHub content

- **Don't** search only Issues when the project uses Discussions
  - **Do** use MCP `list_discussions` to search Discussions (no browser needed)

- **Don't** use vague keywords in MCP `search_issues`
  - **Do** use GitHub search syntax: "is:issue is:open label:bug ctrl+c fish"

- **Don't** read every issue in full
  - **Do** scan titles and metadata first, deep-dive with `get_issue`/`get_discussion` only for 1-3 most relevant

- **Don't** ignore closed issues
  - **Do** search closed issues too - they often contain the fix/workaround

- **Don't** give up after one search query
  - **Do** try 2-3 keyword variations, but stop after finding relevant results

- **Don't** forget to check linked PRs
  - **Do** use `get_pull_request` to check if a fix exists and its status

- **Don't** use browser Playwright for Discussions
  - **Do** use MCP `list_discussions` + `get_discussion_comments` instead (faster, no browser overhead)

- **Don't** include exhaustive search methodology in output
  - **Do** provide concise results with issue numbers, status, and key findings

- **Don't** guess the repo org/name
  - **Do** use MCP `search_repositories` to find and verify the correct repo first

## Hard Limits
- Max 3 Google search queries
- Max 2 MCP `search_issues` calls
- Max 1 MCP `list_discussions` call
- Max 3 issues/discussions for full detail extraction
- Max 10 total issues in final output
- If Google finds the answer in query 1, STOP immediately

## Common Repo Patterns

Some repos have quirks that affect search strategy:

| Pattern | Example | Strategy |
|---------|---------|----------|
| Issues disabled, Discussions only | ghostty-org/ghostty | MCP `list_discussions` + `get_discussion` |
| Monorepo with labels | facebook/react | Add `label:` filter in search_issues query |
| Uses "Issue Triage" category | ghostty-org/ghostty | MCP `list_discussion_categories` then filter |
| Linked issue tracker | Some corps | Google may find external tracker links |
| Uses milestones | Many projects | Filter by milestone for version-specific bugs |

## Example Workflows

### Example 1: Known repo, specific bug (Google one-shots)
**Query:** "ghostty ctrl+c not working fish shell"

**Step 1 - Google:**
```
WebSearch: "ghostty ctrl+c fish shell site:github.com/ghostty-org/ghostty"
```
**Result:** Google directly surfaces Discussion #10541 "fish shell: CTRL-C not clearing prompt input"

**Verify with MCP:**
```
mcp__github__get_discussion(owner="ghostty-org", repo="ghostty", discussionNumber=10541)
```
**Action:** Extract workaround from discussion, STOP.

### Example 2: Known repo, need structured search
**Query:** "react hydration mismatch"

**Step 1 - Google:**
```
WebSearch: "react hydration mismatch site:github.com/facebook/react"
```
**Result:** Multiple issues found, need to find the most relevant.

**Step 2 - MCP structured search:**
```
mcp__github__search_issues(
  query="hydration mismatch is:issue is:open",
  owner="facebook",
  repo="react",
  sort="reactions-+1",
  order="desc"
)
```
**Result:** Top issue with 200+ reactions found. Deep dive, STOP.

### Example 3: Unknown repo
**Query:** "ghostty terminal emulator ctrl+c broken"

**Step 0.5 - Find repo:**
```
mcp__github__search_repositories(query="ghostty terminal emulator", sort="stars", order="desc")
```
**Result:** ghostty-org/ghostty (42.8k stars) — confirmed as the right repo.

**Step 1 - Google with confirmed repo:**
```
WebSearch: "ctrl+c broken site:github.com/ghostty-org/ghostty"
```

### Example 4: Project uses Discussions (MCP replaces browser)
**Query:** "ghostty window resize flicker linux"

**Step 1 - Google:**
```
WebSearch: "ghostty window resize flicker linux site:github.com/ghostty-org/ghostty"
```
**Result:** Links to Discussion, not Issue.

**Step 2 - MCP Discussion search:**
```
mcp__github__list_discussions(owner="ghostty-org", repo="ghostty", orderBy="CREATED_AT", direction="DESC")
```
Scan titles for "resize" / "flicker".

**Step 3 - Get discussion details:**
```
mcp__github__get_discussion(owner="ghostty-org", repo="ghostty", discussionNumber=XXXX)
mcp__github__get_discussion_comments(owner="ghostty-org", repo="ghostty", discussionNumber=XXXX)
```
**Action:** Extract answer and workaround, STOP.
