<div align="center">

# 📷 Client Photo Select

**Offline photographer client gallery — deliver photos for selection, rating, and commenting without any cloud service.**

[![Stack: Tauri v2 / Rust / React](https://img.shields.io/badge/Stack-Tauri%20v2%20%2F%20Rust%20%2F%20React-FFC131?style=flat-square)]()
[![Platform: Windows](https://img.shields.io/badge/Platform-Windows-0078D6?style=flat-square&logo=windows)]()
[![Showcase](https://img.shields.io/badge/Source-Private%20Showcase-lightgrey?style=flat-square)]()

> 🔒 Private source — this repository is a project showcase.

</div>

---

## ⚙️ How It Works

The photographer builds a self-contained client package — a folder with a portable `.exe` and generated previews — ZIPs it, and sends it to the client. The client opens it on their Windows machine, reviews the photos, and saves their picks. The photographer opens the same folder, exports, and gets a CSV and summary ready to send back.

```
📸 Photographer                       👤 Client
────────────────                      ─────────
1. Open app                           4. Receives ZIP
2. Run setup wizard                   5. Unzips, runs .exe
3. ZIP & send folder  ──────────────► 6. Rates / picks / comments
                                      7. Sends folder back
8. Open folder        ◄──────────────
9. Export CSV + summary
10. Compose email with attachments
```

---

## ✨ Features

### 📸 For the Photographer

- 🧙 **Setup wizard** — point at a folder of originals; previews and thumbnails are generated automatically
- 💧 **Watermark support** — logo image or custom text, with configurable opacity, scale, and position
- 🎨 **Custom branding** — studio name, photographer name, Instagram handle, and custom EXE icon; the client sees your brand
- 📤 **Export** — one click writes three files to an `exports/` folder:
  - 📊 `selection_report.csv` — full table with rating, flag, selection, and comment for every photo
  - 📝 `selection_summary.txt` — human-readable sections grouped by rating, flag, and selection status
  - 🖼️ `lightroom_selected_names.txt` — comma-separated filenames ready to paste into Lightroom's filter bar
- 📸 **Burst deduplication** — collapses burst-suffix filenames (`_A747902-3`, `_A747902-4` → `_A747902`) in the name list
- 📧 **Compose email** — opens the default mail client via Windows MAPI with all three files pre-attached

### 👤 For the Client

- 🖼️ **Photo grid** — responsive thumbnail grid with live filtering by rating, flag, and selection status
- 🔍 **Detail viewer** — full-size preview with keyboard navigation, thumbnail strip, star rating, and pick/reject controls
- 📝 **Per-photo notes** — collapsible comment field with autosave; keyboard shortcut `C`
- ⌨️ **Keyboard-first** — arrows to navigate, `1`–`5` to rate, `P` / `X` / `U` to pick/reject/clear
- ↩️ **Undo/redo** — full undo stack for ratings, flags, selections, and comments
- ☑️ **Bulk operations** — checkbox multi-select for applying flags, ratings, or comments to many photos at once
- 📵 **Fully offline** — no internet required; all data lives in a local SQLite database beside the photos

---

## 🛠️ Tech Stack

| Layer | Technology |
|:---|:---|
| Desktop shell | Tauri v2 |
| Backend | Rust |
| Database | SQLite via `rusqlite` (bundled) |
| Image processing | `image` crate — JPEG/PNG/TIFF decode, resize, watermark compositing |
| Email (Windows) | Simple MAPI (`MAPISendMail`) via `windows-sys` |
| Frontend | React 18 + TypeScript |
| Build tool | Vite |
| Font | Geist |

---

## 🏗️ Architecture

```
src/                      React + TypeScript frontend
  App.tsx                 Single-page app — all screens, state, keyboard
                          handling, viewer, grid, undo stack
  styles.css / tokens.css Design tokens + component styles

src-tauri/src/
  lib.rs                  Tauri commands — project creation, image import,
                          preview generation, export, email
  db.rs                   SQLite schema, migrations, typed DTOs
  previews.rs             Parallel preview + thumbnail generation (rayon)
  text_overlay.rs         Text watermark rendering (fontdue)
  windows_mapi.rs         Native Windows MAPI email with attachments
  icon_embed.rs           Embed custom PNG as Windows EXE icon at runtime
```

---

## 💡 Key Decisions

**🪶 Why Tauri?** Produces a ~5 MB binary using the OS WebView instead of bundling Chromium (~150 MB) — important for a tool photographers ZIP and send to clients.

**🗄️ Why local SQLite?** Zero infrastructure. The database travels with the ZIP. No accounts, no sync, no server.

**📧 Why Simple MAPI?** `mailto:` URLs can't attach files reliably across mail clients on Windows. MAPI opens a compose window in the user's actual mail app with files pre-attached.

---

## 📊 Status

Active development. 🔒 Private source — this repository is a project showcase.
