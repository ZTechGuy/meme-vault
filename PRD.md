# PRD.md — Meme-Vault

## Product Overview

**Meme-Vault** is a self-hosted media vault purpose-built for **memes**: images and short videos that users save, collect, and re-share. It is a fork of [Immich](https://github.com/immich-app/immich), re-skinned and extended so that meme-specific workflows (fast capture, OCR-driven text search, semantic/natural-language search, one-tap sharing, deduplication, and meme-pack collections) are first-class.

It is deployed for a single household/community at **https://memes.marinfam.cloud**, running on the existing k3s/flux infrastructure (`emmaryhome.stack`). The backend reuses Immich's high-performance asset pipeline (upload, thumbnails, transcoding, EXIF, ML embeddings, vector search) so we inherit years of optimization rather than rebuilding storage.

**For whom:** A small group (family/friends) who accumulate memes across chat apps and want a searchable, shareable archive instead of a phone camera roll.

**Why:** Existing photo managers (Immich, PhotoPrism, Google Photos) are photo-centric: slow grids, no meme-aware search, no share-to-clipboard, no dedup across re-saves. Chat apps lose searchability. Meme-Vault optimizes for the meme lifecycle: capture → find → share.

## User Personas

- **Collector (primary):** Saves memes from chats/Reddit/Twitter. Wants instant upload from clipboard/share sheet, automatic OCR + tagging, and "find that meme I saw last week" search.
- **Sharer:** Finds a meme and re-shares it in 1-2 taps (copy image, copy link, share sheet). Cares about speed and clean URLs.
- **Curator:** Builds themed collections ("reaction memes", "work memes") and removes duplicates. Cares about bulk ops and tagging.
- **Admin (owner):** Manages users, storage, backups, ML models. Single admin initially.

## Core Features (MVP)

1. **Ingest:** Upload images/videos via web UI, drag-and-drop, paste-from-clipboard, and (later) mobile share sheet. Bulk upload.
2. **Storage & processing:** Thumbnails, transcoding for video, EXIF, content-addressed dedup (hash-based).
3. **Search:**
   - Full-text over OCR text extracted from memes.
   - Semantic/natural-language search via CLIP embeddings ("funny cat with sunglasses").
   - Tag, album, date, and person (face) filters.
   - Fuzzy + typo-tolerant.
4. **Organize:** Albums (meme packs), tags, favorites, archive.
5. **Share:** Public share links with expiry, copy-image-to-clipboard, download, QR. One-tap "copy link".
6. **Dedup:** Detect and merge near-duplicate memes (perceptual hash).
7. **Auth:** Email/password + OAuth (optional), multi-user with sharing.
8. **Admin:** Storage stats, user management, backup/restore, job queue UI.

## Scope Boundaries / Out of Scope (MVP)

- No meme *generator*/editor (caption overlay editor) in MVP — capture & search first.
- No public discovery/feed or social features (likes from strangers). Sharing is link-based only.
- No mobile-native app in MVP; responsive web + PWA first. (Immich mobile app may be reused later.)
- No monetization, ads, or analytics beyond self-hosted.
- No AI image *generation* (we search existing memes only).
- Face recognition retained from Immich but de-prioritized (memes rarely need it; keep for people memes).

## Success Criteria (measurable)

- Upload-to-searchable latency < 30s for a single meme on target hardware.
- Search returns relevant results for a natural-language query in < 1s for ≤ 50k assets.
- Duplicate detection catches ≥ 95% of exact re-saves (hash match) and ≥ 80% of resized/re-compressed variants (pHash).
- Share link creation < 1s; image copy-to-clipboard < 500ms.
- Zero data loss across upgrades (backup/restore tested).
- Lighthouse performance ≥ 90 on meme grid view.

## Non-Functional Requirements

- **Performance:** Virtualized grid; lazy thumbnails; CDN-friendly caching headers.
- **Security:** Auth on all non-public routes; share links scoped + expirable; rate limiting on upload/search; no path traversal; secrets via env/flux secrets, never committed.
- **Privacy:** Self-hosted; no telemetry; ML runs locally (no external AI APIs required). Optional: HF token for model downloads only.
- **Accessibility:** Keyboard navigation, alt text from OCR, sufficient contrast.
- **Reliability:** Health checks; graceful degradation if ML service down (search falls back to text).
- **Observability:** Structured logs; basic metrics; backup verification job.

## Deployment Target

- Host: `memes.marinfam.cloud` (k3s cluster managed by flux `emmaryhome.stack`).
- Images: published from this repo's CI to a registry; deployed via flux.
- Storage: persistent volumes for originals + DB (Postgres) + Redis + ML model cache.
- TLS via cluster ingress.

## Key Differentiators vs. Immich (what we change)

- Rebranding: name, logo, colors, copy ("Meme-Vault").
- Default views optimized for dense meme grids (smaller thumbnails, hover-to-preview).
- OCR + CLIP prioritized; face recognition optional/off by default.
- Share UX: copy image/link, QR, expiry presets tuned for memes.
- Dedup pipeline (perceptual hashing) added.
- Tag suggestions from OCR keywords.
- (Future) Caption overlay editor.

## Risks

- Immich upstream churn: heavy merge cost. Mitigation: keep changes as a patch layer / separate packages where possible; track upstream via `upstream` remote.
- ML resource usage on small node: gate ML behind feature flags; allow disabling face recognition.
- Storage growth: enforce quotas; provide cleanup tools.
