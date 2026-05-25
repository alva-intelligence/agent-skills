# TikTok Tools — Capabilities & Constraints

## Data Retrieval Capabilities

### User Profile (`tiktok_user_info`)
Returns a dict with fields including:
- `uniqueId` / `nickname` — username and display name
- `signature` — bio text
- `verified` — blue checkmark status
- `followerCount`, `followingCount`, `heartCount` — engagement metrics
- `videoCount` — total public videos
- `avatarLarger` — profile photo URL (may expire)
- `region` — user's region setting
- `openFavorite` — whether liked videos are public

### User Videos (`tiktok_user_videos`)
Returns a list of video dicts, each with:
- `id` — video ID
- `desc` — caption text
- `createTime` — Unix timestamp
- `stats` — `{ diggCount, shareCount, commentCount, playCount }`
- `hashtags` — list of hashtag objects with `name` and `id`
- `music` / `sound` — audio information
- `author` — embedded user info

### Video Info (`tiktok_video_info`)
Same structure as user videos but for a single video. Accepts full URL.

### Video Comments (`tiktok_video_comments`)
Returns a list of comment dicts with:
- `text` — comment text
- `user` — commenter info (username, nickname)
- `digg_count` — likes on the comment
- `create_time` — Unix timestamp

### Trending Videos (`tiktok_trending_videos`)
Returns trending video objects (same structure as user videos).

### User Search (`tiktok_search_users`)
Returns user objects matching the search query. Note: the `ms_token` used
must have previously performed a search on tiktok.com for this to work.

## Known Limitations

1. **Anti-bot detection**: TikTok actively detects and blocks automated requests.
   The `ms_token` cookie helps bypass this but may expire.
2. **No private data**: Only publicly visible profiles and content are accessible.
3. **No posting**: The API is read-only — cannot like, comment, follow, or post.
4. **Search restrictions**: User search requires an `ms_token` that has search history.
5. **Data freshness**: Profile stats may be slightly delayed.
6. **Structure changes**: TikTok may change their API response structure without notice.
   Always handle missing fields gracefully.

## Troubleshooting

| Symptom | Likely Cause | Action |
|---------|-------------|--------|
| `EmptyResponseException` | TikTok blocking request | Report error, use Tavily fallback |
| `{"error": "Browser has no attribute"}` | Playwright not installed | Infrastructure issue — report to admin |
| Empty user info | User doesn't exist or is private | Mark as NOT_FOUND |
| Search returns no results | ms_token needs search history | Use Tavily `site:tiktok.com` instead |
| Timeout errors | TikTok rate limiting | Report error, continue with other platforms |
