---
name: reddit-search
description: Intelligent multi-step Reddit search with keyword intelligence. Analyzes query clarity, uses Google to refine ambiguous keywords, and finds the most relevant communities by combining global Reddit search, Google search with site:reddit.com, and targeted deep searches. Asks user for clarification when queries are completely unclear.
---

## What I do
- **Analyze query clarity** and intelligently refine keywords when user queries are ambiguous
- Use **Google search to disambiguate** unclear terms and find better keywords (e.g., "python" → "python programming IDE")
- **Ask user for clarification** when queries are completely unclear (e.g., "is it worth it" → "what specifically?")
- Perform intelligent multi-step searches to find the most relevant Reddit communities for any topic
- Combine global Reddit search + Google search (site:reddit.com) to identify best subreddits
- Do targeted deep searches in the most relevant communities
- Extract post data, comments, and community insights
- Return structured analysis with ranked community relevance
- **Adaptively refine keywords** during search if initial results are poor quality or irrelevant

## When to use me
Use this skill when the user asks to:
- Search Reddit for discussions about a topic
- Find the best subreddits for a specific topic
- Get comprehensive Reddit research across multiple communities
- Analyze which Reddit communities are most active for a topic
- Extract detailed discussions from the most relevant communities

## Quick Decision Tree

```
User Query
    ↓
Is it clear and specific?
    ↓
YES ────────→ Use keywords as-is → Start Step 1
    ↓
NO
    ↓
Search Google for context
    ↓
Did Google clarify intent?
    ↓
YES ────────→ Use refined keywords → Start Step 1
    ↓
NO
    ↓
Ask user for clarification
```

**Examples:**
- "Claude AI promo code" → Clear → Search directly
- "best python" → Ambiguous → Google research → Search with refined keywords
- "what should I do" → Unclear → Ask user "what situation are you in?"
- "is it worth it" → Completely unclear → Ask user "what specifically?"

## Query Clarity Levels

### Level 1: Crystal Clear ✅
- **Examples:** "Claude 50% off promo code", "best mechanical keyboard 2024", "how to learn python"
- **Action:** Use keywords as-is, proceed to Step 1

### Level 2: Ambiguous ⚠️
- **Examples:** "best python" (snake/programming?), "apple review" (fruit/company?), "javascript help" (too broad)
- **Action:** Google search first to determine context, then refine keywords

### Level 3: Completely Unclear ❓
- **Examples:** "is it worth it", "what do you think", "help me decide", "opinions?"
- **Action:** Ask user for clarification before searching

## Keyword Intelligence Workflow

**Before searching, ask yourself:**
1. What is the user actually looking for?
2. Are there multiple meanings for these keywords?
3. Will Reddit search understand this query?
4. Do I need to add context (year, category, specific model)?

**If in doubt:**
- Google the keywords first to see what comes up
- Check if top results match likely user intent
- Refine keywords based on current trends and context

## When to Use This Skill

Use this skill for user queries like:
- "Search Reddit for [topic]"
- "Find discussions about [topic] on Reddit"
- "What does Reddit say about [topic]?"
- "Best subreddits for [topic]"
- Any query asking for Reddit-specific information or community recommendations

## Multi-Step Intelligent Search Process (with Early Termination)

**EFFICIENCY PRINCIPLE:** Stop searching as soon as you have a clear, comprehensive answer. Don't fetch unnecessary data.

### Step 0: Query Analysis & Keyword Intelligence

**BEFORE you start searching, analyze the user's query:**

#### Query Clarity Assessment
1. **Is the query specific and clear?**
   - "Claude 50% off deals" ✅ - Clear topic
   - "best programming language" ⚠️ - Vague, needs context
   - "what should I buy" ❌ - Completely unclear

2. **Does it have ambiguous terms?**
   - "Python" could mean snake, programming language, or Monty Python
   - "Apple" could mean fruit or tech company
   - "Claude" could mean AI assistant or person's name

#### Intelligent Keyword Strategy

**If query is CLEAR:**
- Proceed directly to Step 1 with the user's exact keywords
- Example: "Claude 50% off promo code" → Search as-is

