# STATthat

> The match plays. The crowd reacts. The phone explains what to watch, what's coming, and what's weird.

**Project:** STATthat-001 · **Sport:** NRL · **Demo:** Rabbitohs vs Dolphins · Round 11

## Open the demo

- **TV view**: https://t4h-comms-hub.vercel.app/statthat/demo/?room=SOUTHS01
- **Phone view**: https://t4h-comms-hub.vercel.app/statthat/mobile/?room=SOUTHS01

## What this is

TV holds the match + participation feed. Phone is the personal lens with three cards:

1. **Watch This** — what to keep your eye on right now
2. **Prediction** — what's likely in the next 2-3 sets
3. **Weird Stat** — a fact that reframes what you just saw

Anyone in the room can drop reactions, comments, or mini-predictions. They flow across the TV and into the phone.

## Architecture parent

Forked from Synal Live Companion (SYNAL-002). Reuses Supabase + anon RPC chassis. Adds room/card/participation tables and sport-specific persona.

## v0.1 status

- ✅ Demo room SOUTHS01 seeded (Souths 12 – Dolphins 8, Q2 28:14)
- ✅ 3 cards live
- ✅ Participation feed live (8 seeded messages, accepts new via anon RPC)
- ✅ TV + phone views
- ⏳ Live NRL data feed (v0.2)
- ⏳ QR scan-to-join (v0.2)
- ⏳ LLM-generated cards from live match state (v0.3)

## Slogans

- The match plays. The phone explains.
- The crowd reacts. The phone routes it.
- Three cards. One match. Everyone in the room.
