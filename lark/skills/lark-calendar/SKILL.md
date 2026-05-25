---
name: lark-calendar
description: Lark Calendar & Meetings operations -- event CRUD, freebusy checks, attendee management, meeting minutes, and video conference reserves. Use when scheduling, querying, or managing calendar events and meetings.
metadata:
  version: "1.0.0"
  author: frnd-team
  tags: ["lark", "calendar", "meetings", "events", "scheduling", "freebusy", "minutes", "vc"]
---

# Lark Calendar & Meetings Skill

Use this skill when working with Lark calendar events, meeting scheduling, freebusy checks, meeting minutes, and video conference reserves.

## Available Tools

### Calendar Events

| Tool | Purpose | Key Args |
|------|---------|----------|
| `list_events` | List events on a calendar (time-range, paginated, or incremental sync) | `calendar_id`, `start_time`, `end_time`, `anchor_time`, `page_token`, `sync_token` |
| `get_event` | Get full details of a single event | `calendar_id`, `event_id` |
| `create_event` | Create a new event on a calendar | `calendar_id`, `summary`, `start_time_timestamp`, `end_time_timestamp`, `timezone` |
| `patch_event` | Update specific fields of an existing event | `calendar_id`, `event_id`, + any fields to update |
| `delete_event` | Delete an event from a calendar | `calendar_id`, `event_id` |
| `search_events` | Search events by keyword (requires user token) | `calendar_id`, `query` |
| `reply_event` | RSVP to an event (accept/decline/tentative) | `calendar_id`, `event_id`, `rsvp_status` |

### Attendees

| Tool | Purpose | Key Args |
|------|---------|----------|
| `create_attendees` | Add participants or book meeting rooms for an event | `calendar_id`, `event_id`, `attendees` |
| `list_attendees` | List all attendees of an event | `calendar_id`, `event_id` |

### Calendars

| Tool | Purpose | Key Args |
|------|---------|----------|
| `get_primary_calendar` | Get the user's primary calendar (returns calendar_id) | -- |
| `list_calendars` | List all subscribed calendars | `page_size`, `page_token` |

### Freebusy

| Tool | Purpose | Key Args |
|------|---------|----------|
| `list_freebusy` | Check busy/free status for a user or meeting room | `time_min`, `time_max`, `user_id_to_query` or `room_id` |

### Meeting Minutes

| Tool | Purpose | Key Args |
|------|---------|----------|
| `get_minute` | Get basic info of a Lark Minute (title, duration, URL) | `minute_token` |
| `get_minute_media` | Get download URL for minute audio/video (valid 1 day) | `minute_token` |
| `get_minute_statistics` | Get view stats (PV, UV, viewer list) for a minute | `minute_token` |

### Video Conference Reserves

| Tool | Purpose | Key Args |
|------|---------|----------|
| `apply_reserve` | Book a video conference meeting (requires user token) | `end_time`, `meeting_settings` |
| `get_reserve` | Get details of a reserved meeting | `reserve_id` |
| `update_reserve` | Update a reserved meeting | `reserve_id`, `end_time`, `meeting_settings` |
| `delete_reserve` | Cancel a reserved meeting | `reserve_id` |

## Workflows

### Schedule a Meeting for the User

1. Call `get_primary_calendar()` to get the user's `calendar_id`
2. Call `list_freebusy(time_min, time_max, user_id_to_query=open_id)` to check availability
3. Call `create_event(calendar_id, summary, start_time_timestamp, end_time_timestamp)` to create the event
4. Call `create_attendees(calendar_id, event_id, attendees=[...])` to add participants and/or book meeting rooms

### Check Freebusy and Find Available Slots

1. Call `get_primary_calendar()` to get the user's `calendar_id`
2. Call `list_freebusy(time_min, time_max, user_id_to_query=open_id)` for each participant
3. Analyze the returned busy intervals to find common free slots
4. Present available slots to the user for selection

### Book a Meeting Room

