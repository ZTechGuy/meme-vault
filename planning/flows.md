# planning/flows.md — User Flows & State Machines

## Primary Flows

### Flow A — Capture a meme (Upload)
```
[User] -> Open app / paste / drag-drop / share sheet
   -> Upload dialog (file picker / clipboard / URL)
   -> Client compresses? no, send original
   -> POST /api/assets (multipart)
   -> Server: validate -> store original -> enqueue jobs:
        thumbnail, metadata (EXIF), (video) transcode,
        (image) OCR, CLIP embedding, perceptual hash
   -> Asset status: pending -> processing -> ready
   -> Appears in "Recent" grid on completion (WS/poll)
   -> Optional: quick tag prompt
```

### Flow B — Find a meme (Search)
```
[User] types query in global search bar
   -> Debounce 200ms
   -> If empty: show recent + suggestions (top tags)
   -> Text query:
        - Run OCR full-text (Postgres tsvector)  -- fast
        - Run CLIP semantic search (vector)      -- ranked
        - Merge + rerank (RRF) -> results
   -> Filters: tags, albums, date range, type (image/video), favorites
   -> Grid renders thumbnails; click -> viewer
   -> Empty state -> suggest broader query / clear filters
```

### Flow C — Share a meme
```
[User] in viewer or grid hover
   -> Share button
   -> Modal: Copy image | Copy link | Download | QR | Create public link
   -> Copy image: fetch blob -> clipboard (image/png)
   -> Copy link: asset share URL -> clipboard
   -> Public link: create share token (expiry preset: 1h/24h/7d/never)
        -> URL memes.marinfam.cloud/share/<token>
   -> Revoke from Share management
```

### Flow D — Dedup review
```
[Admin/Curator] opens "Duplicates" panel
   -> Server groups assets by pHash (hamming distance <= threshold)
   -> Show side-by-side groups
   -> Actions: keep one & delete others / merge tags / cancel
   -> Confirm -> soft-delete duplicates (recoverable 30d)
```

### Flow E — Collection (Album) management
```
Create album -> name, description, cover (auto = first asset)
Add assets (multi-select in grid -> "Add to album")
Album view -> grid; reorder; share album (public link)
Remove asset from album (does not delete asset)
```

### Flow F — Auth
```
First run: setup admin account (if none)
Login (email+password or OAuth)
Session cookie (httpOnly) + CSRF
Forgot password -> email reset (SMTP optional)
```

## State Machines

### Asset lifecycle
```
UPLOADED -> PROCESSING -> READY
                    \-> FAILED (retriable)
READY -> (user) ARCHIVED / FAVORITED (flags, not state)
READY -> (admin) DELETED -> TRASHED (30d) -> PURGED
```

### Upload job states (per asset, per job type)
```
QUEUED -> RUNNING -> SUCCESS
                 \-> FAILED (retry x3 with backoff) -> DEAD (surfaced in admin)
```

### Share link states
```
ACTIVE -> (expiry) EXPIRED
ACTIVE -> (user/admin) REVOKED
```

### Search request states (client)
```
idle -> typing (debounce) -> loading -> success | error -> idle
```

## Decision Trees

- **Is this a duplicate?** exact hash match → yes (auto-hide candidate); pHash distance ≤ 6 → candidate for review; else no.
- **Should ML run?** if ML service healthy and feature flag on → run OCR+CLIP; else skip and mark asset "search: text-only".
- **Video handling:** if mime starts video/* → transcode to web-friendly (H.264) + thumbnail at 1s; else image pipeline.
- **Public share access:** token valid & not expired → serve; else 404 (no info leak).

## Navigation Paths

- `/` → timeline/grid (default view: recent)
- `/search` → search results
- `/albums` → album list; `/albums/:id`
- `/tags` → tag cloud/list; `/tags/:id`
- `/favorites`
- `/archive`
- `/trash`
- `/share/:token` → public viewer (no auth)
- `/admin/*` → admin (users, storage, jobs, settings)
- `/settings` → profile, preferences, API keys
