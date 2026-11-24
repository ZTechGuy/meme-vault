# planning/diagrams.md — System Architecture & Data Flow

## High-Level Architecture

```
                     ┌──────────────────────────────┐
                     │  Ingress (TLS) memes.         │
                     │  marinfam.cloud               │
                     └──────────────┬───────────────┘
                                    │
                     ┌──────────────▼───────────────┐
                     │  Web (SvelteKit)  :3000      │
                     │  SSR + static assets         │
                     └──────────────┬───────────────┘
                                    │ /api
                     ┌──────────────▼───────────────┐
                     │  Server (NestJS)  :2283      │
                     │  REST + WS + job queue       │
                     └──┬──────┬──────┬──────┬─────┘
                        │      │      │      │
            ┌───────────▼┐  ┌──▼──┐ ┌▼────┐ ┌▼────────────┐
            │ Postgres   │  │Redis│ │ FS  │ │ ML service  │
            │ (+pgvector)│  │     │ │orig │ │ (FastAPI)   │
            │ metadata,  │  │jobs │ │thumbs│ │ CLIP, OCR,  │
            │ vectors,   │  │     │ │trans │ │ face (opt)  │
            │ text index │  │     │ │codes │ │             │
            └────────────┘  └─────┘ └─────┘ └─────────────┘
```

## Monorepo Layout (forked from Immich)

```
meme-vault/
├── server/            # NestJS API, jobs, storage
├── web/               # SvelteKit frontend
├── mobile/            # Flutter (deferred, keep upstream)
├── machine-learning/  # Python FastAPI: CLIP, OCR, face
├── packages/          # sdk, cli, plugin-core, scripts
├── open-api/          # OpenAPI spec (generated)
├── deployment/        # docker-compose, k8s manifests
├── docker/            # Dockerfiles
└── docs/              # (this ai-coding-docs live at repo root per framework)
```

## Data Flow — Upload

```
Client -> Server /api/assets (multipart)
  -> validate auth + quota
  -> write original to FS (content-addressed path by asset id)
  -> insert asset row (status=processing)
  -> enqueue: thumbnail, metadata, (video) transcode, OCR, CLIP, pHash
  -> BullMQ (Redis) workers process
  -> update asset status=ready; emit WS event
  -> Client refreshes grid
```

## Data Flow — Search

```
Client query -> Server /api/search
  -> parse query + filters
  -> parallel:
       a) Postgres full-text (OCR tsvector) -> candidate ids
       b) CLIP: encode query -> vector search (pgvector) -> candidate ids
  -> merge via RRF (reciprocal rank fusion)
  -> apply filters (tags, albums, dates, type)
  -> paginate -> return asset ids + highlights
  -> Client fetches thumbnails
```

## Data Flow — ML

```
Server job -> HTTP to ML service /predict (image bytes or path)
  -> CLIP: returns embedding (float[])
  -> OCR: returns text + boxes
  -> pHash: computed server-side (sharp/phash) or ML
  -> Server stores embedding in pgvector, OCR text in tsvector, pHash in asset row
```

## Database Schema (overview, reuses Immich)

Core tables (keep): `assets`, `users`, `albums`, `album_assets`, `asset_faces`, `person`, `tags`, `asset_tags`, `shared_links`, `sessions`, `api_keys`, `system_config`, `jobs`.

Meme-Vault additions:
- `assets.perceptual_hash` (text/bytea) + index for dedup grouping.
- `assets.ocr_text` (text) + tsvector generated column + GIN index.
- `assets.clip_embedding` (vector) with pgvector index (HNSW).
- `tags` extended with `source` (manual | ocr-suggested | ml) and `confidence`.
- `asset_duplicates` (group_id, asset_id, chosen) for review state (optional; can compute on demand).
- `share_links` extended with `allow_download`, `password_hash` (optional).

## External Integrations

- **Postgres 14+ with pgvector** (required for semantic search).
- **Redis** (job queue).
- **ML service** (Python) — optional but recommended; graceful degradation.
- **SMTP** (optional) for password reset/invites.
- **Object storage** (optional future): S3-compatible; MVP uses local FS.
- **Ingress/TLS** via cluster.

## Deployment (k3s via flux)

```
flux repo (emmaryhome.stack) references image tags from this repo's CI
  -> Deployment: web, server, ml, redis, postgres (or external DB)
  -> PVCs: library (originals), thumbs, pgdata, ml-cache
  -> Secrets: DB creds, JWT secret, (optional) OAuth, SMTP
  -> Ingress: memes.marinfam.cloud -> web service
```

## Backup & Restore

- Postgres dump (pg_dump) scheduled daily → object storage/PVC backup.
- Library volume snapshot (rsync/restic) daily.
- Restore runbook in docs/ (TBD): restore DB then library; verify asset counts & sample search.

## Observability

- Structured JSON logs (pino) from server/web.
- `/api/health` + `/api/server/health` for probes.
- Job queue metrics (BullMQ) exposed; admin UI surfaces failures.
- (Optional) Prometheus metrics endpoint.

## Security Boundaries

- Auth middleware on all `/api/*` except `/api/auth/*`, `/api/share/:token` (scoped), health.
- Share tokens: high-entropy, hashed at rest, scoped to single asset/album, expiry, optional password.
- Upload validation: mime allow-list (images + common video), size limits, virus scan (optional/clamav later).
- Rate limits: upload, search, auth endpoints.
- No secrets in repo; use env/flux secrets.