**If query is AMBIGUOUS or UNCLEAR:**
**Option A: Google Keyword Research (Preferred)**
```
Search Google first: https://www.google.com/search?q={user_query}
```
- Look at Google's suggestions, related searches, and top results
- Extract 2-3 better keywords that disambiguate the topic
- Use these refined keywords for Reddit search

**Example:**
- User query: "best python"
- Google search shows results for "best python IDE", "best python courses", "best python libraries"
- Check which context matches user's intent based on current trends
- Use refined keywords: "best python IDE 2024" or "best python courses reddit"

**Option B: Ask User for Clarification (If still unclear)**
If Google research doesn't clarify the intent:
```
Ask user: "When you say '[query]', are you asking about:
- Option A: [context 1]
- Option B: [context 2]
- Something else?"
```

**Keyword Expansion Strategy:**
- **Too broad:** "best phone" → Add year/budget/specific model: "best phone 2024", "best budget phone under $500"
- **Too vague:** "is X worth it" → Add context: "X review reddit", "X worth buying 2024"
- **Regional:** "best streaming service" → May need geo-specific keywords

#### Query Intent Detection

**Informational Queries (seeking knowledge):**
- "how to", "what is", "why does", "guide", "tutorial"
- → Use keywords as-is, focus on educational subreddits

**Opinion/Recommendation Queries:**
- "best", "recommend", "should I", "worth it", "vs"
- → Add "reddit" suffix or search specific comparison subreddits

**Deal/Price Queries:**
- "cheap", "deal", "discount", "sale", "coupon"
- → Add current month/year for recency: "claude promo code january 2025"

**Problem/Troubleshooting Queries:**
- "help", "error", "issue", "not working", "bug"
- → Include error message or specific symptom in keywords

### Step 1: Global Reddit Search (All Subreddits) - QUICK SCAN
Search across all of Reddit to identify promising communities:

**URL Pattern:**
```
https://www.reddit.com/search/?q={keywords}&type=posts&sort=new&t=month
```

**⚠️ CRITICAL: Reddit's "new" sort often shows low-quality results**
- Many posts are off-topic, memes, or spam
- **Always apply the Default Claude Filter** (see below)
- Combine with Google search (Step 2) to find truly relevant discussions

**Default Claude Filter - Apply to ALL Reddit results:**
```
MINIMUM THRESHOLD: Score >= 10 AND Comments >= 5
PREFERRED: Score >= 50 AND Comments >= 20
HIGH QUALITY: Score >= 100 AND Comments >= 50
```
- Skip posts below minimum threshold unless they're the ONLY results
- Prioritize posts with high engagement-to-age ratio (hot posts)
- Check post titles for keyword relevance - skip if off-topic

**Early Termination Checkpoints:**
- ✅ **STOP if** you find 3+ highly relevant posts (score >50, >20 comments) that fully answer the query
- ✅ **STOP if** you find a dedicated subreddit (e.g., r/ClaudeAI for Claude) with 5+ quality posts
- ⏭️ **CONTINUE if** results are scattered, low-quality, or lack depth (proceed to Google search)

