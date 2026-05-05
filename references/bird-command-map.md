# Bird Command Map

## Contents

- Global behavior
- Authentication and config
- Output schema
- Read commands
- Search and discovery
- Timelines, lists, bookmarks, likes
- Accounts and graph
- News/trending
- Mutating commands
- Library-only capabilities

## Global Behavior

Bird 0.8.0 is a Node/Bun CLI around X/Twitter web GraphQL endpoints. It authenticates with web-session cookies, not OAuth. It is fast, useful, and fragile: X can rotate operation query IDs, change response shapes, rate-limit, or block automated behavior.

Global options:

```bash
--auth-token <token>
--ct0 <token>
--chrome-profile <name>
--chrome-profile-dir <path>
--firefox-profile <name>
--cookie-timeout <ms>
--cookie-source <safari|chrome|firefox>   # repeatable
--timeout <ms>
--quote-depth <depth>                     # default 1, 0 disables quoted tweet expansion
--plain
--no-emoji
--no-color
--media <path>                            # write commands only
--alt <text>                              # write commands only
```

Use `--json` whenever the output will be parsed by an agent. Use `--json-full` when debugging missing fields or needing raw GraphQL payloads; it adds `_raw` and can be large.

## Authentication And Config

Credential precedence:

1. CLI flags: `--auth-token`, `--ct0`
2. Environment: `AUTH_TOKEN`, `CT0`, falling back to `TWITTER_AUTH_TOKEN`, `TWITTER_CT0`
3. Browser cookies via Safari, Chrome/Chromium, or Firefox

Config precedence:

1. CLI flags
2. Env vars
3. Project config: `./.birdrc.json5`
4. Global config: `~/.config/bird/config.json5`

Config fields:

```json5
{
  cookieSource: ["firefox", "safari"],
  chromeProfile: "Default",
  chromeProfileDir: "/path/to/Chromium/Profile-or-Cookies-db",
  firefoxProfile: "default-release",
  cookieTimeoutMs: 30000,
  timeoutMs: 20000,
  quoteDepth: 1
}
```

Env helpers: `BIRD_TIMEOUT_MS`, `BIRD_COOKIE_TIMEOUT_MS`, `BIRD_QUOTE_DEPTH`, `BIRD_QUERY_IDS_CACHE`, `BIRD_FEATURES_CACHE`, `BIRD_FEATURES_PATH`, `BIRD_FEATURES_JSON`.

## Output Schema

Tweet JSON fields:

- `id`
- `text`
- `author.username`
- `author.name`
- `authorId`
- `createdAt`
- `replyCount`
- `retweetCount`
- `likeCount`
- `conversationId`
- `inReplyToStatusId`
- `quotedTweet`
- `media[]`: `type`, `url`, `previewUrl`, `width`, `height`, `videoUrl`, `durationMs`
- `article`: `title`, `previewText`
- `_raw` only with `--json-full`

User JSON fields from `following`/`followers`:

- `id`, `username`, `name`, `description`, `followersCount`, `followingCount`, `isBlueVerified`, `profileImageUrl`, `createdAt`

Paginated JSON usually returns:

```json
{ "tweets": [], "nextCursor": "..." }
```

If no cursor is present, pagination is exhausted or blocked.

## Read Commands

```bash
bird read <tweet-id-or-url> [--json|--json-full]
bird <tweet-id-or-url> [--json]  # shorthand
```

Reads a single tweet. It extracts Notes and Articles when present. Use this to verify a search hit before summarizing.

```bash
bird thread <tweet-id-or-url> [--all] [--max-pages N] [--cursor C] [--delay ms] [--json|--json-full]
```

Fetches the conversation thread containing a tweet. Bird filters/sorts by `conversationId`, so this is best for reconstructing author threads and quote context.

```bash
bird replies <tweet-id-or-url> [--all] [--max-pages N] [--cursor C] [--delay ms] [--json|--json-full]
```

Fetches direct replies to a tweet. Use for reception analysis, objections, spam filtering, customer complaints, and influencer response quality. Always cap pages.

## Search And Discovery

```bash
bird search '<query>' -n 50 --json
bird search '<query>' --all --max-pages 3 --json
bird search '<query>' --cursor '<cursor>' --json
```

Bird’s search uses X web `SearchTimeline` with `product: "Latest"`. It does not expose Top/Latest selection. Quality must come from the query: account constraints, date windows, engagement thresholds, language, media/link filters, and exclusions.

