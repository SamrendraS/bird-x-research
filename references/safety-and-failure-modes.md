# Safety And Failure Modes

## Contents

- Account risk
- Stale public skills/docs
- Credential handling
- Rate limits and breakage
- Write safety
- Fallback strategy

## Account Risk

Bird uses undocumented X/Twitter web GraphQL endpoints and web-session cookies. X can block, rate-limit, challenge, suspend, or move accounts to read-only mode. Risk increases with:

- automated writes
- automated follows/unfollows
- high-volume pagination
- repeated login/session cookie use across environments
- scraping many pages quickly
- running unbounded `--all`
- brand-new accounts with robotic behavior

Prefer read-only workflows. Use a burner account for experimentation. Keep page caps and delays.

## Stale Public Skills/Docs

Many public Bird skills are stale. Common stale claims:

- Homebrew formula `steipete/tap/bird` may not exist even if docs still mention it.
- Sweetistics engine/API support was present in early Bird but removed by 0.3.0; Bird 0.8.0 does not expose `--engine sweetistics`.
- Minimal skills only mention `whoami`, `read`, `thread`, `search`, `tweet`, and `reply`, missing most current commands.

Before relying on external skill text, inspect installed Bird:

```bash
bird --version
bird --help
bird <command> --help
npm root -g
```

Then inspect source if needed:

```bash
rg "register.*Command|\\.command\\(" /opt/homebrew/lib/node_modules/@steipete/bird/dist
```

## Credential Handling

Never print or store cookie values in notes, logs, or reports. Use environment variables or config chosen by the user. If tokens are pasted into a task, treat them as secrets even if described as burner credentials.

`bird check` confirms whether auth is available but may say “Ready to tweet” because Bird sees write-capable cookies. That does not mean the agent should tweet.

## Rate Limits And Breakage

Symptoms:

- HTTP 404: query ID rotated
- HTTP 429: rate-limited
- GraphQL “Query: Unspecified”
- no results from a query that should work
- stale cursor loops
- missing fields

Recovery:

1. Run `bird query-ids --fresh` once.
2. Retry with smaller `-n`, lower pages, and longer delay.
3. Simplify the query.
4. Split by date windows instead of deep pagination.
5. Use `--json-full` only for debugging.
6. Fall back to official X API, browser automation, or a commercial provider if reliability matters.

## Write Safety

Bird’s CLI mutating commands do not ask for confirmation. Enforce confirmation externally.

Never run without explicit confirmation:

- `tweet`
- `reply`
- `follow`
- `unfollow`
- `unbookmark`
- media upload
- package library methods: `like`, `unlike`, `retweet`, `unretweet`, `bookmark`

Confirmation must include the exact action, target account/tweet, and text/media if any.

## Fallback Strategy

Use Bird when:

- you need quick X web search with advanced web operators
- you need threads/bookmarks/likes/list timelines in a local agent workflow
- occasional breakage is acceptable
- read-only research is enough

Use official X API when:

- production reliability matters
- you need compliant long-running automation
- writes/posting are required
- account suspension risk is unacceptable

Use browser automation when:

- Bird cannot read a UI-only surface
- an X Article or media view does not extract cleanly
- a one-off manual-like inspection is needed

Use commercial scraping/search providers when:

- you need scale
- you need monitoring/alerts
- you need durable pagination/backfill
- you can pay for reliability and compliance review
