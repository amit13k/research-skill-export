---
name: research
description: Deep research on any topic by orchestrating multiple sources - Google search (browser), Reddit, Hacker News, GitHub, official/community forums, and direct web page reading. Use for ANY task that requires searching the web, gathering information, comparing products/tools, finding alternatives, checking reviews, looking up documentation, or answering questions that need external information. Triggers on phrases like "research", "find out about", "investigate", "look into", "what's the latest on", "deep dive into", "compare", "search for", "find the best", "what is", "how does X work", "check out", "explore options", "alternatives to", "reviews of", "search skills.sh", "find on npm", "look up", "tell me about", "what do people think", "is X good", "pros and cons".
argument-hint: <research topic or question>
---

# Research Skill

Lightweight orchestrator for comprehensive multi-source research. Delegates to specialized skills for each source.

> **CRITICAL RULE: NEVER use WebFetch.** Always use the browser (`playwright-logged-in`) for ALL page reading — searching, navigating, extracting content. The only exception is if the user explicitly asks you to use WebFetch.

> **GITHUB RULE: ALWAYS use GitHub MCP tools for ALL GitHub data.** NEVER use `gh` CLI via Bash, and NEVER use the browser to navigate to GitHub URLs.

> **OFFICIAL SOURCE FIRST RULE: ALWAYS identify and read the official source BEFORE any third-party coverage.** For any product/tool/announcement being researched, your FIRST page read MUST be the official website, docs, blog post, or paper — never a VentureBeat/TechCrunch/Medium summary. If you don't know the official source, your first action after Google search results is to find it (look for the company's own domain in results, or do a targeted `site:<company-domain>` search). Third-party articles are only read AFTER the official source.

> **PRACTICALITY RULE (for "easiest/best/popular" queries): OFFICIAL + POPULAR.** After reading official docs, you MUST also search for the most adopted real-world path (integrations, plugins, wrappers, dashboards, community-standard tooling). Do not stop at the official method if the user asked for easiest/common/popular workflow.

> **INTENT CLARIFICATION FIRST RULE (MANDATORY):** Before running any web search, ask a targeted clarifying question to lock scope and provider/context. Do not start external searches until the user answers. Exception: skip only when scope is already explicit and singular (e.g., user names one exact product + account type + environment).

> **RE-CLARIFICATION RULE (MANDATORY):** Clarification is not one-and-done. If new evidence mid-research creates ambiguity, conflicts with user assumptions, or opens multiple valid interpretations, pause and ask another targeted question before continuing. Ask as many clarification rounds as needed to keep scope correct.

> **DISCOVERY-THEN-CLARIFY RULE (MANDATORY):** If the query is too unknown to ask a meaningful clarifying question at the start, do a tiny reconnaissance pass (usually 1-2 high-signal reads) only to learn the landscape, then stop and ask a scoped clarification question before deeper research.

## What I Do

- **Orchestrate** multiple search sources in parallel for speed
- **Delegate** to specialized skills for each source (Google, Reddit, HN, GitHub)
- **Read and extract** key information from the most relevant pages
- **Intelligent link exploration (Best-First Search)** — discovered URLs are scored by a Promise Score heuristic (relevance, credibility, novelty, context). Always reads the most promising unread link next, regardless of depth. No fixed depth limit — depth emerges from scoring and diminishing returns detection
- **Surface practical paths** — include popular community solutions and ecosystem integrations when users ask for easiest/common approaches
- **Synthesize** findings into a clear, sourced summary
- **Adapt search strategy** based on topic type

## When to Use Me

- "Research X" or "Find out about X"
- "What's the latest on X?"
- "Look into X for me"
- "Investigate X"
- "Deep dive into X"
- "Compare X to Y" or "X vs Y"
- "Search for X" or "Find the best X"
- "What is X?" or "How does X work?"
- "Check out X" or "Look up X"
- "Alternatives to X" or "Reviews of X"
- "Is X good?" or "Pros and cons of X"
- "What do people think about X?"
- Any question requiring web search or external information

## Skill Composition

This skill orchestrates these individual skills. **Load them dynamically** as needed using the `Skill` tool:

| Source | Skill to Load | When to Use |
|--------|--------------|-------------|
| Google | `google-search-browser` | Always (primary discovery) |
| Reddit | `reddit-search` | Opinions, experiences, community discussions |
| Hacker News | `hackernews-search` | Technical topics, startup news, dev opinions |
| GitHub | `github-search` | Developer tools, bugs, open source projects |
| Official Forums | *(inline below)* | Troubleshooting, product/service issues |

**How to load:** After classifying the topic (Step 0) and selecting sources (Step 1), call the `Skill` tool for each needed skill before executing searches. Each skill contains its own search patterns, scripts, and best practices.

## Research Workflow

### Step -2: Clarify Intent First (MANDATORY)

Before any search, ask one concise question that resolves the highest-impact ambiguity.

**Required for these query types:**
- usage, limits, quota, billing, pricing, "how much used"
- provider-ambiguous product names (e.g., Codex could mean ChatGPT plan Codex, OpenAI API Codex usage, or third-party routed usage)
- "easiest way" requests where environment/account choice changes the best answer
- unknown/novel topics where initial wording is broad and could map to multiple ecosystems

**Question design rules (per round):**
1. Ask exactly one question at a time.
2. Offer 2-4 concrete options plus custom input.
3. Put the recommended default first.
4. State what changes based on the answer.
5. If ambiguity remains after the answer, ask the next highest-impact question.

