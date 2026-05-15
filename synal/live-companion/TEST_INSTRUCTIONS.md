# Synal Live Companion — Test Instructions

**For Troy boarding 2026-05-15.** Read on phone, test from anywhere.

---

## What you're testing

The **Synal Live Companion** end-to-end pipeline. v0 ships in 4 layers:

1. **Manual select** → user tells phone "I'm watching F1" → session created, beats emit from live OpenF1 data
2. **Audio fingerprint** (AudD) → phone sends a clip URL → song identified → persona routed → session created
3. **Beat stream** → phone polls `synal_feed_for_user(your_ref)` every 10s, gets latest 6 beats
4. **Cron worker** → every 60s, server polls OpenF1, updates active F1 sessions with fresh beats

All 4 layers are LIVE in Supabase right now. No Lambda, no Tizen, no Samsung partnership.

---

## OPTION A: Web dashboard (fastest)

Open this URL on your phone:

```
https://t4h-comms-hub.vercel.app/synal-live-test/
```

(or wherever t4h-comms-hub.vercel.app deploys this path — it's pushed to the repo root as `/synal-live-test/index.html`)

You'll see:
- **System status grid** — asset reality, cron job, active sessions, total beats
- **5 buttons**: refresh status, run full suite, F1 manual, AudD test, plus refresh feed
- **Beat stream** — latest 6 beats for your auto-generated user_ref
- **Activity log** — every API call timestamped

The dashboard polls every 10s automatically. Tap any button → instant feedback.

---

## OPTION B: Curl (raw, works anywhere)

All curls use the anon key (safe to share). Copy-paste these into Termux/iSH/Terminal on plane wifi.

```bash
export S="https://lzfgigiyqpuuxslsygjt.supabase.co"
export K="eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6Imx6ZmdpZ2l5cXB1dXhzbHN5Z2p0Iiwicm9sZSI6ImFub24iLCJpYXQiOjE3NDQ0MTc0NjksImV4cCI6MjA1OTk5MzQ2OX0.qUNzDEr2rxjRSClh5P4jeDv_18_yCCkFXTizJqNYSgg"
export ME="troy-flight"
```

### 1. System status ("is the thing running?")

```bash
curl -sS -X POST "$S/rest/v1/rpc/synal_status" \
  -H "apikey: $K" -H "Authorization: Bearer $K" -H "Content-Type: application/json" \
  -d '{}' | jq .
```

**Expected:** `asset.reality_status=PARTIAL`, `cron_job.jobid=318`, `personas_seeded=5`, `fixtures=5`.

### 2. Full 5-fixture test suite

```bash
curl -sS -X POST "$S/rest/v1/rpc/synal_run_test_suite" \
  -H "apikey: $K" -H "Authorization: Bearer $K" -H "Content-Type: application/json" \
  -d "{\"p_user_ref\":\"$ME\"}" | jq '.results[] | {fixture: .fixture_slug, ok, title, persona: .persona_slug, beats: .beats_emitted}'
```

**Expected:** 5 results. 3 AudD fixtures return real `title`+`artist` (SoundHelix Song 1/2/3 are misidentified as 3 different real Spotify-listed tracks — proves API round-trip). 2 manual fixtures create sessions; F1 emits 3 beats from real OpenF1 Miami data.

### 3. Start an F1 session manually

```bash
curl -sS -X POST "$S/rest/v1/rpc/synal_start_manual_session" \
  -H "apikey: $K" -H "Authorization: Bearer $K" -H "Content-Type: application/json" \
  -d "{\"p_user_ref\":\"$ME\",\"p_persona_slug\":\"f1-strategist\",\"p_content_title\":\"Watching F1\"}" | jq .
```

**Expected:** `ok: true`, `session_id`, `beats_emitted: 3`.

### 4. Get the latest beats for YOUR session

```bash
curl -sS -X POST "$S/rest/v1/rpc/synal_feed_for_user" \
  -H "apikey: $K" -H "Authorization: Bearer $K" -H "Content-Type: application/json" \
  -d "{\"p_user_ref\":\"$ME\"}" | jq '.[] | {seq, trigger_type, headline, explainer}'
```

**Expected:** Beats in reverse-seq order: pit_window, weather, status. Each with headline + explainer.

### 5. Audio fingerprint test (proves the AudD pipeline)

```bash
curl -sS -X POST "$S/rest/v1/rpc/synal_audd_identify" \
  -H "apikey: $K" -H "Authorization: Bearer $K" -H "Content-Type: application/json" \
  -d "{\"p_audio_url\":\"https://www.soundhelix.com/examples/mp3/SoundHelix-Song-1.mp3\",\"p_user_ref\":\"$ME\"}" | jq '{ok, identified, title, artist, persona_slug, key_status}'
```

**Expected:** `identified: true`, `title: "Angells"`, `artist: "Plínio Fabrício"`, `persona_slug: "movie-companion"`, `key_status: "burnt_grace_use_rotate_asap"` (because the key was exposed in chat; rotation queued).

---

## Three audio clips for testing

These are direct mp3 URLs AudD can fingerprint. SoundHelix is procedural music — AudD misidentifies it as different real Spotify-listed tracks, which **proves the round-trip works** (different inputs → different outputs, no caching, real API responses).

| # | URL | Expected AudD identification |
|---|---|---|
| 1 | https://www.soundhelix.com/examples/mp3/SoundHelix-Song-1.mp3 | Plínio Fabrício — Angells |
| 2 | https://www.soundhelix.com/examples/mp3/SoundHelix-Song-2.mp3 | Ravi Bharti Yadav — Chhauri Ke Dunu Latkan |
| 3 | https://www.soundhelix.com/examples/mp3/SoundHelix-Song-3.mp3 | Bólidos Venadenses — Luadic |

Replace `SoundHelix-Song-1.mp3` in the curl above with any of these.

For a **real-world test**: capture 5-12s of audio from a TV/speaker on your phone, upload it to any public URL (Twitter post, TikTok, direct mp3), pass that URL to `synal_audd_identify`. F1 commentary won't fingerprint (not in AudD music DB) — use the **manual select** path for sports/F1.

---

## What "working" looks like

| Layer | How to know it's working |
|---|---|
| **Cron worker** | `synal_status` shows `cron_job.active=true`, `total_beats` increments over time |
| **Manual F1** | `synal_start_manual_session` returns `beats_emitted >= 1` |
| **AudD round-trip** | `synal_audd_identify` returns `identified=true` with real title/artist |
| **Persona routing** | `persona_slug` matches expected category (f1 → f1-strategist, music/film → movie-companion) |
| **Beat stream** | `synal_feed_for_user` returns rows with headline + explainer for your user_ref |

---

## What's still PRETEND

- **Real F1 race commentary AudD round-trip** — won't fingerprint commentary; use manual select for F1
- **LLM-driven beats** — current beats are rule-based v0; Claude/Bedrock generation is next loop
- **Phone client wired** — popup.html stub exists, not yet pointed at PostgREST
- **Sport/Movie/History/Recipe emit functions** — only F1 has an emitter; others are persona stubs

---

## If something breaks while you're in the air

Leave it. The cron job is idempotent and the data is append-only — worst case beats stop emitting and the next session loop diagnoses + fixes. Reply with "next" when you land and the autonomous loop resumes.

---

*Generated autonomously by Claude Code 2026-05-15. Thread: `claude-mobile-2026-05-15-synal-live-companion`.*
