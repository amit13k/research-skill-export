---
name: browser-fetch
description: Fetch and read any URL using the browser instead of WebFetch. Use this whenever you need to read a web page, JSON API, or any URL content. This is the universal replacement for WebFetch — always prefer this skill over WebFetch. Triggers on any URL reading need.
user-invocable: false
---

# Browser Fetch

Read any URL using the Playwright browser. This is a drop-in replacement for WebFetch that uses real browser navigation, avoiding CAPTCHA issues, rate limits, and auth problems.

> **RULE: Always use this instead of WebFetch.** WebFetch should never be used. This skill provides the same functionality with better reliability.

## When to Use

- Any time you need to read a web page
- Reading JSON API endpoints (e.g., Algolia API, REST APIs)
- Reading documentation pages
- Extracting content from any URL
- Any situation where you would have used WebFetch

## Instructions

### Reading a Web Page

1. Navigate to the URL:
```
mcp__playwright-logged-in__browser_navigate: <url>
```

2. Capture the page content:
```
mcp__playwright-logged-in__browser_snapshot
```

3. If the page is long, scroll down and take additional snapshots to get full content.

### Reading a JSON API

For JSON endpoints (REST APIs, Algolia, etc.), parse the JSON directly:

1. Navigate to the API URL:
```
mcp__playwright-logged-in__browser_navigate: <url>
```

2. Extract and parse the JSON:
```
mcp__playwright-logged-in__browser_evaluate with function: () => {
  try { return JSON.parse(document.body.innerText); }
  catch { return document.body.innerText.substring(0, 10000); }
}
```

### Extracting Specific Content

To extract only specific content from a page (similar to WebFetch's prompt parameter):

1. Navigate to the URL
2. Use `browser_evaluate` to extract what you need:
```
mcp__playwright-logged-in__browser_evaluate with function: () => {
  return document.body.innerText.substring(0, 15000);
}
```

Then analyze the returned text for the information you need.

### Extracting Links from a Page

To discover internal links for DFS exploration:

```
mcp__playwright-logged-in__browser_evaluate with function: () => {
  const links = [...document.querySelectorAll('a[href]')];
  return links
    .map(a => ({ text: a.textContent?.trim(), href: a.href }))
    .filter(l => l.text && l.href && !l.href.startsWith('javascript:') && l.text.length > 3)
    .slice(0, 30);
}
```

## Browser Choice

- **Default: `playwright-logged-in`** — has cookies, avoids CAPTCHAs, handles rate limiting
- **Use `playwright-fresh`** only when you need a clean session (no cookies, no history)

## Notes

- This skill does NOT close the browser after use — the caller manages browser lifecycle
- For GitHub URLs, prefer GitHub MCP tools instead
- For multiple pages, use browser tabs for parallelism
