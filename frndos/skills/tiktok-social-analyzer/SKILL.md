---
name: tiktok-social-analyzer
description: TikTok profile and content analysis using direct TikTokApi scraping tools. Use when analyzing TikTok users, videos, trends, or searching for TikTok accounts.
metadata:
  version: "1.0.0"
  author: frnd-team
  tags: ["tiktok", "social-media", "analysis", "scraping"]
---

# TikTok Social Analyzer Skill

Use this skill when performing TikTok analysis as part of social media profiling.

## When to Use

- User asks to analyze a TikTok profile or username
- User provides a TikTok URL (tiktok.com/@username or video links)
- Task requires TikTok data as part of cross-platform social media analysis
- User asks about TikTok trending content
- User wants to search for a person/brand on TikTok

## Available Tools

You have these TikTok-specific tools (prefixed with `tiktok_`):

| Tool | Purpose | Key Args |
|------|---------|----------|
| `tiktok_user_info` | Get user profile data (bio, followers, following) | `username` |
| `tiktok_user_videos` | Get recent videos from a user | `username`, `count` |
| `tiktok_video_info` | Get details about a specific video | `video_url` |
| `tiktok_video_comments` | Get comments on a video | `video_url`, `count` |
| `tiktok_trending_videos` | Get currently trending videos | `count` |
| `tiktok_search_users` | Search for TikTok users by keyword | `query`, `count` |

## Workflow

### Profile Analysis (most common)

1. **Get user info**: Call `tiktok_user_info(username)` to get profile data
2. **Get recent videos**: Call `tiktok_user_videos(username, count=10)` to see content patterns
3. **Analyze content**: Look at video captions, hashtags, and engagement metrics
4. **Cross-reference**: Compare username and bio with other platform findings

### Video Analysis

1. **Get video info**: Call `tiktok_video_info(video_url)` for stats and metadata
2. **Get comments**: Call `tiktok_video_comments(video_url, count=10)` for audience sentiment
3. **Check author**: Extract author info from video data for profile linking

### User Search

1. **Search**: Call `tiktok_search_users(query)` with the person's name or known handle
2. **Verify**: Cross-reference returned profiles with known info from other platforms
3. **Deep dive**: Call `tiktok_user_info` on the most likely match

## Important Constraints

- **Read-only**: These tools can only retrieve data, not post or interact
- **No authentication**: Only publicly available data is accessible
- **Rate sensitivity**: TikTok may block requests if too many are made quickly
- **Error handling**: If a tool returns `{"error": ...}`, report it and fall back to Tavily web search
- **Max count**: Each tool returns at most 30 items per call
- **Username format**: Always strip the `@` prefix before passing to tools

## Error Recovery

If `tiktok_user_info` or `tiktok_user_videos` fails with an error:
1. Report the error in your analysis (mark TikTok status as ERROR)
2. Fall back to Tavily web search with `site:tiktok.com` for basic profile data
3. Do NOT retry the same tool call — move on to other platforms

## Output Integration

When including TikTok data in your cross-platform analysis:
- Include follower/following counts in the profile summary
- List recent video themes and hashtags as content patterns
- Note engagement rates (likes/views ratio) as digital behaviour signals
- Use TikTok bio for cross-platform name/identity verification
