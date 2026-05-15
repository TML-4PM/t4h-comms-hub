# STATthat · Test Instructions

## Open the demo (60 seconds)

1. **TV view** (laptop / projector / cast to TV):
   https://t4h-comms-hub.vercel.app/statthat/demo/?room=SOUTHS01

2. **Phone view** (your phone):
   https://t4h-comms-hub.vercel.app/statthat/mobile/?room=SOUTHS01
   *(or scan the QR on the TV view)*

You should immediately see:
- TV: Souths 12 - Dolphins 8, Q2 28:14, three cards, participation ticker, QR code
- Phone: same scoreboard, same three cards, reaction buttons + comment input

## What to try

| Test | How | Pass criteria |
|---|---|---|
| Cards render | Open TV + phone | 3 cards visible on both |
| Send a reaction | Tap 🐰 on phone | Within 4s, your handle + 🐰 appears on TV ticker |
| Send a comment | Type 'huge run' → Send | Within 4s, appears on TV ticker AND phone feed |
| Multiple devices | Open phone view on 2nd device with different handle | Both see each other on TV ticker |
| Room not found | Visit `?room=DOESNOTEXIST` | Mobile shows join screen |
| Persistence | Refresh both | Last 20 messages still in feed |

## Raw API testing

```bash
export S="https://lzfgigiyqpuuxslsygjt.supabase.co"
export K="eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6Imx6ZmdpZ2l5cXB1dXhzbHN5Z2p0Iiwicm9sZSI6ImFub24iLCJpYXQiOjE3NDQ0MTc0NjksImV4cCI6MjA1OTk5MzQ2OX0.qUNzDEr2rxjRSClh5P4jeDv_18_yCCkFXTizJqNYSgg"
```

### Room state
```bash
curl -sS -X POST "$S/rest/v1/rpc/statthat_room_state" \
  -H "apikey: $K" -H "Authorization: Bearer $K" -H "Content-Type: application/json" \
  -d '{"p_code":"SOUTHS01"}' | jq
```

### Post a reaction
```bash
curl -sS -X POST "$S/rest/v1/rpc/statthat_post_participation" \
  -H "apikey: $K" -H "Authorization: Bearer $K" -H "Content-Type: application/json" \
  -d '{"p_code":"SOUTHS01","p_handle":"@you","p_kind":"reaction","p_payload":"huge hit","p_emoji":"🔥"}'
```

## v0.1 honest limits

- **Match state is static.** Score and clock won't move unless an admin updates it. v0.2 wires live NRL data.
- **Cards are hand-seeded.** Three cards exist as proof of the room/card shape. v0.3 generates cards from live state via Claude.
- **No moderation.** Anonymous anyone-can-post via anon key. Acceptable for demo; must be added before public rollout.
- **No QR scan flow yet.** Phone joins via URL param `?room=`. The TV displays a QR rendered via api.qrserver.com. v0.2 generates QR locally and adds scan-to-pair.

## When something breaks

- TV blank → check URL has `?room=SOUTHS01`
- Phone stuck on join → room code wrong (must be uppercase, no spaces)
- Reactions not appearing on TV → wait 4 seconds (poll interval)
- 401 → anon key rotated, re-fetch from this doc