**What to extract (minimal):**
- Top 5-10 posts that PASS the Default Claude Filter
- Which subreddits appear 2+ times with quality posts
- Quick relevance assessment (don't deep-read yet)

### Step 2: Google Search with site:reddit.com - ESSENTIAL QUALITY FILTER
**⚠️ CRITICAL: This step is MANDATORY, not optional**

Reddit's native search often fails to surface the best content. Google search acts as a quality filter and relevance validator.

**URL Pattern:**
```
https://www.google.com/search?q={keywords}+site:reddit.com&tbs=qdr:m
```

**Why Google Search is Essential:**
- Google's algorithm better identifies high-quality, relevant discussions
- Surfaces posts that Reddit's "new" sort buries
- Finds authoritative discussions with high engagement
- Often reveals different communities than Reddit search
- **Acts as the primary quality filter when Reddit results are noisy**

**Google Search Strategy:**
1. Use the SAME keywords as Reddit search
2. Extract top 5-10 results (Google already ranks by relevance)
3. Note which subreddits appear most frequently in Google results
4. Compare with Reddit search results - Google often finds better posts

**Decision Matrix:**
| Reddit Results | Google Results | Action |
|---------------|----------------|---------|
| High quality (5+ good posts) | Confirms Reddit | Use Reddit findings, stop |
| High quality | Finds better posts | Use Google's top posts |
| Low quality/scattered | High quality | Rely primarily on Google results |
| Low quality | Low quality | Expand search terms or time range |

**Early Termination Checkpoints:**
- ✅ **STOP if** Google results provide 3+ high-quality posts that answer the query
- ✅ **STOP if** Google and Reddit agree on top communities with quality content
- ⏭️ **CONTINUE to Step 3** only if both searches yield incomplete results

### Step 3: Analyze & Rank Communities - COMBINED FILTER
Combine Reddit + Google results to identify TOP 3 communities:

**Combined Scoring (Reddit results + Google results):**
```
Score = (Reddit Frequency × 1) + (Google Frequency × 2) + (Avg Engagement / 10)

Bonus:
+ 20 points: Dedicated community (e.g., r/ClaudeAI for Claude)
+ 10 points: Post with >200 votes in either search
+ 5 points: Post appears in BOTH Reddit and Google results
```

**Why weight Google higher?**
- Google's algorithm is better at identifying quality
- If Google surfaces a community multiple times, it's likely more relevant
- Reddit's "new" sort is noisy; Google acts as a validator

**Ranking Tiers:**
1. **Tier 1 (Score >30):** Dedicated community OR appears frequently in both searches
2. **Tier 2 (Score 15-30):** Appears in Google OR has high engagement posts
3. **Tier 3 (Score <15):** Only in Reddit search, moderate engagement

**STOP if you have:**
- One Tier 1 community with 3+ quality posts
- Two Tier 2 communities with clear discussions
- Any post with >200 votes that directly answers the query

### Step 4: Targeted Deep Searches - MINIMAL DEPTH
Only search the top 1-3 communities, with strict limits:

**Query Strategy:**
- Use ONLY 1-2 query variations per community (not 5+)
- Extract maximum 5 posts per community (not 10)
- Fetch JSON comments ONLY for posts with >50 comments AND high relevance

**Hard Limits:**
- Maximum 3 communities total
- Maximum 5 posts per community  
- Maximum 2 posts for full comment extraction
- If you find 3 excellent posts that answer the query, STOP immediately

## URL Patterns

### Global Reddit Search
```
https://www.reddit.com/search/?q={encoded_keywords}&type=posts&sort=new&t={time}
```

### Subreddit-Specific Search
```
https://www.reddit.com/r/{subreddit}/search/?q={encoded_keywords}&sort=new&t={time}
```

### Google Search (site:reddit.com)
```
https://www.google.com/search?q={encoded_keywords}+site:reddit.com&tbs=qdr:{time}
```

### Post JSON Endpoint
```
https://www.reddit.com/r/{subreddit}/comments/{post_id}.json
```

## Time Filter Options

**Reddit:**
- `hour` - Past hour
- `day` - Past 24 hours
- `week` - Past week
- `month` - Past month
- `year` - Past year
- `all` - All time

**Google (tbs parameter):**
- `qdr:h` - Past hour
- `qdr:d` - Past day
- `qdr:w` - Past week
- `qdr:m` - Past month
- `qdr:y` - Past year

## Sort Options (Reddit)
- `relevance` - Most relevant (default)
- `hot` - Currently trending
- `top` - Highest voted
- `new` - Most recent first
- `comments` - Most commented

## Data Extraction

### From Search Results Page
Extract for each post:
- Title
- Subreddit name
- Author
- Score (votes)
- Number of comments
- Post URL
- Timestamp
- Award count (if available)

### From JSON Endpoint
```json
[
  {
    "kind": "Listing",
    "data": {
      "children": [{
        "kind": "t3",
        "data": {
          "title": "Post Title",
          "author": "username",
          "subreddit": "subreddit_name",
          "selftext": "Post content...",
          "score": 42,
          "num_comments": 10,
          "upvote_ratio": 0.85,
          "created_utc": 1234567890
        }
      }]
    }
  },
  {
    "kind": "Listing",
    "data": {
      "children": [{
        "kind": "t1",
        "data": {
          "author": "commenter",
          "body": "Comment text...",
          "score": 5,
          "replies": { ... }
        }
      }]
    }
  }
]
```

## Output Format (Concise)

Return ONLY what's necessary to answer the query. Be brief.

### 1. Quick Answer (if found early)
If you found a clear answer in Step 1 or 2, provide it immediately:
```
## Answer
[Direct answer to user's query]

## Source
- r/SubredditName: "Post Title" (X votes, X comments)
- URL: [link]
```

### 2. Community Rankings (only if multiple communities needed)
List ONLY communities you actually searched:
```
## Relevant Communities
1. **r/CommunityName** - [1 sentence why]
2. **r/CommunityName** - [1 sentence why]
```

### 3. Key Findings (bullet points, max 5)
- Most important insight
- Second key point
- Third finding (if relevant)

### 4. Top Posts (max 5, only if they add value)
For each: Title, subreddit, votes - ONE key insight only

**DON'T include:**
- Sections that don't add value
- Posts with <10 votes unless crucial
- Detailed methodology unless asked
- Full comment threads unless essential

## Example Usage

### Example 1: Clear Query
**User query:** "latest Claude model"

**Step 1 - Global Reddit Search:**
```
https://www.reddit.com/search/?q=latest%20claude%20model&type=posts&sort=new&t=month
```
**Apply Default Claude Filter:** Keep only posts with score >= 10, comments >= 5
**Result:** Many scattered results, some low-quality, r/ClaudeAI appears but mixed with noise

**Step 2 - Google Search (MANDATORY):**
```
https://www.google.com/search?q=latest+claude+model+site:reddit.com&tbs=qdr:m
```
**Result:** Google surfaces higher quality posts, confirms r/ClaudeAI is primary community, reveals r/singularity has good discussions

**Step 3 - Combined Analysis:**
| Community | Reddit | Google | Engagement | Score |
|-----------|--------|--------|------------|-------|
| r/ClaudeAI | 3 posts | 5 posts | High (avg 200+ votes) | 35 (Tier 1) |
| r/singularity | 1 post | 3 posts | High (avg 150+ votes) | 22 (Tier 2) |
| r/LocalLLaMA | 2 posts | 1 post | Medium (avg 50+ votes) | 12 (Tier 3) |

**Decision:** Focus on r/ClaudeAI (Tier 1) + r/singularity (Tier 2)

**Step 4 - Deep Search (only Tier 1 & 2):**
```
https://www.reddit.com/r/ClaudeAI/search/?q=Claude%204%20OR%20Sonnet%205&sort=new&t=week
https://www.reddit.com/r/singularity/search/?q=Claude%20release%20OR%20Anthropic&sort=new&t=week
```
**Extract:** Top 5 posts from each that pass Default Claude Filter

### Example 2: Ambiguous Query (Keyword Intelligence)
**User query:** "best python"

**Step 0 - Query Analysis:**
Query is ambiguous - could mean programming language, snake, or Monty Python.

**Google Keyword Research:**
```
https://www.google.com/search?q=best+python
```
**Result:** Top results are about Python programming language (IDEs, courses, libraries)

**Decision:** Proceed with programming context: "best python IDE 2024"

**Step 1 - Reddit Search with refined keywords:**
```
https://www.reddit.com/search/?q=best%20python%20IDE%202024&type=posts&sort=new&t=year
```

### Example 3: Unclear Query (User Clarification)
**User query:** "is it worth it"

**Step 0 - Query Analysis:**
Query is completely unclear - no subject mentioned.

**Google Keyword Research:**
```
https://www.google.com/search?q=is+it+worth+it
```
**Result:** Results are mixed - subscriptions, investments, products, life decisions

**Ask User for Clarification:**
```
"When you ask 'is it worth it', what are you referring to? For example:
- A specific product or service?
- A subscription (Netflix, gym, etc.)?
- An investment or purchase?
- Something else?"
```

## Efficiency Guidelines (CRITICAL)

### Early Termination Triggers
**STOP searching when:**
1. ✅ You find 3+ high-quality posts (>50 votes, >20 comments) that fully answer the query
2. ✅ You find a dedicated community (e.g., r/ClaudeAI) with clear, comprehensive discussions
3. ✅ Any single post with >200 votes directly answers the user's question
4. ✅ You've collected enough data to provide a confident answer (usually 5-10 total posts)
5. ✅ Results become repetitive (same information across multiple posts)

### Efficiency Rules
- **Always apply Default Claude Filter** to Reddit results (score >= 10, comments >= 5)
- **Always do Google search** - it's mandatory for quality filtering, not optional
- **Weight Google results higher** (2x) when ranking communities
- **Don't fetch JSON** unless the post is highly relevant (>50 comments AND directly answers query)
- **Limit deep searches** to 1-3 communities maximum
- **Extract minimal data** - titles, votes, comment counts, key insights only
- **Return early** - better to give a quick, accurate answer than exhaustive data

### Hard Limits (Never Exceed)
- Max 3 communities for deep search
- Max 5 posts per community
- Max 2 posts for full JSON comment extraction
- Max 10 total posts in final output
- If query is answered in Step 1, skip Steps 2-4 entirely

### Rate Limiting
- Add 2-3 second delays between requests
- Timeout settings:
  - Page navigation: 10s
  - Search results: 10s
  - JSON extraction: 15s

### Quality Checks
- **Handle errors**: Some subreddits may block search
- **Respect Reddit TOS**: Don't spam requests
- **Filter low-quality**: Skip posts with <5 votes unless essential

## Adaptive Keyword Refinement

**During search, if results are poor, refine keywords:**

### Signs Keywords Need Refinement:
- Reddit search returns mostly irrelevant posts (spam, memes, wrong topic)
- Google results show different meaning than intended
- No posts pass the Default Claude Filter (score <10 or <5 comments)
- Results are too old (all from 1+ years ago)

### Keyword Refinement Strategies:

**1. Add Context/Scope:**
- "laptop" → "laptop gaming" or "laptop programming" or "laptop budget 2024"

**2. Add Time Constraint:**
- "iphone review" → "iphone review 2024" or "iphone review january 2025"

**3. Specify Format:**
- "learn javascript" → "learn javascript reddit guide" or "javascript tutorial beginner"

**4. Use Synonyms:**
- "cheap" → "budget", "affordable", "deal", "discount"
- "best" → "top rated", "recommended", "review"

**5. Add Reddit-Specific Terms:**
- Append "reddit" or "subreddit" to disambiguate
- Use "AMA" for expert discussions
- Use "ELI5" for simple explanations

### Example Refinement Flow:
**Initial search:** "good movies" → Too broad, scattered results
**Refined:** "best movies 2024 reddit" → Better but still broad
**Final:** "underrated movies netflix reddit recommendation" → High quality, specific results

## Anti-Patterns to Avoid

❌ **Don't:** Search only one subreddit
✅ **Do:** Start broad, then narrow down quickly

❌ **Don't:** Rely only on Reddit's "new" sort (too noisy, low-quality)
✅ **Do:** Always combine with Google site:reddit.com search for quality filtering

❌ **Don't:** Skip the Default Claude Filter (score >= 10, comments >= 5)
✅ **Do:** Always filter Reddit results - "new" sort surfaces too much noise

❌ **Don't:** Treat Google search as optional
✅ **Do:** Make Google search mandatory - it finds quality posts Reddit buries

❌ **Don't:** Continue searching after finding a clear answer
✅ **Do:** Stop immediately when query is answered (usually after 5-10 good posts)

❌ **Don't:** Fetch all results exhaustively
✅ **Do:** Use early termination - get enough to answer, then stop

❌ **Don't:** Search too many communities
✅ **Do:** Max 3 communities, often just 1 dedicated community is enough

❌ **Don't:** Extract full comments from every post
✅ **Do:** Only fetch JSON for posts with >50 comments AND high relevance

❌ **Don't:** Include exhaustive methodology in output
✅ **Do:** Provide concise answer with key sources only

❌ **Don't:** Use ambiguous keywords without clarification
✅ **Do:** Always assess query clarity and disambiguate before searching

❌ **Don't:** Persist with poor keywords if results are irrelevant
✅ **Do:** Refine keywords adaptively based on search results quality
