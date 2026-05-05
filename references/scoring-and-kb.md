# Scoring And KB Ingestion

## Contents

- Minimum record schema
- Suggested labels
- Scoring model
- Competitor-specific interpretation
- Report format

## Minimum Record Schema

Store every ingested tweet with:

```json
{
  "tweet_id": "string",
  "url": "https://x.com/{username}/status/{id}",
  "author_username": "string",
  "author_name": "string",
  "author_id": "string",
  "created_at": "string",
  "text": "string",
  "reply_count": 0,
  "retweet_count": 0,
  "like_count": 0,
  "conversation_id": "string",
  "in_reply_to_status_id": "string|null",
  "quoted_tweet_id": "string|null",
  "media": [],
  "article": null,
  "source_query": "string",
  "source_command": "string",
  "fetched_at": "ISO-8601",
  "labels": [],
  "score": {},
  "notes": "string"
}
```

Deduplicate by `tweet_id`. Preserve `source_query` and `source_command` so results are reproducible.

## Suggested Labels

Competitor/product:

- `product_launch`
- `feature_update`
- `pricing_packaging`
- `integration`
- `partnership`
- `customer_story`
- `case_study`
- `benchmark`
- `technical_claim`
- `roadmap`
- `funding`
- `hiring`
- `regulatory`
- `security`
- `outage`

Content/market:

- `viral_format`
- `founder_pov`
- `tutorial`
- `teardown`
- `contrarian_take`
- `meme`
- `community_complaint`
- `influencer_reaction`
- `competitor_mention`
- `market_narrative`
- `customer_pain`
- `social_proof`

Research status:

- `verified`
- `needs_verification`
- `inference`
- `low_quality`
- `spam`
- `duplicate`

## Scoring Model

Raw engagement:

```text
engagement_score = likeCount + 2*retweetCount + 3*replyCount
```

Quality score, 0-5:

- 5: specific, novel, relevant, evidence-backed, strong audience signal
- 4: relevant and actionable, some evidence or strong reception
- 3: useful but generic, needs more context
- 2: weak signal, low specificity, or mostly promotional
- 1: noise, spam, engagement bait, irrelevant

Competitor intelligence score, 0-5:

- strategic relevance: product/pricing/positioning/customer evidence
- novelty: new or under-discussed
- durability: likely to matter beyond the day
- evidence strength: source quality and specificity
- reception: meaningful replies/quotes, not just passive likes

Account-relative score:

1. Fetch recent posts from the same author with `user-tweets`.
2. Compute median raw engagement for comparable post types.
3. Mark a post as outperforming if it is >2x median or top decile.

This matters because 1,000 likes can be weak for a huge account and massive for a niche competitor.

## Competitor-Specific Interpretation

For each high-scoring competitor post, answer:

- What did they announce, claim, or imply?
- What audience segment is it aimed at?
- What evidence did they show?
- What positioning language did they use?
- What objections appeared in replies?
- What did the audience repeat or quote?
- Is this a one-off or part of a repeated narrative?
- What should we watch next?

Mark facts and inferences separately:

```text
Fact: The post announced a new API endpoint and linked docs.
Inference: They are pushing developer adoption before broader enterprise launch.
Confidence: Medium, based on two launch posts and docs timing.
```

## Report Format

For a concise research brief:

```markdown
## Top Signals

1. [@handle: short title](https://x.com/handle/status/id)
   - Date:
   - Engagement:
   - Labels:
   - Why it matters:
   - Evidence:
   - Confidence:

## Patterns

- Pattern:
- Supporting posts:
- Implication:

## Watch Queries

- `from:competitor ...`
- `@competitor -from:competitor ...`
```

For KB cards:

```markdown
---
tweet_id:
author:
created_at:
labels:
source_query:
score:
---

# Short Title

Source: https://x.com/...

Summary:

Why it matters:

Evidence:

Open questions:
```
