# T4H Comms Hub

Static publication hub for Tech 4 Humanity. Reads from `public.comms_publication`
in Supabase S1, baked at deploy-time into `pubs.json`.

## What it serves

- **External** — public-facing pages (synal extensions, valdocco, ai-sweet-spots, brand)
- **Partner** — partner-restricted pages (supervisor reports, risk packs)
- All HTML rendered inline; gdrive-sourced pages with no body show a "pending fetch" banner.

## Deploy

```bash
vercel --prod --yes
```

Vercel auto-detects as static. No build step.

## Refresh content

Re-bake `pubs.json` from Supabase, then redeploy:

```bash
# from the bridge:
curl -s -X POST "$BRIDGE_URL/lambda/invoke" \
  -H "x-api-key: $BRIDGE_KEY" \
  -d '{"fn":"troy-sql-executor","payload":{"sql":"SELECT json_agg(row_to_json(t)) FROM (SELECT id::text,slug,title,description,audience,source_type,source_ref,tags,meta,byte_size,html_sha256,status,html FROM public.comms_publication WHERE audience IN ('\''external'\'','\''partner'\'') AND is_active ORDER BY audience, slug) t;"}}' \
  | python3 -c "import json,sys; d=json.load(sys.stdin); print(json.dumps(d['rows'][0]['json_agg'], indent=2))" > pubs.json

vercel --prod --yes
```

## Source registry

| table | purpose |
|---|---|
| `public.comms_publication` | canonical publishable HTML, audience-classified |
| `public.comms_publish_log` | append-only audit trail of every publish/deploy |

## RLS

- `service_role` — full access
- `anon` — SELECT where `audience IN ('public','external') AND status='published' AND is_active`

## Audience classification rules

| audience | sources | pattern |
|---|---|---|
| `external` | ui_snippet | `page_key LIKE 'synal-%'` OR `page_key IN ('valdocco','thriving-kids','reading-buddy','widget-catalog')` |
| `external` | gdrive | filename matches `homepage`, `portal`, `brand`, `river`, `merch`, `valdocco`, `ai-sweet*` |
| `partner` | gdrive | filename matches `report`, `risk`, `supervisor` |
| `restricted` | ui_snippet | `page_key = 'biz_property-decision-pack'` |
| `internal` | * | default |

## TODO (next iteration)

- Comms publisher Lambda to fetch GDrive HTML bodies into `comms_publication.html` (16 pending)
- Per-slug static page generation (`/p/<slug>` as actual file vs hash-route)
- Auth gate for `partner` audience (currently rendered same as external)
- DNS: point `comms.tech4humanity.com.au` → this Vercel project
