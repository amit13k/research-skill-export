---
name: google-search-browser
description: Search Google using browser automation. Use when the user asks to google something with browser, search google via playwright, or needs real browser-based google search results. Triggers on phrases like "google search with browser", "search google using browser", "browser google".
argument-hint: <search query>
---

# Google Search via Browser

Search Google using Playwright browser automation with efficient, minimal-data extraction.

## Instructions

When invoked with a search query, perform these steps:

### Step 1: Navigate directly to Google search results

**Default: Use `playwright-logged-in`** (better for rate limiting, cookies, and avoiding CAPTCHAs).

Use `mcp__playwright-logged-in__browser_navigate` with the query URL-encoded:
```
url: https://www.google.com/search?q=<query with spaces as +>
```

Example for "best keyboards 2024":
```
url: https://www.google.com/search?q=best+keyboards+2024
```

### Step 2: Extract organic results using JavaScript

Use `mcp__playwright-logged-in__browser_evaluate` to extract ONLY organic results (no ads):

```javascript
function: () => {
  const results = [];
  const h3s = document.querySelectorAll('h3');

  for (const h3 of h3s) {
    const link = h3.closest('a');
    if (!link?.href) continue;

    let url = link.href;
    // Skip ads and internal Google links by URL pattern
    if (url.includes('google.com/aclk') ||
        url.includes('googleadservices') ||
        url.includes('google.com/search') ||
        url.includes('accounts.google.com') ||
        url.includes('support.google.com') ||
        url.includes('maps.google.com')) continue;

    // Normalize URL: strip Google's text fragments to prevent duplicates
    url = url.split('#:~:text=')[0].split('#')[0];

    // Walk up to find the result container (element with >200 chars)
    let container = link;
    for (let i = 0; i < 8 && container; i++) {
      container = container.parentElement;
      if (container?.textContent?.length > 200) break;
    }

    // Find snippet in container's children (skip the title section)
    let snippet = '';
    if (container) {
      for (const child of container.children) {
        if (child.contains(h3)) continue;
        const text = child.textContent?.trim();
        if (text && text.length > 50 && !text.startsWith('http') && text.includes(' ')) {
          snippet = text
            .replace(/Read more$/, '')
            .replace(/^\d+:\d+/, '')  // Remove video duration at start
            .replace(/\d+ pages$/, '')  // Remove page count at end
            .trim();
          break;
        }
      }
    }

    results.push({
      title: h3.textContent?.trim(),
      url: url,
      snippet: snippet.slice(0, 250)
    });
  }

  // Dedupe by normalized URL
  const seen = new Set();
  return results.filter(r => {
    if (seen.has(r.url)) return false;
    seen.add(r.url);
    return true;
  }).slice(0, 10);
}
```

### Step 3: Present results

Format the results cleanly:

```
## Search Results for "query"

1. **Title**
   https://example.com/page
   Snippet text here...

2. **Title**
   ...
```

### Step 4: Close the browser

Use `mcp__playwright-logged-in__browser_close` to clean up.

## Browser Options

| Browser | When to Use |
|---------|-------------|
| `playwright-logged-in` | **Default.** Has cookies, less rate limiting, avoids CAPTCHAs |
| `playwright-fresh` | For incognito/clean searches, testing without personalization |

User can request fresh browser by saying "incognito" or "fresh" in their query.

## Why This Approach Works

**No hardcoded CSS classes** - Google changes classes frequently. Instead we use:
- `h3` elements for titles (semantic HTML, stable)
- `a.closest()` for links (DOM relationship, stable)
- URL patterns to filter ads (`aclk`, `googleadservices`)
- Text length heuristics for snippets (>50 chars, has spaces)

**Minimal data transfer** - Returns ~2KB JSON instead of ~50KB+ snapshot.

**Ad filtering** - Skips sponsored results by checking URL patterns.

**Duplicate prevention** - Strips `#:~:text=` fragments that Google adds.

## Example Usage

User: `/google-search-browser best mechanical keyboards 2024`

Returns top 10 organic Google results with title, URL, and snippet.

## Notes

- Uses `playwright-logged-in` by default for better rate limiting
- Automatically filters sponsored/ad results
- Handles special characters in queries (URL encoded)
- Skips Google Maps/Places results (those are buttons, not links)

## Reducing Context Usage

The `browser_navigate` tool returns a full page snapshot (~15-50k tokens). To minimize context:

1. **Don't parse the navigate snapshot** - Just navigate, ignore the returned snapshot
2. **Use `browser_evaluate`** - Extract only needed data via JS (returns ~500 tokens)
3. **Optional: Save snapshot to file** - If you need the full snapshot later:
   ```
   mcp__playwright-logged-in__browser_snapshot with filename: "search-results.md"
   ```
   Then read it with the Read tool only if needed.
