# LessWrong RSS Feed Builder

A simple tool to build custom RSS feed URLs for [LessWrong](https://www.lesswrong.com).

**[Open the tool](https://brendanlong.github.io/lesswrong-rss-builder/)**

## Features

- Post feeds: default, frontpage, curated, community, meta, by-tag
- Comment feeds: per-post (multiple sort orders), per-user, shortform/quick takes, tag subforum, debate responses, moderator comments, and more
- Username slug to user ID lookup via LessWrong's GraphQL API
- Karma threshold filtering for post feeds
- Context-aware UI that only shows relevant options for the selected view

## LessWrong RSS Feed Reference

LessWrong's RSS feed is at `https://www.lesswrong.com/feed.xml`. It returns the 10 most recently published posts by default, but supports many query parameters for customization.

### Quick Examples

| Use case | URL |
|---|---|
| Default (10 newest posts) | `https://www.lesswrong.com/feed.xml` |
| Frontpage only | `https://www.lesswrong.com/feed.xml?view=frontpage` |
| Posts by a user | `https://www.lesswrong.com/feed.xml?userId=USER_ID` |
| Comments on a post | `https://www.lesswrong.com/feed.xml?type=comments&view=postCommentsNew&postId=POST_ID` |
| Quick takes (frontpage) | `https://www.lesswrong.com/feed.xml?type=comments&view=shortformFrontpage` |
| User's quick takes | `https://www.lesswrong.com/feed.xml?type=comments&view=shortform&userId=USER_ID` |

### Query Parameters

| Parameter | Type | Description |
|---|---|---|
| `type` | string | Set to `comments` for comment feeds; omit for post feeds |
| `view` | string | Feed view (see tables below). Defaults to `rss` |
| `karmaThreshold` | number | Minimum karma for posts (see [Karma Threshold Logic](#karma-threshold-logic)) |
| `userId` | string | Filter by author user ID |
| `postId` | string | Filter comments to a specific post |
| `tagId` | string | For `tagRelevance` post view or tag comment views |
| `parentCommentId` | string | For `commentReplies` view |
| `parentAnswerId` | string | For `repliesToAnswer` view |
| `topLevelCommentId` | string | For `repliesToCommentThread` view |
| `forumEventId` | string | For `forumEventComments` view |
| `sortBy` | string | Sort mode for views that support it (see [sortBy Values](#sortby-values)) |
| `filterSettings` | JSON | Advanced filtering (JSON-encoded object) |

### Post Feed Views

Post feeds return **10 items**. This limit is not configurable.

| View | Description | Sort Order |
|---|---|---|
| `rss` (default) | All newest posts | postedAt descending |
| `frontpageRss` | Frontpage posts only | frontpageDate descending |
| `curatedRss` | Curated posts only | curatedDate descending |
| `communityRss` | Non-frontpage posts with karma > 2 | postedAt descending |
| `metaRss` | Meta posts only | postedAt descending |
| `tagRelevance` | Posts by tag (requires `tagId`) | Tag relevance score |

View names accept either camelCase (`frontpageRss`) or kebab-case (`frontpage-rss`).

### Comment Feed Views

All comment views require `type=comments`. Comment feeds return **50 items**. This limit is not configurable.

#### General

| View | Description | Sort |
|---|---|---|
| `rss` (default) | Recent comments with positive score | postedAt desc |
| `recentComments` | Recent comments with positive score | postedAt desc |
| `allRecentComments` | All recent comments including neutral/negative | postedAt desc |
| `commentReplies` | Replies to a comment (requires `parentCommentId`) | postedAt desc |
| `moderatorComments` | Comments posted with a moderator hat | postedAt desc |

#### Per-Post (require `postId`)

These views exclude answers and answer-replies.

| View | Description | Sort |
|---|---|---|
| `postCommentsNew` | Comments on a post, newest first | postedAt desc |
| `postCommentsOld` | Comments on a post, oldest first | postedAt asc |
| `postCommentsTop` | Comments on a post, highest karma | baseScore desc |
| `postCommentsBest` | Comments on a post, best first | baseScore desc |
| `postCommentsMagic` | Comments on a post, Wilson sort | score desc |
| `postCommentsRecentReplies` | Comments by recent subthread activity | lastSubthreadActivity desc |
| `postsItemComments` | Recent non-deleted comments on a post | postedAt desc |
| `questionAnswers` | Answers to a question post (supports `sortBy`) | baseScore desc |
| `answersAndReplies` | Answers and their replies (supports `sortBy`) | baseScore desc |
| `debateResponses` | Debate responses on a post | postedAt asc |
| `recentDebateResponses` | Recent debate responses | postedAt desc |

#### Per-User

| View | Description | Sort |
|---|---|---|
| `profileComments` | Comments by a user (supports `sortBy`) | postedAt desc |

#### Shortform / Quick Takes

| View | Description | Sort |
|---|---|---|
| `shortform` | Top-level shortform comments | lastSubthreadActivity desc |
| `topShortform` | Top shortform by score | baseScore desc |
| `shortformFrontpage` | Frontpage shortform, filtered by quality | score desc |

#### Per-Tag (require `tagId`)

| View | Description | Sort |
|---|---|---|
| `tagDiscussionComments` | Discussion comments on a tag | default |
| `tagSubforumComments` | Subforum comments for a tag (supports `sortBy`) | lastSubthreadActivity desc |

#### Other

| View | Description | Sort |
|---|---|---|
| `repliesToAnswer` | Replies to a specific answer (requires `parentAnswerId`) | postedAt desc |
| `repliesToCommentThread` | Full thread under a comment (requires `topLevelCommentId`) | postedAt desc |
| `forumEventComments` | Comments on a forum event (requires `forumEventId`) | postedAt desc |

### sortBy Values

Views that support the `sortBy` parameter accept these values:

| Value | Sort Order |
|---|---|
| `top` | baseScore descending |
| `new` / `newest` | postedAt descending |
| `old` / `oldest` | postedAt ascending |
| `magic` | score descending |
| `recentComments` | lastSubthreadActivity descending |

### Karma Threshold Logic

The `karmaThreshold` parameter controls when posts appear in the feed based on when they reached certain karma levels. Input values are rounded to the nearest supported threshold:

| Input Range | Actual Threshold | Date Field Used |
|---|---|---|
| < 16 (or not set) | 2 | scoreExceeded2Date |
| 16-36 | 30 | scoreExceeded30Date |
| 37-59 | 45 | scoreExceeded45Date |
| 60-99 | 75 | scoreExceeded75Date |
| 100-161 | 125 | scoreExceeded125Date |
| >= 162 | 200 | scoreExceeded200Date |

The feed item's date is the later of the karma threshold date (when the post reached the threshold) and the view-specific date (e.g., frontpageDate for the frontpage feed). This allows higher-threshold feeds to surface older posts that recently became popular.

### Finding User IDs

The RSS feed requires user IDs, not usernames. To look up a user ID from a username slug, use LessWrong's GraphQL API:

```bash
curl -s -X POST https://www.lesswrong.com/graphql \
  -H 'Content-Type: application/json' \
  -d '{"query": "{ user(input: { selector: { slug: \"USERNAME\" } }) { result { _id } } }"}'
```

## License

MIT
