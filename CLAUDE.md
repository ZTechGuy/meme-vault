# CLAUDE.md — Code Style & Architecture Rationale

## Code Style

**Naming:**
- Server (NestJS): `camelCase` for methods/properties, `PascalCase` for classes/DTOs, `UPPER_SNAKE_CASE` for constants.
- Web (SvelteKit): `camelCase` for functions/stores, `PascalCase` for components, same constants.
- Python (ML): `snake_case` per PEP8; `PascalCase` for classes.
- SQL: `snake_case` for tables/columns; migrations named `YYYYMMDDHHMMSS_<description>.sql`.

**Comments:**
- JSDoc for public APIs (NestJS controllers, service signatures).
- Avoid inline comments; prefer self-explanatory names. Only comment "why" (not "what"), e.g., why a weird calculation or workaround.

**Line length:** 100 characters max (enforced by prettier/black).

**Formatting:** Prettier (web) + Black (ML) + ESLint + clang-format configured. Run via `mise`.

## Architecture Decisions & Rationale

### 1. Monorepo (pnpm workspaces + Python + Rust)
**Why:** Shared types, single CI pipeline, coordinated releases. Immich pattern. Monorepo cost (slow installs) << benefit (consistency) for a household app.

### 2. NestJS (server)
**Why:** TypeScript + strong DI + built-in validation/guards. Reused from Immich. Mature. Drawback: heavyweight; acceptable here.

### 3. SvelteKit (web)
**Why:** Immich choice. SSR by default, type-safe routes, excellent DX. Drawback: smaller ecosystem; mitigated by Immich's battle-tested setup.

### 4. FastAPI (ML service)
**Why:** Fast, async, easy to iterate on CLIP/OCR. Immich standard. Separate process = resilient (web stays up if ML crashes).

### 5. Postgres + pgvector (search backend)
**Why:** pgvector = HNSW index for semantic search. Full-text via tsvector. Single DB = simpler ops. Alternative (Elasticsearch) adds operational burden; overkill for small meme collection.

### 6. Redis (job queue)
**Why:** BullMQ; simple, proven, fast. Upload processing, ML jobs, dedup scans all queued.

### 7. File storage (local FS, S3-ready later)
**Why:** MVP = local `IMMICH_UPLOAD_LOCATION` PVC. S3 abstraction layer ready if storage needs grow.

### 8. JWT + httpOnly cookies (auth)
**Why:** Immich pattern. Stateless; immune to Redis loss. CSRF via SameSite=Strict. No refresh-token rotation needed for household use.

## Patterns to Enforce

### Upload Validation
- Mime allow-list: image/{jpeg,png,webp,gif}, video/{mp4,webm,quicktime}.
- Size: images ≤ 100 MB, videos ≤ 500 MB (config), total user quota enforced.
- Virus scan: optional (clamav) in future; skip MVP.

### Error Handling
- Server: throw custom exceptions (BadRequestException, UnauthorizedException, etc.); NestJS filters transform to 400/401/409/500 + JSON `{statusCode, message, error}`.
- Web: catch and show toast (from server message, do not add extra UI copy).
- ML: return 400 (bad input) or 500 (service); retry logic in BullMQ.

### Async Jobs
- All heavy lifting (thumbnails, transcoding, OCR, CLIP, pHash, dedup scans): enqueued.
- Job status persisted; UI polls or uses WebSocket for progress.
- Failed jobs: retry up to 3 times with exponential backoff; admin dashboard surfaces dead-letter queue.

### API Response Format
```json
{
  "data": { /* or array */ },
  "pagination": { "total": 1000, "page": 1, "pageSize": 20 },
  "error": null /* or {statusCode, message} */
}
```

### Type Safety
- Server DTOs match client schema; generated OpenAPI. No "any" types (tsc strict mode).
- Web: SvelteKit routes type-safe, `src/lib/api.ts` for fetch wrapper.
- ML: Pydantic schemas for request/response.

### Testing
- Server: unit (services) + integration (e2e via Nest TestingModule).
- Web: component (Vitest) + integration (Playwright).
- ML: unit on models, integration on pipeline.
- Target: 70%+ coverage, all job workers tested.

### Logging
- Structured JSON (pino/winston server, structlog Python).
- Include request ID (x-request-id header) for correlation.
- Log level: debug/info/warn/error; debug silent in production.
- Sensitive data (passwords, tokens): never logged; scrub via logger middleware.

### Database
- Migrations tracked in `server/src/migrations/`; run on app start (Typeorm).
- No manual SQL edits; schema via ORM entities.
- N+1: use `leftJoinAndSelect`, batch queries.

### Caching
- HTTP: ETag + Cache-Control headers on thumbnails, public assets.
- Redis: session, job status, (optional) search query results.
- Never cache user-specific data longer than session TTL.

## Anti-Patterns (Don't)

- Hand-edit generated files (OpenAPI, SQL, Dart).
- Commit secrets (use .env, Kubernetes secrets, flux sealed-secrets).
- Synchronous ML calls in request path; always queue.
- Mutable shared state in workers; each job = fresh context.
- N+1 queries; always prefetch relations.
- Logging sensitive data (tokens, emails in logs, etc.).
- Comment every line; code should explain itself.
- Global state in web components (use stores, props).

## Performance Priorities

1. Grid render: virtual scroll, lazy thumbnails (Intersection Observer).
2. Search: indexed columns (tsvector GIN, pgvector HNSW).
3. Upload: parallel job workers; server streams response as processing completes.
4. ML: batch CLIP/OCR when queue backlog exists (ONNX quantization for edge devices).
5. Lighthouse: defer non-critical JS, minify, use modern compression (Brotli).

## Deployment & Ops

- Environment variables: `.env.example` committed (no secrets); secrets via Kubernetes/flux.
- Versioning: semantic (MAJOR.MINOR.PATCH); tags on main, CI auto-builds.
- Rollout: Kubernetes rolling deployment (gradual traffic shift); Flux reconcile interval 10s.
- Database: backup daily (pg_dump + object storage); restore tested quarterly.

## Tech Stack Links (for reference)

- NestJS: https://docs.nestjs.com
- SvelteKit: https://kit.svelte.dev
- FastAPI: https://fastapi.tiangolo.com
- pgvector: https://github.com/pgvector/pgvector
- BullMQ: https://docs.bullmq.io
- Immich source: https://github.com/immich-app/immich

## Fork Maintenance

- Upstream remote: `git remote add upstream https://github.com/immich-app/immich.git` (already done).
- Sync strategy: `git fetch upstream && git rebase upstream/main` (conservative; avoid merge commits).
- Custom changes: isolated in feature branches (`feat/meme-*`), merged to main without squashing (preserves history for rebase).
- Conflict resolution: manual; prioritize Immich pattern when in doubt.