**Codex example (must use this pattern):**
"Which Codex usage do you want to track? (A) ChatGPT plan usage, (B) OpenAI API usage, (C) OpenRouter/provider-routed usage. This changes the dashboard and commands I should use."

If user does not answer, proceed with the clearly stated default and label assumptions explicitly.

If you cannot form concrete options yet, run Step -2.5 first.

### Step -2.5: Minimal Reconnaissance for Better Questions (ONLY when needed)

Use only when you genuinely cannot ask a meaningful clarification question yet.

Rules:
1. Read at most 1-2 high-signal sources (official landing/docs first).
2. Extract candidate interpretations (product type, provider, account model, environment constraints).
3. Stop reconnaissance immediately.
4. Ask a clarifying question with concrete options based on what you learned.
5. Continue full research only after clarification (or explicit default assumption).

### Step -1.5: Clarification Loop During Research (MANDATORY)

Run this check after each wave and before final synthesis.

Trigger a re-clarification question if any of these occur:
- New evidence indicates a different provider/account context than assumed
- Multiple products share the same name (or overlapping branding)
- User asked for "easiest" but evidence splits by environment/stack
- Findings conflict across sources and the right answer depends on user preference (speed vs cost vs control)
- You cannot produce one unambiguous recommendation without guessing
- Early exploration changed your mental model of what the user might mean
- New terminology appears that implies different intent than the original phrasing

When triggered:
1. Pause new searches.
2. Ask one targeted question that unlocks the decision.
3. Resume searches only after answer (or explicit default assumption).

### Step -1: Load Research Memory

Before anything else, check for learned strategies from previous research sessions.

1. **Glob** the memory directory: `~/.claude/skills/research/memory/*.md` (**NEVER modify SKILL.md or any file outside the `memory/` subdirectory**)
2. **Always read** `_general.md` (cross-topic patterns, kept small)
3. **Scan filenames** of other files (e.g., `ai-ml.md`, `linux-ubuntu.md`, `web-dev.md`)
4. **Pick 0-2 topic files** whose names match the research question — the LLM matches naturally (e.g., "fix Bluetooth on Ubuntu" → `linux-ubuntu.md`)
5. **Read the selected files** — they contain learned strategies: which sources work, which sites to avoid, query patterns that helped, navigation tips
6. **Apply these strategies** throughout the research session — they influence source selection, query generation, and site priorities

**If no memory files match the query topic, skip this step.** Memory is a boost, not a requirement. The skill works fine without it — memory just makes it faster over time.

### Step 0: Classify the Topic & Select Sources

Before searching, classify the query to determine which skills to load.

| Topic Type | Primary Sources | Secondary Sources |
|-----------|----------------|-------------------|
| **Tech/Dev** (new API, tool, framework) | Google, GitHub, Hacker News, **Official Forum** | Reddit |
| **Product/Service** (reviews, pricing, issues) | Google, Reddit, **Official Forum** | Hacker News |
| **News/Announcement** (just released) | Google, Reddit, Hacker News | Direct page reads, Official Forum |
| **Opinion/Experience** (is X worth it) | Reddit, Hacker News | Google, Official Forum |
| **How-to/Tutorial** (enable X, configure Y) | Google, GitHub, **Official Forum** | Reddit |
| **Troubleshooting** (X not working) | Google, **Official Forum**, GitHub, Reddit | Stack Overflow via Google |

#### Recency Assessment

Also assess how time-sensitive the topic is. This determines search filtering and source priority.

| Recency Level | Signals | Action |
|--------------|---------|--------|
| **Breaking** | "just announced", "today", "new release" | Use Google `tbs=qdr:d` (past 24h), prioritize news sites and official blogs |
| **Recent** | "latest", "current", "2025/2026", version numbers | Use Google `tbs=qdr:m` (past month) for at least one search pass |
| **Evolving** | Pricing, comparisons, "best X", "is X worth it" | Use Google `tbs=qdr:y` (past year) — older info is likely outdated |
| **Evergreen** | Concepts, algorithms, "how does X work" | No time filter needed, but still prefer recent sources when quality is equal |

**Default to "Evolving"** if unsure — most research questions benefit from recency bias. Only skip time filtering for truly timeless topics.

#### Intent Lens (MANDATORY)

Before searching, classify user intent keywords and enforce matching output requirements:

- **Ease intent** (`easiest`, `simplest`, `quickest`, `fastest`): return a default path with the fewest steps and lowest setup overhead.
- **Adoption intent** (`popular`, `common`, `most used`, `recommended`): include popularity evidence (stars, active maintenance, repeated mentions, docs maturity).
- **Ecosystem intent** (`integration`, `plugin`, `extension`, `connector`, `works with`): explicitly search integration/plugin surfaces, not just core docs.

If any of these intents are present, the final answer MUST include:
1. one official path,
2. one popular ecosystem path,
3. a short recommendation of which to pick first and why.

If Step -2 clarification has not been completed for an ambiguous query, do not execute Step 1 searches yet.
If Step -1.5 triggers during research, pause Step 2 and re-clarify before continuing.

#### Identifying the Official Forum

For topics about a specific product/service, identify the official community forum:

| Product/Company | Official Forum |
|----------------|---------------|
| ChatGPT / OpenAI | `community.openai.com` |
| Claude / Anthropic | `community.anthropic.com` |
| Google / Android | `support.google.com/community`, `issuetracker.google.com` |
| Apple / macOS / iOS | `discussions.apple.com`, `developer.apple.com/forums` |
| Microsoft / Windows | `answers.microsoft.com`, `learn.microsoft.com/answers` |
| Ubuntu / Linux | `askubuntu.com`, `discourse.ubuntu.com` |
| Docker | `forums.docker.com` |
| Vercel / Next.js | `github.com/vercel/next.js/discussions` |
| VS Code | `github.com/microsoft/vscode/issues` |
| Stripe | `support.stripe.com`, `github.com/stripe` |
| AWS | `repost.aws` |
| Cloudflare | `community.cloudflare.com` |

If you don't know the official forum, the first Google search will often surface it.

### Step 0.5: Identify Official Sources (MANDATORY)

Before reading ANY search results, identify the **official source** for each product/tool/company in the research query. This is non-negotiable.

**How to identify official sources:**
1. **Known products**: Use the company's own domain (e.g., `anthropic.com/blog` for Claude, `moonshot.ai` or `arxiv.org` for Kimi K2.5, `openai.com/blog` for GPT)
2. **Unknown products**: Scan Google results for the company's own domain. If not visible, do a quick search: `<product-name> official site` or `<product-name> announcement site:<company-domain>`
3. **Open-source projects**: The GitHub repo README, release notes, or the project's official docs site
4. **Academic/research**: The arXiv paper or official research blog post

**Official source examples by company:**

| Company | Official Sources |
|---------|-----------------|
| Anthropic (Claude) | `anthropic.com/blog`, `docs.anthropic.com`, `anthropic.com/research` |
| OpenAI (ChatGPT/GPT) | `openai.com/blog`, `platform.openai.com/docs` |
| Google (Gemini) | `blog.google`, `ai.google.dev`, `deepmind.google` |
| Moonshot AI (Kimi) | `moonshot.ai`, `kimi.ai`, arXiv papers |
| Meta (Llama) | `ai.meta.com/blog`, `llama.meta.com` |
| Mistral | `mistral.ai/news` |

**Record the official URLs** you find — these become your FIRST page reads in Step 2.

### Step 0.8: Generate Diverse Queries (Vocabulary Expansion)

Before searching, generate **3 distinct query framings** for the research question. This is critical — the Vocabulary Problem (Furnas 1987) shows that the probability two people use the same term for the same concept is <20%. A single query framing misses 80% of how relevant content might be worded.

**The three framings:**

1. **User's natural language** — the question as-is, how a normal person would search
2. **Expert/technical terminology** — how a domain expert or academic would phrase it. Use jargon, formal terms, specific model names, version numbers, standard abbreviations
3. **Lateral/adjacent framing** — rephrase the question from a different angle entirely:
   - Search for the **problem** the thing solves, not the thing's name
   - Search for **who** would know (person, team, community) rather than **what**
   - Use terms from an **adjacent field** that discusses the same concept differently
   - Search for a **specific symptom/artifact** rather than the general topic

**Examples:**

| Research Question | Framing 1 (natural) | Framing 2 (expert) | Framing 3 (lateral) |
|-------------------|---------------------|--------------------|--------------------|
| "Is Claude good at coding?" | `Claude AI coding review` | `Claude Opus benchmark SWE-bench HumanEval performance` | `best LLM for code generation comparison 2025` |
| "Fix slow file manager Ubuntu" | `Ubuntu file manager slow fix` | `nautilus high CPU GNOME 46 performance regression` | `xdg-desktop-portal dbus timeout snapd` |
| "How does deep research work?" | `deep research AI how it works` | `agentic search MCTS retrieval augmented generation pipeline` | `information foraging theory LLM agent query decomposition` |

**Use all three in your Google searches.** The first framing goes to the standard Google search. The second and third framings become additional Google searches (or replace the time-filtered search if the topic is evergreen). This triples your vocabulary coverage at minimal cost.

**Memory boost:** If research memory (Step -1) provided query patterns that worked for this topic, use those as a 4th framing or to refine the three above.

**Extra framing for practicality queries:** When the prompt includes ease/adoption/ecosystem intent, run a 4th query framing:
- **Ecosystem framing** — search for integrations/plugins/wrappers users actually deploy.
- Pattern examples: `"<product> plugin"`, `"<product> integration"`, `"<product> dashboard"`, `"awesome <product>"`, `"<product> self-hosted"`.

### Step 1: Load Skills & Launch Parallel Searches

**Load the needed skills** via the `Skill` tool, then execute searches following each skill's instructions.

#### 1a. Google Search (ALWAYS)
Load `google-search-browser` skill. Follow its instructions for navigating and extracting results.

**Recency filtering for Google:** Based on the recency assessment from Step 0, append a time-range parameter to at least one Google search:
- Past 24 hours: append `&tbs=qdr:d` to the Google search URL
- Past week: `&tbs=qdr:w`
- Past month: `&tbs=qdr:m`
- Past year: `&tbs=qdr:y`

**Two-pass strategy for Evolving/Recent topics:** Do one unfiltered search (for authoritative evergreen sources) and one time-filtered search (for fresh results). This catches both the canonical answer and any recent updates that supersede it.

