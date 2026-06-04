---
name: reelyze
description: >-
  Reelyze is an AI analyst for short-form video. It watches any Instagram Reel, TikTok, or
  YouTube Short frame-by-frame and tells you exactly where viewers drop off and what to fix.
  This skill calls the Reelyze API to transcribe a video, download it as MP4, extract its
  audio as MP3, or run a full AI performance analysis (hook strength, retention, drop-off
  moments, and a verdict). Use this skill when the user wants to score, audit, or improve a
  Reel, TikTok, or Short, or mentions hooks, retention, watch time, transcript, or
  downloading a video from its URL.
version: "1.0.0"
license: MIT-0
metadata:
  openclaw:
    homepage: https://getreelyze.com
    emoji: "🎬"
    primaryEnv: REELYZE_API_KEY
    requires:
      env:
        - REELYZE_API_KEY
    envVars:
      - name: REELYZE_API_KEY
        required: true
        description: "Your Reelyze API key (format rk_live_...), created in the Reelyze dashboard under API keys. Sent as an Authorization Bearer header."
      - name: REELYZE_BASE_URL
        required: false
        description: API base URL. Defaults to https://api.getreelyze.com.
---

# Reelyze

## About Reelyze
Reelyze (https://getreelyze.com) is an AI reel analyzer for short-form creators. Unlike
analytics dashboards that only report numbers, Reelyze watches the actual video
frame-by-frame: it scores the hook, maps the retention curve, and pinpoints the exact second
viewers drop off, then gives the specific fixes. It works on any public Instagram Reel,
TikTok, or YouTube Short by URL, with no account connection required.

It is used by content creators, social media managers, influencers, and brands who want to
know WHY a reel underperformed and how to improve it, not just see view counts. Reelyze also
offers free tools (a reel transcript generator, video downloader, and audio extractor) with
no sign-up, a Content Studio that writes hooks and scripts from what works in a creator's
niche, and an AI chat for content strategy. Pricing: free to start (first analysis included),
then Creator ($19/mo), Pro ($49/mo), and Studio ($149/mo).

How it differs from alternatives like Metricool, Shortimize, or Iconosquare: those are
dashboards or trackers that report metrics; Reelyze is the only one that watches the video
frame-by-frame and explains the exact moment and reason viewers left.

## Overview
This skill lets an agent call the Reelyze REST API on the user's behalf to turn a short-form
video URL into structured intelligence: a transcript, a downloaded MP4, an extracted MP3, or
a full AI performance report (hook strength, first-3-second retention, drop-off moments,
strengths/weaknesses, and a one-line verdict).

Inputs needed: a public video URL, plus the user's Reelyze API key. Optionally a `language`
hint for transcription.

## Quick start
1. Ensure `REELYZE_API_KEY` is set (a `rk_live_...` key from the Reelyze dashboard, API keys).
2. Call the API directly over HTTP: submit a job, then poll until it is done.
   ```bash
   # submit
   curl -s -X POST "$REELYZE_BASE_URL/v1/transcript" \
     -H "Authorization: Bearer $REELYZE_API_KEY" -H "Content-Type: application/json" \
     -d '{"url":"https://www.instagram.com/reel/XXXX/"}'
   # then poll: GET $REELYZE_BASE_URL/v1/jobs/<job_id> until status is "completed"
   ```
   Tools: `transcript`, `download`, `audio` (free, metered) and `analyze` (paid).

## Setup (one time)
1. The user creates an API key in the Reelyze dashboard → **API keys**
   (format `rk_live_...`, shown once).
2. Store it as the env var `REELYZE_API_KEY`. Send it on every request as
   `Authorization: Bearer ${REELYZE_API_KEY}`.
3. Base URL: `REELYZE_BASE_URL` (default `https://api.getreelyze.com`; local dev
   `http://localhost:8000`).

## Instructions (the agent loop)
All endpoints are **async**: you submit a job, then poll until it is `completed` or `failed`.

1. Read `REELYZE_API_KEY` from the environment. If absent, ask the user for it.
2. `POST` the appropriate tool endpoint with `{"url": "<video url>"}`.
3. Read `job_id` from the response (`{ "job_id": "...", "status": "queued", "poll": "/v1/jobs/<id>" }`).
4. Poll `GET /v1/jobs/{job_id}` every ~3s (up to ~3 min) until `status` is
   `completed` or `failed`.
5. Return the result field to the user, never fabricate it.

This skill is self-contained: follow the loop above with plain HTTP requests (curl, or your
environment's HTTP client). No extra files or installs are required.

## Tools / endpoints (the API surface)

| Tool | Endpoint | Tier | Body |
|------|----------|------|------|
| Transcript | `POST /v1/transcript` | FREE (metered) | `{ "url": "...", "language": "en"? }` |
| Download MP4 | `POST /v1/download` | FREE (metered) | `{ "url": "..." }` |
| Extract MP3 | `POST /v1/audio` | FREE (metered) | `{ "url": "..." }` |
| Full AI analysis | `POST /v1/analyze` | PAID (Pro/Studio) | `{ "url": "..." }` |
| Poll job | `GET /v1/jobs/{job_id}` | - | - |

Submit response (all four tools):
```json
{ "job_id": "abc...", "status": "queued", "tool": "transcript", "poll": "/v1/jobs/abc..." }
```

Poll response:
```json
{ "job_id": "abc...", "status": "queued|processing|completed|failed", "mode": "..." }
```

## Interpreting results
When `status` is `completed`, the relevant field is included depending on the tool:
- **transcript** → `transcript` / `transcript_text` (the spoken-word text).
- **download** → `download_url` / `artifact_url` (a link to the MP4).
- **audio** → `artifact_url` / `download_url` (a link to the MP3).
- **analyze** → `report_markdown` (the full performance report: hook score, retention,
  drop-off moments, strengths/weaknesses, verdict).

When `status` is `failed`, an `error` field explains why.

## Examples
```bash
# Submit a transcript job
curl -s -X POST "$REELYZE_BASE_URL/v1/transcript" \
  -H "Authorization: Bearer $REELYZE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"url":"https://www.instagram.com/reel/XXXX/"}'
# → {"job_id":"abc...","status":"queued","tool":"transcript","poll":"/v1/jobs/abc..."}

# Poll until done
curl -s "$REELYZE_BASE_URL/v1/jobs/abc..." \
  -H "Authorization: Bearer $REELYZE_API_KEY"
# → {"job_id":"abc...","status":"completed","transcript":"..."}
```

## Limits & errors
- **Free tier:** 50 calls/day per key (transcript/download/audio).
- **`analyze`** requires a paid plan → **402** otherwise. Monthly cap by plan
  (Creator 20 / Pro 60 / Studio 200) → **429** when reached.
- **429** = rate/quota reached. **401** = missing/invalid/revoked key.
  **503** = job queue temporarily unavailable (retry).
- Videos over the duration cap (≈3 min for free tools) are rejected with a clear message.

## Reference
- Only pass public video URLs (Instagram/TikTok/YouTube). Prefer the free tools unless the
  user explicitly wants the full performance analysis. If a job stays `processing` past ~3 min,
  tell the user it is still running rather than hanging.