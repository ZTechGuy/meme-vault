# memory.md — Long-term Learnings

## Bugs Fixed

(To be populated as issues emerge.)

## Patterns That Worked

(To be populated as we iterate.)

## Integration Gotchas

### Immich Upstream Churn
- Immich updates fast; rebase-on-main frequently causes conflicts in `web/src/routes/`, `server/src/controllers/`, `machine-learning/`.
- **Solution:** Keep custom changes minimal; use feature branches; track merge conflicts in this doc when they're tricky.

### pgvector Index Performance
- Initial HNSW index build on large datasets (> 10k assets) can lock the table briefly.
- **Solution:** Build index concurrently during off-hours; test restore time in pre-prod.

### CLIP Embedding Latency
- First run of CLIP model loads from disk; subsequent queries fast.
- **Solution:** Pre-warm model in ML service startup; consider model quantization if edge hardware.

### ML Service Crashes
- If FastAPI crashes, server keeps accepting uploads but search degraded (falls back to text-only).
- **Solution:** Health check on `/api/health` endpoint; restart policy (Kubernetes: always); alerting on restarts.

### File Deduplication at Scale
- Computing perceptual hashes on thousands of images serially is slow.
- **Solution:** Batch pHash jobs (10 at a time); consider GPU acceleration if available.

## Code Examples (Reusable Patterns)

(To be populated with snippets of patterns that worked well.)