#### 1b. Reddit Search (for opinions, experiences, discussions)
Load `reddit-search` skill. Follow its instructions for searching and reading threads.

#### 1c. Hacker News Search (for technical topics)
Load `hackernews-search` skill. Follow its instructions for Algolia search and reading discussions.

#### 1d. GitHub Search (for dev/tool topics)
Load `github-search` skill. Follow its instructions for Google + MCP tool search strategy.

#### 1e. Official Forum Search (when an official forum exists)

Search the official forum using **Google's `site:` operator** (more effective than most forums' built-in search):

```
Navigate to: https://www.google.com/search?q=site:<forum-domain>+<query+encoded>
```

For deep research, also search directly within the forum:
- **Discourse-based forums** (OpenAI, Cloudflare, Docker, etc.): `https://<forum>/search?q=<query>`
- **GitHub Discussions**: Use `mcp__github__search_issues` with the repo filter
- **Stack Exchange sites**: `https://<site>/search?q=<query>` or use Google `site:` operator

**When to use official forum search:**
- Always for **Troubleshooting** topics
- Always for **Product/Service** issues
- For **Deep research** on any topic
- Skip for generic **Opinion/Experience** questions

#### 1f. Ecosystem/Plugin Search (for integration-heavy or "easiest" queries)

When the user asks for easiest/common setup, explicitly search these surfaces:

- Official integration directories/marketplaces (if available)
- GitHub queries for wrappers/proxies/connectors around the product
- Community "awesome-*" lists and highly starred integrations

Then rank options by:
1. setup time,
2. operational complexity,
3. adoption evidence.

### Step 2: Intelligent Source Exploration (Best-First Search)

Research is **best-first search on an unknown information graph**. URLs are nodes. Pages contain edges (links to deeper content). You maintain a mental **priority queue** of all discovered URLs, always reading the most promising unread one next — regardless of what depth level it sits at.

There is no fixed "initial reads" vs "follow-up reads" split. There is no hard depth limit. The algorithm dynamically decides where to invest read budget based on what it discovers.

#### The Algorithm

```
QUEUE: all discovered URLs, each with a Promise Score
VISITED: set of URLs already read
FINDINGS: accumulated research knowledge
continue_research: true

1. SEED — Add all URLs from Step 1 searches to QUEUE at depth 0
2. SCORE — Compute Promise Score for every URL in QUEUE
3. READ — Pop the highest-scored unread URL
   - Route to skill (Reddit/GitHub/HN) or delegate to subagent
   - Mark as VISITED
4. HARVEST — Extract new links from the page (subagents return DISCOVERED_LINKS)
   - Add new links to QUEUE at depth = parent_depth + 1
   - Score each new link
5. EVALUATE — Check stopping conditions
6. REPEAT from step 3 if no stopping condition fired
```

#### Promise Score Heuristic (0-100)

Score every candidate URL before reading it. This is the core intelligence — spend a moment evaluating each link rather than reading blindly.

| Factor | Points | How to Assess |
|--------|--------|---------------|
| **Relevance** | 0-30 | Does the title/link text/snippet directly address the research question? Exact query terms in title = 25-30. Related terms = 15-24. Generic "Learn more" on relevant page = 10-14. Vague/unrelated = 0-9. |
| **Credibility** | 0-25 | Use the credibility tiers below. Tier 1 official = 25. Tier 2 expert = 20. Tier 3 community = 15. Tier 4 publication = 12. Tier 5 blog = 8. Tier 6 SEO/content mill = 0. |
| **Novelty** | 0-25 | Would this add NEW information vs what you already have? Different domain from pages already read = +10. Different content type (docs vs blog vs forum vs paper) = +8. Covers an aspect not yet explored = +7. Same-domain deeper page = +5. Likely duplicate/summary of what you have = 0. |
| **Context signals** | 0-20 | Linked from official/authoritative page = +8. Link text says "technical details" / "full documentation" / "API reference" / "whitepaper" = +6. Has visible date and is recent = +4. High engagement visible (votes, comments, reactions) = +2. |

**Depth decay** (soft multiplier, NOT a hard cutoff):

| Depth | Multiplier | Meaning |
|-------|-----------|---------|
| 0 | ×1.0 | Direct search results |
| 1 | ×0.9 | One click from search results |
| 2 | ×0.8 | Two clicks deep |
| 3 | ×0.7 | Three clicks deep |
| 4+ | ×0.6 | Very deep — only follow if raw score is exceptional |

**Final Promise Score = (Relevance + Credibility + Novelty + Context) × Depth Decay**

**Example scoring:**
- Official blog post (depth 0) directly about your topic → (28 + 25 + 20 + 4) × 1.0 = **77** → READ IMMEDIATELY
- "Full API documentation" link from that blog (depth 1) → (25 + 25 + 22 + 14) × 0.9 = **77** → READ NEXT (high novelty + authoritative context compensates for depth)
- Random Medium post (depth 0) with vague title → (12 + 8 + 10 + 0) × 1.0 = **30** → LOW PRIORITY
- Third docs sub-page (depth 3) covering edge case → (15 + 25 + 8 + 6) × 0.7 = **38** → MAYBE, only if budget remains
- SEO listicle (depth 0) → (10 + 0 + 5 + 0) × 1.0 = **15** → SKIP (below threshold)

Notice: the depth-1 docs link ties with the depth-0 official post at 77. **That's the point** — a high-value deep link can outrank a mediocre shallow one. Depth decays the score gently, it doesn't block exploration.