1. Call `list_freebusy(time_min, time_max, room_id=room_calendar_id)` to check room availability
2. Call `create_event(...)` to create the event
3. Call `create_attendees(calendar_id, event_id, attendees=[{"type": "resource", "room_calendar_id": "xxx"}])` to book the room
4. Note: room booking is asynchronous -- check attendee status after a few seconds to confirm

### Schedule a Meeting with Video Conference

1. Call `create_event(calendar_id, summary, start, end, vc_type="vc")` to create event with Lark VC
2. Or call `apply_reserve(end_time, meeting_settings={"topic": "..."})` to reserve a standalone VC meeting
3. Call `create_attendees(...)` to add participants

### View and RSVP to Events

1. Call `list_events(calendar_id, start_time, end_time)` to see upcoming events
2. Call `get_event(calendar_id, event_id)` for full event details
3. Call `reply_event(calendar_id, event_id, rsvp_status="accept")` to RSVP

### Access Meeting Minutes

1. Extract `minute_token` from the minute URL (last 24 characters)
2. Call `get_minute(minute_token)` for title, duration, and URL
3. Call `get_minute_statistics(minute_token)` for view counts
4. Call `get_minute_media(minute_token)` for audio/video download link (valid 1 day)

## Timezone Handling

- All timestamps are **Unix seconds** (not milliseconds)
- Timezones use the **IANA Time Zone Database** format (e.g. `Asia/Shanghai`, `America/New_York`)
- Non-all-day events default to `Asia/Shanghai` if no timezone is specified
- All-day events use `date` field (RFC 3339 format: `2024-01-15`) instead of `timestamp`, with timezone fixed to UTC+0
- When the user mentions a time without timezone, assume their local timezone or `Asia/Shanghai`

## Constraint-Based Scheduling Patterns

### Finding the Best Meeting Time

When a user asks to "find a time that works for everyone":

1. Collect all participant open_ids
2. Define the search window (e.g. next 5 business days, 9am-6pm)
3. For each participant, call `list_freebusy` with the search window
4. Compute the intersection of free intervals across all participants
5. Filter by meeting duration requirement
6. Present the top 3 available slots

### Handling Recurring Events

- Use the `recurrence` field with RFC 5545 RRULE syntax
- Examples: `FREQ=DAILY;INTERVAL=1`, `FREQ=WEEKLY;BYDAY=MO,WE,FR`, `FREQ=MONTHLY;BYMONTHDAY=15`
- Add `COUNT=10` to limit occurrences, or `UNTIL=20241231T235959Z` for an end date
- Do NOT use COUNT and UNTIL together

## Important Notes

- `get_primary_calendar` uses `POST` method (not GET) -- this is correct per the Lark API
- `list_freebusy` uses `tenant_access_token` only (bot token) -- no user token needed
- `search_events` requires `user_access_token` -- will not work with bot-only auth
- VC reserve operations (`apply_reserve`, `get_reserve`, `update_reserve`, `delete_reserve`) all require `user_access_token`
- Meeting room booking is done via `create_attendees` with `type: "resource"`, NOT via the event location field
- Room booking is **asynchronous** -- the initial status may show "needs_action" before confirming
- `list_events` supports three mutually exclusive modes: time-range (start_time/end_time), anchor-based (anchor_time+page_token), or incremental (sync_token). Do NOT mix them
- Event attendee limit is 3000 per event
- Calendar subscription limit is 1000 per user/app

## Search Best Practices

When using `list_events`, `search_events`, or `list_freebusy`:

- **TOTAL BUDGET**: Maximum 8 search/lookup API calls PER TASK
- **PER-QUERY**: Max 3 search calls for a single piece of information
- Each search MUST use a DIFFERENT strategy:
  - ❌ 'today' → 'today' → 'today' (repetitive!)
  - ✅ 'list_events for this week' → 'list_events for next week' → 'search_events by title' (different angle)

**WHEN SEARCH FAILS (2-3 attempts with no results):**
- STOP searching and return results to main agent
- Include: what you searched for, what time range you checked
- Let the main agent decide next steps

**WHEN YOU ALREADY FOUND IT:**
- If events were found, do NOT search again with different parameters
- Use what you found
