# Research Workflows

## Contents

- Competitor intelligence
- Viral/content pattern mining
- Bookmark/likes knowledge workflows
- Thread and reception analysis
- News and market monitoring
- Account graph research
- Cross-source grounding

## Competitor Intelligence

Goal: collect competitor activity, identify what worked, classify strategic moves, and preserve evidence.

Inputs:

- competitor handles
- product/category keywords
- date range
- optional curated list ID
- minimum engagement threshold

Process:

1. Fetch recent profile timelines:

```bash
bird user-tweets @competitor -n 100 --max-pages 5 --json
```

2. Fetch top original posts for the period:

```bash
bird search 'from:competitor since:2026-01-01 until:2026-05-01 min_faves:100 -filter:replies -filter:nativeretweets' -n 100 --json
```

3. Fetch product/narrative posts:

```bash
bird search 'from:competitor (launch OR release OR pricing OR API OR integration OR benchmark OR customer OR case study) since:2026-01-01 -filter:replies' -n 100 --json
```

4. Fetch external reception:

```bash
bird search '@competitor -from:competitor filter:has_engagement since:2026-01-01 lang:en' -n 100 --json
```

5. For each candidate high-signal post, read context:

```bash
bird thread <id-or-url> --max-pages 3 --json
bird replies <id-or-url> --max-pages 3 --delay 1500 --json
```

6. Categorize and score using [scoring-and-kb.md](scoring-and-kb.md).

Output:

- top posts table
- narrative/product categories
- engagement-normalized winners
- repeated formats/hooks
- audience objections and praise
- recommended watch queries

## Viral And Content Pattern Mining

Goal: learn what formats, hooks, topics, and proof points outperform.

Queries:

```bash
bird search '(topic OR synonym) min_faves:1000 lang:en -filter:replies -filter:nativeretweets since:2026-01-01' -n 100 --json
bird search '(from:knowncreator1 OR from:knowncreator2) min_faves:500 -filter:replies since:2026-01-01' -n 100 --json
```

For each result, record:

- opening hook pattern
- claim type: result, contrarian take, tutorial, teardown, announcement, story, warning
- evidence type: screenshot, metric, demo, chart, customer quote, source link
- CTA or community move
- media/link/thread shape
- replies sentiment

Do not optimize only for raw likes. Compare against author baseline and audience fit.

## Bookmark/Likes Knowledge Workflows

Bookmarks and likes are valuable because the user already filtered them. The hard part is context recovery and prioritization.

Lightweight inbox:

```bash
bird bookmarks -n 100 --include-parent --thread-meta --json
bird likes -n 100 --json
```

Thread-aware extraction:

```bash
bird bookmarks --expand-root-only --author-chain --thread-meta --sort-chronological --json
```

Full context for selected bookmarks:

```bash
bird bookmarks --full-chain-only --include-ancestor-branches --thread-meta --json
```

Workflow:

1. Deduplicate by `id`.
2. Group by author, topic, category, and actionability.
3. Promote items with high novelty, high relevance, or clear next action.
4. For ambiguous items, run `thread` or `read`.
5. Save to KB with source query `bookmarks` or `likes`.

Useful outputs:

- daily reading brief
- idea bank
- competitor watchlist
- actions extracted from bookmarks
- Obsidian/Notion-ready markdown cards

## Thread And Reception Analysis

Use when a post matters enough to inspect context and audience reaction.

```bash
bird read <id> --json
bird thread <id> --max-pages 3 --json
bird replies <id> --max-pages 3 --delay 1500 --json
```

Analyze:

- author’s full argument across thread
- quoted tweet context
- top objections
- high-authority replies
- spam/reply-bait ratio
- customer support issues
- repeated questions
- whether engagement came from praise, controversy, or confusion

## News And Market Monitoring

Bird news/trending is useful as a discovery layer:

```bash
bird news --ai-only --with-tweets --tweets-per-item 3 -n 10 --json
bird trending --trending-only --with-tweets --tweets-per-item 3 -n 10 --json
```

Then verify through source tweets, web search, official docs, filings, blogs, or news sites. Do not treat X trending as ground truth.

Recurring monitor pattern:

1. Run a small set of watch queries.
2. Deduplicate against stored tweet IDs.
3. Score relevance and novelty.
4. Read threads for top items.
5. Produce a brief with links and why each item matters.

## Account Graph Research

Use graph commands to understand who a competitor, founder, or category account follows or attracts.

```bash
bird user-tweets @handle -n 5 --json
bird following --user <authorId> -n 100 --max-pages 3 --json
bird followers --user <authorId> -n 100 --max-pages 3 --json
```

Use cases:

- discover adjacent influencers
- build a curated X list
- identify partner/customer clusters
- find accounts to add to monitoring

Do not auto-follow. Recommend accounts; ask for confirmation before mutation.

## Cross-Source Grounding

Bird is strong for X-native evidence, but competitor intelligence should not stop there.

Use other sources for:

- official company announcements
- docs/changelog pages
- GitHub releases/issues
- SEC/financial filings
- product pages/pricing pages
- Reddit/Hacker News/customer communities
- media coverage

Cross-source rule: if a tweet makes a factual business, financial, legal, technical, or product claim, mark it as unverified until grounded elsewhere.
