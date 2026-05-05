# X Search Cookbook

## Contents

- Bird-specific search facts
- Query construction principles
- Operator families
- Quality recipes
- Competitor recipes
- Pagination and backfill
- Common failure modes

## Bird-Specific Search Facts

Bird calls X web SearchTimeline with `product: "Latest"`. It does not expose X’s “Top” ranking mode. Therefore, quality comes from query engineering:

- constrain authors/lists
- add date windows
- add engagement thresholds
- filter tweet type/media/link shape
- exclude low-quality terms
- read full threads after finding hits

Search broad first only when discovering vocabulary. For extraction, narrow the query.

## Query Construction Principles

Use single quotes around shell queries so parentheses and quotes pass through intact:

```bash
bird search '(from:openai OR from:anthropicai) (launch OR release OR model OR API OR pricing) min_faves:1000 -filter:replies' -n 50 --json
```

Use X operators inside the query, not shell pipes. Keep queries short; X search can silently degrade when too complex. Split long competitor sets into multiple queries or list-based searches.

Avoid dot-heavy exact phrases when possible. X tokenization can turn `"bird.fast"` into unrelated “fast bird” matches. Prefer handle/domain/source constraints:

```bash
bird search '(@steipete/bird OR "Bird CLI" OR from:steipete bird) -filter:replies' -n 20 --json
```

## Operator Families

Core text:

- `word1 word2`: implicit AND
- `word1 OR word2`: OR must be uppercase
- `"exact phrase"`: phrase match
- `(a OR b) c`: grouping
- `-word`, `-"phrase"`: exclusions
- `+term`: force exact term where X tries spelling correction
- `#hashtag`, `$TICKER`, `@handle`

Authors and graph:

- `from:handle`: posts by account
- `to:handle`: replies to account
- `@handle`: mentions account
- `list:<id>` or `list:owner/slug`: posts from list members
- `filter:follows`: posts from accounts the authenticated account follows

Dates and language:

- `since:YYYY-MM-DD`
- `until:YYYY-MM-DD`
- `since:YYYY-MM-DD_HH:MM:SS_UTC`
- `until:YYYY-MM-DD_HH:MM:SS_UTC`
- `lang:en`, `lang:ja`, etc.

Engagement:

- `min_faves:N`
- `min_retweets:N`
- `min_replies:N`
- `-min_faves:N`, `-min_retweets:N`, `-min_replies:N` for upper bounds
- `filter:has_engagement`
- `-filter:has_engagement`

Tweet type:

- `filter:replies`, `-filter:replies`
- `filter:quote`
- `filter:self_threads`
- `filter:retweets`, `-filter:retweets`
- `filter:nativeretweets`
- `include:nativeretweets`
- `-filter:nativeretweets`
- `conversation_id:<tweet_id>`
- `quoted_tweet_id:<tweet_id>`
- `quoted_user_id:<user_id>`

Content/media:

- `filter:links`, `-filter:links`
- `url:domain.com`
- `filter:media`
- `filter:images`
- `filter:videos`
- `filter:native_video`

Official X API v2 uses newer `is:` and `has:` operators (`-is:retweet`, `has:links`, `has:media`), while X web search and Bird commonly accept classic web operators (`-filter:retweets`, `filter:links`, `filter:media`). For Bird, prefer the web-search forms above unless testing proves an API form works.

## Quality Recipes

Find top original posts from an account:

```bash
bird search 'from:handle min_faves:500 -filter:replies -filter:nativeretweets' -n 50 --json
```

Find product/launch posts across competitors:

```bash
bird search '(from:comp1 OR from:comp2 OR from:comp3) (launch OR release OR shipped OR introducing OR update OR API OR pricing) min_faves:100 -filter:replies' -n 100 --json
```

Find reception to a competitor:

```bash
bird search '@competitor -from:competitor filter:has_engagement since:2026-01-01 lang:en' -n 100 --json
```

Find customer pain:

```bash
bird search '(@competitor OR "Competitor Name") (broken OR expensive OR confusing OR "wish" OR "why does" OR "how do I") min_replies:2 lang:en -filter:nativeretweets' -n 100 --json
```

Find high-signal tutorials/how-to content:

```bash
bird search '(\"how to\" OR tutorial OR guide OR walkthrough) (topic1 OR topic2) min_faves:100 lang:en -filter:replies' -n 100 --json
```

Find high-conviction founder/operator takes:

```bash
bird search '(from:founder1 OR from:founder2 OR from:founder3) (learned OR lesson OR mistake OR strategy OR distribution) min_faves:100 -filter:replies' -n 100 --json
```

Find fast-moving early signals:

```bash
bird search '(topic OR synonym) min_faves:20 min_replies:3 since:2026-05-01 lang:en -filter:replies' -n 100 --json
```

Find already-viral posts:

```bash
bird search '(topic OR synonym) min_faves:5000 OR min_retweets:500 lang:en -filter:replies'
```

Prefer parenthesized forms for OR:

```bash
bird search '(topic OR synonym) (min_faves:5000 OR min_retweets:500) lang:en -filter:replies' -n 50 --json
```

## Competitor Recipes

Monthly competitor backfill:

```bash
bird search 'from:competitor since:2026-04-01 until:2026-05-01 -filter:replies -filter:nativeretweets' -n 100 --json
```

Top competitor posts by engagement:

```bash
bird search 'from:competitor since:2026-01-01 min_faves:500 -filter:replies' -n 100 --json
```

Competitor launch analysis:

```bash
bird search '(from:competitor OR @competitor) (launch OR release OR waitlist OR beta OR pricing OR integration) since:2026-01-01 lang:en' -n 100 --json
```

Compare several accounts without overloading query complexity:

```bash
bird search '(from:comp1 OR from:comp2) min_faves:200 -filter:replies since:2026-01-01' -n 100 --json
bird search '(from:comp3 OR from:comp4) min_faves:200 -filter:replies since:2026-01-01' -n 100 --json
```

List-based sector feed:

```bash
bird list-timeline <list-id> -n 100 --max-pages 5 --json
bird search 'list:<list-id> (launch OR pricing OR integration) min_faves:20 since:2026-01-01' -n 100 --json
```

## Pagination And Backfill

Do not rely on deep pagination for comprehensive backfill. Split by dates:

```bash
bird search 'topic min_faves:50 lang:en since:2026-01-01 until:2026-01-08' -n 100 --json
bird search 'topic min_faves:50 lang:en since:2026-01-08 until:2026-01-15' -n 100 --json
```

For high-volume topics, use daily or hourly windows. For low-volume competitor accounts, monthly windows may be fine.

## Common Failure Modes

- Garbage results: add `from:`, `list:`, `lang:`, `min_faves`, `-filter:replies`, and exclusions.
- Missing historical posts: X search is incomplete; use `user-tweets`, date windows, or official/full-archive alternatives.
- Too few results: lower engagement thresholds, remove exact phrases, add synonyms, widen dates.
- Too much slop: raise engagement thresholds, constrain to handles/lists, add `-giveaway`, `-airdrop`, `-follow`, `-"RT"` or domain-specific exclusions.
- “Latest” bias: Bird search returns recent/latest-biased results; use engagement thresholds and backfill windows to approximate “top”.
