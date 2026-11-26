# tasks.md — Project Tasks & Milestones

## Milestone 0 — Fork & Bootstrap (done-ish)
- [x] Fork immich-app/immich → ZTechGuy/meme-vault
- [x] Rename repo to meme-vault
- [x] Clone locally and verify build
- [ ] Add `upstream` remote for syncing immich
- [ ] CI: build & push images on main
- [x] Docker compose with Traefik labels for memes.marinfam.cloud

## Milestone 1 — Rebrand & Baseline (MVP foundation)
- [ ] Rebrand web: title, logo, favicon, colors, copy → "Meme-Vault"
- [ ] Rebrand server: app name, emails, defaults
- [ ] Verify full stack runs via docker-compose (server+web+ml+db+redis)
- [ ] Smoke test: upload image, view thumbnail, search text

## Milestone 2 — Meme-optimized UX
- [ ] Dense grid view (compact density default)
- [ ] Hover-to-preview (image zoom, video autoplay muted)
- [ ] Upload: paste-from-clipboard, drag-drop, progress
- [ ] Command palette (Cmd+K)
- [ ] Asset viewer with keyboard nav
- [ ] Favorites, archive, trash flows verified

## Milestone 3 — Search (core value)
- [ ] Confirm OCR pipeline runs on upload (ML service)
- [ ] Postgres tsvector index on ocr_text
- [ ] CLIP embedding pipeline + pgvector index
- [ ] Unified search endpoint merging OCR + CLIP (RRF)
- [ ] Filter panel (tags, albums, dates, type)
- [ ] Search suggestions (top tags, recent queries)
- [ ] Graceful degradation when ML down

## Milestone 4 — Sharing
- [ ] Copy image to clipboard action
- [ ] Copy link action
- [ ] Public share links with expiry presets
- [ ] Share management UI + revoke
- [ ] Public share viewer (minimal chrome)
- [ ] QR code for share link
- [ ] Album sharing

## Milestone 5 — Dedup & Organization
- [ ] Perceptual hash computation on upload
- [ ] Duplicate grouping endpoint
- [ ] Duplicates review UI (keep/merge/delete)
- [ ] Soft-delete + 30d trash + purge
- [ ] Tag suggestions from OCR keywords
- [ ] Bulk tag/album operations

## Milestone 6 — Admin & Ops
- [ ] Admin dashboard (users, storage, jobs)
- [ ] User quotas
- [ ] Backup job + restore runbook
- [ ] Health checks & probes
- [ ] Rate limiting
- [ ] Observability (logs, metrics)

## Milestone 7 — Deployment
- [ ] Docker images published
- [ ] Flux manifests applied; ingress live at memes.marinfam.cloud
- [ ] TLS verified
- [ ] Backup/restore tested end-to-end
- [ ] Performance: Lighthouse ≥ 90 on grid

## Milestone 8 — Polish / Future
- [ ] PWA + offline cache
- [ ] Mobile share sheet (reuse/extend Immich mobile)
- [ ] Caption overlay editor (meme generator)
- [ ] S3-compatible storage backend
- [ ] Advanced search (date ranges, color, dominant color)
- [ ] Multi-language UI (i18n)

## Tracking Conventions
- Check a box when merged to `main`, not when started.
- Add subtasks as nested bullets.
- Update this file every session.
- Keep milestones ordered; don't skip.