#### Stopping Conditions (evaluate after every wave)

Stop exploring when ANY of these fire:

1. **Diminishing returns** — repeated reads mostly confirm what you already know
2. **Queue starved** — no promising unread URLs remain
3. **Confident answer** — you can fully answer the specific question. **But first, pass the Answer-Completeness Gate (see below).**
4. **Saturation** — same facts/conclusions keep repeating across independent sources
5. **User stop signal** — user asks to stop/ship current findings

**No fixed depth limit.** A rich trail of official documentation might warrant reading 4 levels deep. A dead-end blog post gets 0 follow-ups. Depth is emergent from the scoring. Trust the heuristic.

#### Answer-Completeness Gate (required before stopping on "Confident answer")

Before triggering the "Confident answer" stop, explicitly verify alignment between your findings and the **specific** question asked. This gate prevents stopping when you've answered the general topic but missed what was actually asked.

**The check:** Re-read the original question. For each specific aspect or sub-question it contains, confirm you have a concrete answer — not just general coverage of the topic. If any aspect is unanswered, you do NOT have a confident answer.

**Common failure modes this catches:**
- Question asks "which option/config/setting" → you only know "which product/version" (answered the category, not the choice within it)
- Question asks "how to do X" → you know X exists and is good, but not the actual steps
- Question asks "X vs Y" → you deeply researched X but barely touched Y
- Question asks about a specific feature → you have general product overview but nothing on that feature
- Question asks for "easiest/common" way → you returned only official docs and skipped dominant integrations/plugins users actually use
- Question is provider/account ambiguous → you researched the wrong ecosystem (e.g., OpenRouter vs ChatGPT plan Codex)

**If the gate fails:** Do NOT stop. Instead, identify what's missing and target it — the missing aspect likely needs a read of the primary artifact (repo README, docs page, config reference) or a more specific search query. Resume the wave pattern.

**Mandatory artifact check:** The gate also verifies that the primary artifact (if one exists for this topic) has been read. If researching software/tools/models and the repo README or project page hasn't been read yet, the gate fails regardless of how many other sources you have. Commentary sources cannot substitute for the artifact itself.

#### Route known sites to their skills

| URL Pattern | Handle With |
|-------------|-------------|
| `reddit.com/r/*/comments/*` | Use `reddit-search` skill patterns |
| `reddit.com/search*` | Use `reddit-search` skill patterns |
| `github.com/*` | Use `github-search` skill / GitHub MCP tools |
| `hn.algolia.com/*` or `news.ycombinator.com/*` | Use `hackernews-search` skill patterns |

These skills produce compact, structured output. Use them directly — don't wrap in subagents.

#### Delegate unknown/generic pages to subagents

For all other URLs (blog posts, news articles, official forums, docs pages), **delegate to subagents** so raw page content stays out of the main context.

> **WHY SUBAGENTS?** Reading a generic page dumps 5-20k chars into context. With 3-5 pages, that's 15-100k chars. Subagents read pages in their own isolated context and return only concise findings (~500-1000 chars each).

Launch **parallel Task subagents** in waves — read the top 2-3 highest-scored URLs in parallel each wave:

```
Task({
  description: "Read <site-name> page",
  subagent_type: "general-purpose",
  prompt: `Read this URL and extract information relevant to the research question below.

URL: <url>
Research question: <the user's original question>

Page reading method:
1. Use playwright-logged-in: browser_navigate to the URL
2. Use playwright-logged-in: browser_snapshot to capture the full accessibility tree
3. Analyze the snapshot content for information relevant to the research question

Output rules:
- Return ONLY findings relevant to the research question
- Keep response under 1500 chars — be concise and specific
- Include: key facts, dates, usernames/authors if relevant, solutions/workarounds
- Always include the source URL at the end
- If the page is irrelevant or inaccessible, say so in one line

INFO_GAIN: Rate how much NEW information this page provided: HIGH (major new facts/data not findable elsewhere), MEDIUM (useful details that add depth), LOW (mostly confirms common knowledge), NONE (irrelevant/inaccessible/duplicate).

DISCOVERED_LINKS: Extract up to 5 links from this page that could lead to deeper content about the research topic. For each link provide:
- url: <url>
  text: <the visible link text or anchor text>
  type: <docs|blog|paper|forum|changelog|api-ref|tutorial|announcement|other>
  why: <one line — why this link might contain valuable information>
Only include links that would add NEW information. Skip: nav links, social links, unrelated pages, links to content already covered on this page.`
})
```

#### Priority constraints (override raw Promise Score)

These constraints take precedence over the computed score. They are hard overrides:

1. **Official source is ALWAYS the first read** — If you identified an official source in Step 0.5, it gets read first regardless of its score. Non-negotiable.
2. **Primary artifact is MANDATORY** — For any software, tool, model, library, or open-source project being researched, the **primary artifact** must be read and **cannot be skipped by any stopping condition** (including "Confident answer"). The primary artifact is the thing itself — not a page that talks *about* it, but the page that *defines* it:
   - **Open-source projects**: The GitHub repo README (use GitHub MCP tools to read it)
   - **Models/weights**: The model card or project page listing available variants/options
   - **Libraries/packages**: The npm/PyPI/crates.io page or repo README showing API and configuration
   - **Tools/apps**: The official downloads/docs page showing available options and settings
   - **Why this matters**: Papers, blogs, and forums discuss a thing. The artifact *defines* it — what options exist, what configurations are available, what the actual usage looks like. You cannot fully answer "which X should I use" without reading the thing itself. A research session that reads 3 commentary sources but skips the repo README has a blind spot.
   - **When to read it**: Include in Wave 1 alongside the official source. If discovered later via DISCOVERED_LINKS, it jumps to the top of the queue regardless of depth or current score.
