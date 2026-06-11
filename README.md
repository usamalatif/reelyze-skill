# Reelyze Skill 🎬

An agent skill that gives **Claude, Cursor, or any AI agent** eyes on short-form video. It calls the [Reelyze](https://getreelyze.com) API to analyze any Instagram Reel, TikTok, or YouTube Short frame-by-frame — hook strength, retention, the exact second viewers drop off — plus transcribe, download, extract audio, and generate scripts and content ideas.

> Built by [Usama Latif](https://www.linkedin.com/in/usamaaaa/), founder of [Reelyze](https://getreelyze.com).

## What it does

Given a public Reel / TikTok / Short URL (or a topic), an agent with this skill can:

- **Analyze** — full performance report: hook score, first-3-second retention, scene-by-scene signals, exact drop-off seconds, verdict + fixes *(paid)*
- **Transcribe** — spoken-word transcript *(free, metered)*
- **Download** — clean MP4 link *(free, metered)*
- **Extract audio** — MP3 link *(free, metered)*
- **Generate a script** — hooks + timed hook→body→CTA, caption, hashtags, shot list *(paid)*
- **Generate content ideas** — a batch of niche-tuned ideas with hooks *(paid)*

The free tools work at 50 calls/day per key.

## Quick start

1. Create an API key in the [Reelyze dashboard](https://getreelyze.com) → **API keys** (format `rk_live_...`).
2. Set it in your environment:
   ```bash
   export REELYZE_API_KEY=rk_live_xxx
   ```
3. Drop `SKILL.md` into your agent (Claude, Cursor, or any agent that loads skill files). That's it — no SDK, no install. The API is plain HTTP behind a Bearer key with open CORS.

Then just ask your agent:

```text
> audit this reel and tell me why it flopped: <url>
→ Verdict: strong concept, buried payoff.
→ Hook: weak — lost ~62% by 0:03.
→ Drop-off at 0:11: 3s of B-roll with no narration.
→ Fix: cold-open on the result; cut the intro.
```

## How it works

- **Video tools** (analyze, transcript, download, audio) are **async**: submit a job, poll `GET /v1/jobs/{id}` until `completed`.
- **Content tools** (script, ideas) are **synchronous**: the response is the result.
- Base URL: `https://api.getreelyze.com` (override with `REELYZE_BASE_URL`).

Full endpoint reference, request/response shapes, and recipes are in [`SKILL.md`](./SKILL.md). A walkthrough with code is on the [Reelyze blog](https://getreelyze.com/guides/reelyze-api-agent-skill).

## License

MIT-0 — use it however you like.

---

**Reelyze** — know why your Reels flop, before you post. → [getreelyze.com](https://getreelyze.com)
