# Client Photo Select

A desktop application that lets photographers deliver photo galleries to clients for selection, rating, and commenting — all without any cloud service, subscription, or internet connection.

The photographer builds a self-contained client package (a folder with a portable `.exe` and generated previews), ZIPs it, and sends it to the client. The client opens it on their Windows machine, reviews the photos, and saves their picks. The photographer opens the same folder, runs an export, and gets a CSV + summary ready to send back.

---

## How It Works

```
Photographer                          Client
──────────                            ──────
1. Open app                           4. Receives ZIP
2. Run setup wizard                   5. Unzips, runs .exe
3. ZIP & send folder  ──────────────► 6. Rates / picks photos
                                      7. Sends folder back
8. Open folder        ◄──────────────
9. Export CSV + summary
10. Compose email with attachments
```

---

## Features

### For the Photographer
- **Setup wizard** — point at a folder of originals, choose an output location, configure watermarks and branding; previews and thumbnails are generated automatically
- **Watermark support** — overlay a logo image or custom text on all generated previews; opacity, scale, and position are all configurable
- **Custom branding** — set studio name, photographer name, Instagram handle, and a custom EXE icon; the client sees your brand, not generic software
- **Client email field** — stored per-project; pre-filled as the To address when composing export emails
- **Export** — one click writes three files into an `exports/` folder:
  - `selection_report.csv` — full table of every photo with rating, flag, selection, and comment
  - `selection_summary.txt` — human-readable sections grouped by rating, flag, and selection status
  - `lightroom_selected_names.txt` — comma-separated filenames ready to paste into Lightroom's filter bar
- **Burst deduplication** — optional (on by default): strips trailing `-N` suffixes from filenames so `_A747902-3` and `_A747902-4` collapse to a single `_A747902` in the name list
- **Compose email** — opens the default mail client via Windows MAPI with all three export files attached automatically; falls back to `mailto:` on non-MAPI systems

### For the Client
- **Photo grid** — responsive thumbnail grid with live filtering by rating, flag, and selection status
- **Detail viewer** — full-size preview with keyboard navigation, thumbnail carousel strip, star rating, pick/reject/clear controls
- **Collapsible note field** — per-photo comments with autosave; accessible via keyboard shortcut `C`
- **Keyboard-first** — arrow keys to navigate, `1`–`5` to rate, `P`/`X`/`U` to pick/reject/clear, `Enter` in the note field to save and advance
- **Undo/redo** — full undo stack for ratings, flags, selections, and comments
- **Bulk operations** — checkbox multi-select for applying flags, ratings, or comments to many photos at once
- **No internet required** — fully offline; all data stays in a local SQLite database beside the photos

---

## Tech Stack

| Layer | Technology |
|---|---|
| Desktop shell | [Tauri v2](https://tauri.app/) |
| Backend | Rust |
| Database | SQLite via `rusqlite` (bundled) |
| Image processing | `image` crate — JPEG/PNG/TIFF decode, resize, watermark compositing |
| Email (Windows) | Simple MAPI (`MAPISendMail`) via `windows-sys` — native attach-and-compose |
| Frontend | React 18 + TypeScript |
| Build tool | Vite |
| Font | [Geist](https://vercel.com/font) |

---

## Architecture

```
src/                        React + TypeScript frontend
  App.tsx                   Single-page app (~3500 lines): all screens,
                            state, keyboard handling, viewer, grid
  styles.css / tokens.css   Design tokens + component styles

src-tauri/src/
  lib.rs                    All Tauri commands: project creation, image
                            import, preview generation, export, email
  db.rs                     SQLite schema, migrations, typed DTOs
  previews.rs               Parallel preview + thumbnail generation (rayon)
  text_overlay.rs           Text watermark rendering (fontdue)
  windows_mapi.rs           Native Windows MAPI email with attachments
  icon_embed.rs             Embed custom PNG as Windows EXE icon at runtime
```

**Data flow:**
- On setup, originals are scanned, previews generated in parallel (rayon), and metadata written to SQLite
- All client interactions (ratings, flags, comments) write immediately to SQLite with undo entries
- On export, the Rust backend reads all rows, formats three output files, and optionally deduplicates burst filenames
- The portable EXE is named after the studio (e.g. `Jane Doe Photography.exe`) and copied beside the database on package creation

---

## Key Engineering Decisions

**Why Tauri instead of Electron?**
Tauri produces a much smaller binary (~5 MB vs ~150 MB) and uses the OS WebView rather than bundling Chromium — important for a tool that photographers ZIP and send to clients.

**Why a local SQLite database?**
Zero infrastructure. The database lives beside the photos and travels with the ZIP. No accounts, no sync, no server.

**Why Simple MAPI for email?**
`mailto:` URLs can't attach files reliably across mail clients on Windows. MAPI opens a compose window in the user's actual default mail app (Outlook, Thunderbird, etc.) with files pre-attached — one click to send.

**Why a single App.tsx?**
This app has one screen with a lot of interconnected state (filtered list, detail view, bulk selection, undo stack, keyboard handler, autosave timers). Splitting into many components would introduce prop-drilling or a global store with more boilerplate than benefit at this scale.

---

## Screenshots

> Coming soon

---

## Status

Active development. Private source — this repository is a project showcase.
