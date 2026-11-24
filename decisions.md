# decisions.md — Decision Log (Human Only)

**This file is for your personal reflection. Do not share with AI agents.**

## [2026-11-24] Fork immich vs. Build custom meme app

**Problem:** Need a searchable meme vault for household collection. Existing solutions (Google Photos, Immich, PhotoPrism) designed for photos, not meme-centric UX.

**Options:**
1. Build from scratch (React + Node + Postgres)
2. Fork/customize an existing solution
3. Use a third-party SaaS

**Choice:** Fork Immich.

**Reasoning:**
- Immich already has: storage, thumbnails, transcoding, ML (CLIP, OCR), web UI, mobile, database schema, auth, deployments.
- 330MB codebase, mature. Saves 6+ months of scaffolding.
- Self-hosted, privacy-first (aligns with home k3s setup).
- NestJS + SvelteKit + FastAPI stack well-known to me.
- Risk: Immich updates fast; rebase cost. Mitigation: keep changes minimal, track custom features separately.

**Outcome:** (In progress)

## [2026-11-24] Database: Postgres + pgvector vs. Elasticsearch

**Problem:** Need full-text + semantic search. Postgres-only or separate search service?

**Options:**
1. Postgres + pgvector (single DB, GIN index for full-text, HNSW for vectors)
2. Elasticsearch (separate, battle-tested for search)
3. Milvus (vector-only DB)

**Choice:** Postgres + pgvector.

**Reasoning:**
- Immich uses Postgres; inheritance from fork.
- pgvector mature enough; HNSW index solid.
- Single DB = simpler ops for household scale.
- Full-text + vector in same DB; no cross-service query.

**Outcome:** (In progress)

## [2026-11-24] Deployment: k3s + flux vs. managed hosting

**Problem:** Where to run Meme-Vault?

**Options:**
1. k3s cluster (emmaryhome.stack, existing)
2. Docker Compose (single machine, simpler ops)
3. Kubernetes managed (AWS EKS, etc.)

**Choice:** k3s + flux.

**Reasoning:**
- Already have k3s cluster + flux for other services.
- Learning curve low; reuse existing patterns.
- Self-hosted + privacy.
- Incremental cost low.

**Outcome:** (In progress)