3. **arXiv/research papers third** — For ML/AI topics, the technical paper is read before any third-party commentary.
4. **Never read a third-party summary before the primary source** — Even if a TechCrunch article scores higher due to relevance keywords, the official source comes first.

After these priority reads are done, the Promise Score governs all remaining read decisions.

**Recency tiebreaker:** When two URLs have similar Promise Scores (within 5 points), prefer the more recent one.

#### Source Credibility Tiers (used in Promise Score — Credibility factor)

| Tier | Source Type | Examples | Points |
|------|-----------|----------|--------|
| 1 | **Official sources** | Company blogs, official docs, changelogs, press releases | 25 |
| 2 | **Independent technical experts** | Known engineers' personal blogs, reputable tech journalists, peer-reviewed content | 20 |
| 3 | **Community forums with voting** | Reddit (high-upvote threads), HN (high-point threads), Stack Overflow (accepted answers) | 15 |
| 4 | **Established tech publications** | Ars Technica, The Verge, TechCrunch, etc. | 12 |
| 5 | **Generic blogs and tutorials** | Medium posts, dev.to, random WordPress blogs | 8 |
| 6 | **SEO-optimized content mills** | Sites with generic names, listicles, "Top 10 Best..." format | 0 |

**Red flags — score 0 credibility regardless of apparent tier:**
- **Affiliate/sponsored content**: "Best X in 2026" listicles are almost always affiliate marketing. Every product has a purchase link. Prefer forum threads where real users share unprompted opinions
- **Undisclosed advertising**: Company blogs reviewing their own product category (e.g., a VPN company's "best VPN" article). Look at the domain — if the reviewer sells the thing they're reviewing, it's an ad
- **Thin aggregator sites**: Pages that just summarize other pages without adding original insight. Go to the original source instead
- **Outdated SEO-dominant results**: Old pages that rank high purely due to accumulated backlinks, not current accuracy. Check the date before reading
- **AI-generated content farms**: Generic, repetitive phrasing, no author byline, covers every topic under the sun. Low signal
- **Non-expert echo chambers**: Blog posts that repackage the same wrong information from other non-expert blog posts. If 10 Medium articles all say the same thing but none cite a primary source, they're amplifying noise, not signal. Look for who said it FIRST and whether they had actual expertise/access.

**Positive signals — boost Novelty/Context scores:**
- **Real user experiences** on forums (Reddit, HN, official forums) with specific details, not vague praise
- **Author with a track record**: Named person with a history of writing about this topic, personal stake, or demonstrable expertise
- **Discussion with pushback**: Threads where people disagree and debate are more informative than echo chambers
- **Staff/maintainer responses** in official forums or GitHub issues
- **Primary sources**: The actual announcement, the actual changelog, the actual benchmark — not someone's summary of it
- **Contrarian evidence**: Sources that challenge the prevailing narrative deserve a read — they either expose real issues or strengthen the consensus by forcing you to evaluate it
- **Adoption evidence**: high stars/forks/downloads, active recent releases, broad mention across independent sources

#### When to skip subagents

- **Quick research**: If you only need to read 1 page, read it directly
- **Known-site URLs**: Reddit, GitHub, HN — handle with their respective skill patterns

#### Practical execution: Wave pattern

In practice, the Best-First Search executes in **waves** (because you launch parallel subagents, not sequential single reads):

**Wave 1** (after Step 1 searches complete):
- Score all search result URLs using Promise Score
- Read the top 2-3 highest-scored URLs in parallel (via subagents)
- Official source always included in Wave 1 (priority override)

**Wave 2** (after Wave 1 subagents return):
- Merge DISCOVERED_LINKS from all Wave 1 reads into the queue
- Re-score everything: new links compete with remaining search results on equal footing
- Check stopping conditions — if confident answer, diminishing returns, or queue starved → STOP
- **Run Mid-Wave Reflection (see below)** — this may generate entirely new queries
- Read the top 2-3 highest-scored remaining URLs in parallel (including any new search results)
- Note: a depth-2 docs link with score 75 beats a depth-0 blog post with score 35

**Wave 3+** (if budget remains and no stopping condition fired):
- Same pattern: merge new links, re-score, **run reflection again**, read best candidates
- Most research sessions complete in 2 waves. Deep investigations may use 3.

#### Mid-Wave Reflection (after each wave)

After each wave of reads returns, pause and explicitly reflect. This is the single highest-value step — it turns static query planning into adaptive, evolving search.

**Ask yourself these 4 questions:**

1. **What specific sub-questions remain UNANSWERED?**
   - Re-read the original question. For each aspect, do I have a concrete answer or just general coverage?
   - If gaps exist, generate a targeted query for each gap

2. **What NEW terminology did the sources use that I didn't search for?**
   - Pages often use terms you didn't think of. If a source calls the concept "retrieval-augmented reasoning" but you searched "agentic search", you're missing a whole vocabulary cluster.
   - Generate a new Google search using the discovered terminology

3. **Should I COMPLETELY change my search direction?**
   - Sometimes Wave 1 reveals that the question is actually about something different than initially assumed
   - Example: You searched for "slow file manager" but the pages all point to a dbus/portal issue — now search for THAT instead
   - Don't be afraid to abandon the original query framing entirely if the evidence points elsewhere

4. **Which source type gave the best results so far?**
   - If Reddit threads had the most actionable info, do another Reddit search with refined terms
   - If official docs were the goldmine, look for deeper docs pages
   - If a specific person/author keeps appearing, search for their other work directly

5. **Did I learn something that changes user intent scope?**
   - If yes, trigger Step -1.5 and ask a clarification question before more searching
6. **Was my initial interpretation likely wrong or incomplete?**
   - If yes, ask a re-clarification question immediately (do not continue searching on the old track)

**Output of reflection**: 0-2 new Google searches using terms you couldn't have generated at the start. These get executed immediately, and their results enter the queue to compete with everything else.

**Example reflection in action:**
```
Original question: "best keyboard for programming"
Wave 1 reads: review sites, a Reddit thread, an ergonomics blog

Reflection:
1. Unanswered: specific switch types for typing comfort (reviews focused on features, not feel)
2. New terminology: "thock", "linear vs tactile", "actuation force" — terms I didn't search
3. Direction change: No — still on track, just need more specific info
4. Best source: Reddit r/MechanicalKeyboards had real user experiences

→ New search: "best tactile switch for programming long typing sessions reddit"
→ This query couldn't have been written before reading Wave 1 sources
```

### Step 3: Synthesize and Report

Present findings in this format:

```
## Research: [Topic]

### Key Findings
- [Most important finding #1]
- [Most important finding #2]
- [Most important finding #3]
- [Additional findings as needed]

### Details
[2-4 paragraphs synthesizing the information, with inline links to sources]

### Community Sentiment (if applicable)
[What Reddit/HN users are saying - positive, negative, concerns]

### Recommended Path (for "easiest/best/popular" queries)
- Default recommendation: [the easiest path you recommend first]
- Why: [1-2 concrete reasons: fewer steps, better adoption, lower ops burden]
- Alternative: [official-native or more advanced option]

### Sources
- [Source Title](URL) - brief note on what it covers *(date if known)*
- [Source Title](URL) - brief note *(date if known)*
- ...

### Freshness Note (if applicable)
[Flag if key sources are outdated, if the landscape is changing fast, or if information conflicts between old and new sources. E.g., "Pricing info is from Jan 2026 — verify on their site as it may have changed." or "Older guides recommend X but as of v3.0 (Dec 2025) the recommended approach is Y."]
```

### Step 4: Update Research Memory

After delivering the final synthesis, update the memory files to capture what worked. **ONLY write to files inside `~/.claude/skills/research/memory/`. NEVER modify SKILL.md.**

**What to record** (be concise — each topic file should stay under 40 lines):

1. **Which sources gave the best results?** (e.g., "Reddit r/LocalLLaMA was the best source for GPU benchmarks")
2. **Which query patterns worked?** (e.g., "Searching author name + topic found the niche paper")
3. **Which sites were useless or misleading?** (e.g., "Medium AI articles were all repackaged, skip for this topic")
4. **Navigation tips discovered?** (e.g., "On HuggingFace model cards, the 'Usage' section has what you need")
5. **What terminology shift happened?** (e.g., "User asked about 'slow file manager' but the real topic was 'xdg-desktop-portal dbus timeout'")

**How to save:**

1. **Does a matching topic file already exist?** (from Step -1 glob)
   - **Yes**: Read it, append new learnings under the relevant section, remove anything outdated
   - **No**: Create a new file with a descriptive name: `~/.claude/skills/research/memory/<topic-slug>.md`

2. **Update `_general.md`** only if you learned something that applies across ALL topics (rare — most learnings are topic-specific)

**Topic file template** (for new files):
```markdown
# [Topic Name] Research Strategies

## Best Sources
- [source]: [why it works for this topic]

## Query Patterns That Work
- [pattern]: [example query]

## Sites to Avoid
- [site]: [why]

## Navigation Tips
- [site/tool]: [how to get the most out of it]

## Terminology Map
- User says "[X]" → experts call it "[Y]"
```

**Rules:**
- Keep files concise — strategies and hints only, not research findings
- One topic per file — don't merge unrelated topics
- Delete outdated entries (e.g., if a site improved or a subreddit died)
- Skip this step for trivial research sessions (single Google search, obvious answer)

## Reading Official Forum Threads

Official forums (especially Discourse-based) often have the most valuable first-hand reports and staff responses. Use `browser_run_code` to extract clean content:

**For Discourse-based forums** (OpenAI, Cloudflare, Docker, etc.):
```js
browser_run_code({ code: `async (page) => {
  await page.goto('https://community.openai.com/t/TOPIC-SLUG/ID');
  await page.waitForLoadState('domcontentloaded');
  await page.waitForTimeout(2000);
  return await page.evaluate(() => {
    const title = (document.querySelector('.fancy-title') || document.querySelector('h1'))?.innerText || '';
    const posts = document.querySelectorAll('.topic-body, .post');
    let md = "# " + title + "\\n\\n";
    let count = 0;
    posts.forEach(post => {
      if (count++ > 20) return;
      const author = post.querySelector('.username, .names .username')?.innerText || '';
      const body = post.querySelector('.cooked')?.innerText || '';
      if (!body.trim()) return;
      md += "**" + author + "**: " + body.substring(0, 600) + "\\n\\n---\\n\\n";
    });
    return md.substring(0, 10000);
  });
}` })
```

**For generic forums** (non-Discourse), use `browser_navigate` + `browser_snapshot`.

**Key signals to look for:**
- Staff/official responses (badges, special formatting)
- "Solved" or "Accepted answer" markers
- High reply counts
- Recent timestamps

## Efficiency Rules

### Parallel Execution
- **Always** launch Google + at least one other source in parallel
- Don't wait for one search to finish before starting another
- Use `playwright-logged-in` for non-GitHub pages (multiple tabs)
- Use GitHub MCP tools for any GitHub URLs
- Use multiple browser tabs for parallel page reads

### Early Termination
- If Google results clearly answer the question with authoritative sources, skip community searches
- If the topic is very new (hours old), prioritize Google and direct page reads
- If you find the official documentation/announcement, read it first

### Depth Control
- **Quick research** (simple factual question): use only enough reads to answer confidently
- **Standard research** (how-to, new feature): cover official source plus the most relevant community/implementation evidence
- **Deep research** (complex topic, many opinions): expand breadth and depth until additional reads stop improving the answer

### Research Boundaries (soft guidance)
- No fixed depth limit — let Promise Score and usefulness drive exploration
- Prefer stopping when additional reads stop changing the answer
- Users can stop research at any point; provide best-so-far synthesis on request
- Scale source count to question complexity (concise for simple asks, broader for deep dives)

## Browser Allocation

Default to `playwright-logged-in` — it has cookies, avoids CAPTCHAs (especially Reddit), and handles rate limiting better. Use `playwright-fresh` only when isolation genuinely matters.

For parallelism, use **multiple tabs** on the same browser.

## Handling Very New Topics

For topics released in the last few hours/minutes:
1. Google search first — news sites index fast
2. Check the official source directly (company blog, GitHub releases, etc.)
3. Reddit/HN may not have discussions yet — check but don't rely on them
4. Be transparent: "This was just announced, so community reactions are still forming"

## Anti-Patterns

- **Don't read third-party summaries before the official source** — if Anthropic announced something, read `anthropic.com/blog` first, not VentureBeat's summary of it. Third-party articles add commentary but lose technical detail.
- **Don't skip finding the official source** — if you don't know the official site for a product, find it first (scan Google results for company domains, or search `<product> official site`). Never assume a third-party article is "good enough."
- **Don't assume official == easiest** — when asked for easiest/common workflow, validate what users actually deploy (integrations, plugins, wrappers) and compare it against the official route.
- **Don't skip ecosystem search for integration-heavy products** — if plugins/connectors/marketplaces exist, they are part of the answer, not optional trivia.
- **Don't start searching before intent is clarified** — for ambiguous queries, ask one scope-defining question first (provider/account/context) to avoid wasted research.
- **Don't treat clarification as one-time** — if ambiguity appears mid-research, pause and ask again.
- **Don't over-research before clarifying** — if initial understanding is weak, do minimal reconnaissance only, then clarify before deeper searches.
- Don't search all sources for simple factual questions (Google alone suffices)
- Don't keep reading mechanically — stop when new reads no longer improve the answer
- Don't include low-quality Reddit posts (score < 10) in findings
- Don't present raw search results — always synthesize into insights
- Don't sequentially search when you can search in parallel
- Don't duplicate skill-specific patterns inline — load the skill instead
- Don't use `gh` CLI via Bash for GitHub data — use `mcp__github__*` tools
- Don't open GitHub URLs in the browser — use GitHub MCP tools instead
- **Don't use fixed depth limits** — let the Promise Score heuristic guide how deep to go. A rich trail of official docs might warrant 4 levels deep; a dead-end blog deserves 0 follow-ups. Depth is emergent, not hardcoded.
- **Don't ignore discovered links** — every page read can surface high-value links. Score them and let them compete fairly in the priority queue against remaining candidates, regardless of depth level.
- **Don't keep reading when returns diminish** — if the last 2 reads just confirmed what you already know (INFO_GAIN: LOW/NONE), stop. Spending budget on redundant reads wastes slots that could go to unexplored angles.
- **Don't trust non-expert echo chambers** — 10 blog posts saying the same thing doesn't make it true if none cite a primary source. The web is full of non-experts redistributing the same wrong information. Always trace claims back to whoever said it first and whether they had actual expertise or access.
- **Don't skip the primary artifact** — For software, tools, models, or libraries, reading 3 commentary sources (blogs, Reddit, papers) is NOT a substitute for reading the repo README or project page. Commentary tells you what people *think* about it; the artifact tells you what it *actually offers* (options, configs, variants, flags). If you haven't read the artifact, your answer has a blind spot — you're reporting opinions about a thing you haven't examined yourself.
- **Don't stop on "Confident answer" without checking completeness** — Re-read the original question before stopping. If the user asked "which model option" and you only know "which version," that's not a confident answer — it's a confident answer to a different question. The Answer-Completeness Gate exists for this reason; use it.
- **NEVER modify SKILL.md** — Memory writes go ONLY to files inside `~/.claude/skills/research/memory/`. The skill definition file is not a scratchpad.
