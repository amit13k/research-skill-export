---
name: hackernews-search
description: Search Hacker News for technical discussions, startup news, and developer community opinions. Uses Algolia HN Search API via browser automation. Use when the user asks to search Hacker News, find HN discussions, check what hackers think, or needs tech community perspectives. Triggers on phrases like "search hacker news", "HN discussions", "what does HN think", "hackernews", "search hn".
argument-hint: <search query>
---

# Hacker News Search

Search Hacker News using Algolia's HN Search API via browser automation.

## What I Do

- Search Hacker News stories and comments via Algolia's search API
- Extract post titles, scores, comment counts, and URLs
- Read top discussions for detailed community insights
- Filter by date range and sort by relevance or date

## When to Use Me

- "Search Hacker News for X"
- "What does HN think about X?"
- "Find HN discussions about X"
- "Check Hacker News for X"
- Any query about tech topics where developer community opinion matters

## Search Workflow

### Step 1: Search via Algolia HN Search

Use `playwright-fresh` browser (no auth needed).

Navigate to:
```
https://hn.algolia.com/?q=<query+encoded>&sort=byDate&dateRange=pastMonth&type=story
```

**Sort options:**
- `byPopularity` - Most points first (good for finding best discussions)
- `byDate` - Most recent first (good for breaking news)

**Date range options:**
- `pastMonth` - Last 30 days (default)
- `pastWeek` - Last 7 days
- `past24h` - Last 24 hours
- `pastYear` - Last year
- `all` - All time

**Type options:**
- `story` - Posts/links (default)
- `comment` - Comments only
- `all` - Both stories and comments

### Step 2: Extract Results

Use `mcp__playwright-fresh__browser_snapshot` to capture the search results page.

From the snapshot, extract:
- Post titles and URLs
- Points (score)
- Number of comments
- Post age
- Author

### Step 3: Read Top Discussions (if needed)

For the most relevant posts (highest score + most comments), read the HN discussion:

**Option A: Direct HN page (browser snapshot)**
Navigate to the HN thread and take a snapshot:
```
mcp__playwright-fresh__browser_navigate: https://news.ycombinator.com/item?id=<item_id>
mcp__playwright-fresh__browser_snapshot
```

**Option B: Algolia API via browser (JSON, more structured)**
Navigate to the Algolia API endpoint in the browser and extract JSON:
```
mcp__playwright-fresh__browser_navigate: https://hn.algolia.com/api/v1/items/<item_id>
mcp__playwright-fresh__browser_evaluate with function: () => JSON.parse(document.body.innerText)
```
This returns structured JSON with all comments without using WebFetch.

> **IMPORTANT: NEVER use WebFetch.** Always use the browser (`playwright-fresh`) for all HN page reading and API access.

### Step 4: Present Results

```
## Hacker News: [Query]

### Top Discussions

1. **[Title](HN link)** - X points, Y comments
   - Key insight from discussion
   - Notable comment/opinion

2. **[Title](HN link)** - X points, Y comments
   - Key insight from discussion

### Community Sentiment
[Brief summary of what HN users think about the topic]

### Sources
- [Post Title](URL) - X points, Y comments
- ...
```

## Quality Filters

Apply these thresholds:
- **Minimum:** Score >= 5 AND Comments >= 3
- **Preferred:** Score >= 20 AND Comments >= 10
- **High quality:** Score >= 100 AND Comments >= 50

Skip posts below minimum threshold unless they're the only results.

## Efficiency Rules

- Take a snapshot of search results first, don't read every discussion
- Only fetch full comments for 1-2 most relevant threads
- Use Algolia API (JSON) instead of HTML scraping when possible
- Stop when you have 3-5 good results that answer the query

## Hard Limits

- Max 10 posts in search results
- Max 2 threads for full comment reading
- Max 5 sources in final output
