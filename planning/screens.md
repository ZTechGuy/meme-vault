# planning/screens.md — Screen Inventory

## Global Chrome
- **TopBar:** Logo (Meme-Vault), global search input, upload button, notifications, user menu.
- **SideNav (desktop) / BottomNav (mobile):** Home (Recent), Search, Albums, Tags, Favorites, Archive, Trash, Admin (if admin).
- **Command palette (Cmd/Ctrl+K):** quick actions (upload, new album, search, settings).

## Screens

### 1. Home / Recent Grid (`/`)
- Layout: dense responsive grid (6-10 cols desktop, 3-4 mobile). Virtualized.
- Components: FilterBar (type, date, tags), AssetTile (thumb, hover preview, quick actions), UploadFAB, EmptyState.
- Quick actions on tile: favorite, add-to-album, share, delete.
- Infinite scroll pagination.

### 2. Search Results (`/search?q=...`)
- Layout: search bar sticky top + filter sidebar (desktop) / filter sheet (mobile) + results grid.
- Components: SearchBar, FilterPanel, ResultsGrid, SuggestionChips (top tags), EmptyState.
- Shows query interpretation ("OCR text + semantic").

### 3. Asset Viewer (modal / `/assets/:id`)
- Layout: centered media (image/video) with metadata panel (collapsible).
- Components: MediaViewer (zoom/pan, video controls), MetaPanel (OCR text, tags, albums, dates, hash), ActionBar (favorite, tag, album, share, download, delete, info).
- Keyboard: arrows navigate, Esc close, S share, F favorite.

### 4. Upload Dialog
- Drag-and-drop zone, file picker, paste-from-clipboard, URL import (optional).
- Progress list per file with status; retry failed.
- Post-upload: optional tag prompt.

### 5. Albums List (`/albums`)
- Grid of album cards (cover, name, count, last updated).
- Actions: new album, sort, search albums.

### 6. Album View (`/albums/:id`)
- Header: name, description, cover, share button, edit, delete.
- Grid of assets; reorder (drag) optional; multi-select toolbar.

### 7. Tags (`/tags`)
- Tag cloud / list with counts; click → filtered grid.
- Manage tags (rename, merge, delete) for admin/curator.

### 8. Favorites (`/favorites`)
- Grid of favorited assets; same tile actions.

### 9. Archive (`/archive`)
- Grid of archived assets; restore action.

### 10. Trash (`/trash`)
- Grid with days-until-purge; restore / purge now.

### 11. Duplicates Review (`/admin/duplicates` or curator view)
- Groups of similar assets; side-by-side; keep/merge/delete actions.

### 12. Share Management (`/shares` or in settings)
- List of public links: target, created, expiry, views, revoke.

### 13. Public Share Viewer (`/share/:token`)
- Minimal chrome: media + download button; no nav; watermark optional.
- Expired/revoked → friendly 404.

### 14. Admin Dashboard (`/admin`)
- Tabs: Users, Storage, Jobs, Settings, System health.
- Users: list, invite, roles, quota.
- Storage: usage by user, volumes.
- Jobs: queue status, failed jobs, retry.
- Settings: ML toggles (OCR, CLIP, face recog), dedup threshold, upload limits, branding.

### 15. Settings / Profile (`/settings`)
- Profile (name, avatar, password), preferences (theme, grid density, default view), API keys, connected devices.

### 16. Setup (first run)
- Create admin account, storage path, optional SMTP/ML config.

## Component Library (reuse Immich's, rebrand)

- Button, Input, Select, Modal, Toast, Dropdown, ContextMenu, Badge, Skeleton, EmptyState, VirtualGrid, MediaPreview, UploadDropzone, SearchBar, FilterPanel, TagInput, ShareModal, ConfirmDialog.
- All components: dark mode default (memes viewed in dark), light mode available.

## Layout Conventions

- Desktop: TopBar + SideNav (collapsible) + main content.
- Mobile: TopBar (compact) + BottomNav + main content; filters in bottom sheet.
- Breakpoints: sm 640, md 768, lg 1024, xl 1280.
- Grid density setting: compact (default for memes) / comfortable.