`--max-pages` on `search` requires `--all` or `--cursor`.

```bash
bird mentions -n 20 --json
bird mentions --user @handle -n 20 --json
```

Mentions searches for tweets mentioning a user. Useful for brand reception, customer issues, influencer mentions, and competitor comparisons.

## Timelines, Lists, Bookmarks, Likes

```bash
bird user-tweets @handle -n 100 --max-pages 5 --delay 1000 --json
```

Fetches a profile timeline by resolving the handle to a user ID. Maximum is 10 pages/200 tweets per run. It includes profile-timeline content, so check for replies, repost text, quotes, and self-thread fragments.

```bash
bird home -n 50 --json
bird home --following -n 50 --json
```

Fetches authenticated For You or chronological Following feed. Use sparingly; it is account-personalized and not a neutral market sample.

```bash
bird lists -n 100 --json
bird lists --member-of -n 100 --json
bird list-timeline <list-id-or-url> -n 100 --max-pages 5 --json
```

Lists are excellent for competitor or category research because they pre-curate accounts. Avoid unbounded `list-timeline --all`; Bird’s own help warns it can risk account bans. Prefer list search queries too: `list:<id> keyword min_faves:50`.

```bash
bird bookmarks -n 100 --json
bird bookmarks --folder-id <id> -n 100 --json
bird bookmarks --all --max-pages 3 --json
```

Bookmark thread-context flags:

- `--expand-root-only`: expand only if bookmarked tweet is the root
- `--author-chain`: include author self-reply chain connected to the bookmark
- `--author-only`: include all tweets by the bookmarked tweet author in that thread
- `--full-chain-only`: include reply chain connected to the bookmark
- `--include-ancestor-branches`: with full chain, include sibling branches of ancestors
- `--include-parent`: include direct parent for non-root bookmarks
- `--thread-meta`: add `isThread`, `threadPosition`, `hasSelfReplies`, `threadRootId`
- `--sort-chronological`: sort globally oldest to newest

High-signal bookmark extraction:

```bash
bird bookmarks --expand-root-only --author-chain --thread-meta --sort-chronological --json
bird bookmarks --include-parent --thread-meta --json
bird bookmarks --full-chain-only --include-ancestor-branches --thread-meta --json
```

```bash
bird likes -n 100 --json
bird likes --all --max-pages 3 --json
```

Likes can become an implicit reading queue, but they may include low-intent likes. Treat them as signals to rescore, not as a clean knowledge base.

## Accounts And Graph

```bash
bird whoami --json
bird about @handle --json
bird following -n 100 --json
bird followers -n 100 --json
bird following --user <userId> -n 100 --json
bird followers --user <userId> -n 100 --json
```

`about` returns “About this account” data such as account-based-in/source when available. `following`/`followers` requires a numeric user ID for another account. Get that ID from `authorId` in tweet JSON or from `user-tweets` output.

## News/Trending

```bash
bird news -n 10 --json
bird news --ai-only -n 20 --json
bird news --news-only --ai-only -n 10 --json
bird news --with-tweets --tweets-per-item 3 -n 10 --json
bird trending --trending-only -n 10 --json
```

Default tabs are For You, News, Sports, Entertainment; Trending is excluded by default to reduce noise. Use news/trending for discovery, then verify source tweets and external sources.

## Query IDs

```bash
bird query-ids --json
bird query-ids --fresh --json
```

Bird caches GraphQL operation query IDs at `~/.config/bird/query-ids-cache.json` with a 24h TTL. On 404, Bird refreshes and retries for several operations. If repeated reads fail, refresh once, retry once, then fall back.

## Mutating Commands

Require explicit confirmation:

```bash
bird tweet "text"
bird tweet "text" --media img.png --alt "description"
bird reply <tweet-id-or-url> "text"
bird follow <username-or-id>
bird unfollow <username-or-id>
bird unbookmark <tweet-id-or-url...>
```

Bird does not implement a built-in confirmation prompt. The agent must enforce confirmation before running these.

## Library-Only Capabilities

The installed package’s `TwitterClient` includes methods beyond registered CLI commands, including `like`, `unlike`, `retweet`, `unretweet`, and `bookmark`. Treat these as mutating actions requiring explicit confirmation. Do not create custom scripts that call them unless the user deliberately asks for those actions.
