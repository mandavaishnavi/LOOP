# LOOP - working prototype

An AI executive-function assistant: capture notes -> extract commitments -> schedule -> get reminded.

## Run it

```bash
# 1. put your Sarvam key in the environment (never in a file)
export SARVAM_API_KEY=your_key_here

# 2. start the server (serves the app AND proxies Sarvam)
node server.js

# 3. open
http://localhost:8787
```

`server.js` has no dependencies - plain Node (18+). It does two jobs: serves `loop.html`, and proxies two Sarvam calls so the key stays server-side and CORS is not a problem.

## Files

- `loop.html` - the whole app (UI, calendar, reminders, delivery engine, localStorage persistence). Self-contained.
- `server.js` - static host + two endpoints: `POST /api/extract` (sarvam-105b) and `POST /api/tts` (bulbul:v3).

## What works

- Capture -> extract -> review -> approve. LLM extraction via **sarvam-105b**; falls back to a built-in rule-based extractor if the server or API is unavailable.
- Week calendar with tasks and appointments: create, edit, complete, delete, open detail.
- Multiple reminders per item (1 day / 1 hour / 10 min before, or "at time"), each with its own method (push / voice / both). Scheduled times are shown and recompute automatically when an item is rescheduled.
- Delivery: in-tab browser notification **and** spoken reminder via **bulbul:v3** (browser speech as fallback), with snooze/dismiss and a visible delivery log.
- Everything persists in localStorage and survives a refresh.

## The honest limitation

Reminders are delivered **while the tab is open**. The server-side scheduler + Web Push that would fire them when the browser is closed was deliberately cut to hit the deadline. This is the one acceptance criterion not met; say so rather than implying otherwise.

Also: iOS web push needs an installed PWA, and spoken reminders need the page open (no web platform can play audio while the site is closed).

## API notes (verified against the live API)

- Chat: `POST /v1/chat/completions`, model `sarvam-105b` (sarvam-m is deprecated), `Authorization: Bearer <key>`. Set `max_tokens` - the model spends budget on reasoning and can return empty content otherwise.
- TTS: `POST /text-to-speech`, model `bulbul:v3` (bulbul:v2 is deprecated), header `api-subscription-key`. Speakers valid for v3 include priya, ritu, kavya, aditya, rahul.
- Sarvam sends no CORS headers, so browser-direct calls are blocked - hence the proxy.
