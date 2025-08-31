# DevLog

A lightweight, project‑centric developer log. Type your steps as Markdown bullets; DevLog auto‑groups them by **date** and prefixes each bullet with a **timestamp**. Switch projects instantly from a left sidebar. Everything is stored as plain Markdown in a user‑chosen folder so Dropbox/iCloud/Drive can sync.

> North Star: **Open app → pick project → type `- fixed header` → DevLog inserts today’s date and `[14:32]` automatically. Done.**

---

## Table of contents

* [Why DevLog](#why-devlog)
* [Key features](#key-features)
* [Quick start](#quick-start)

    * [HTML/CSS prototype (now)](#htmlcss-prototype-now)
    * [Web / PWA](#web--pwa)
    * [Desktop (Tauri)](#desktop-tauri)
* [Project structure](#project-structure)
* [Data model & file layout](#data-model--file-layout)
* [Markdown format](#markdown-format)
* [Editor behaviors](#editor-behaviors)
* [UI & interactions](#ui--interactions)
* [Keyboard shortcuts](#keyboard-shortcuts)
* [Export / Import](#export--import)
* [Settings](#settings)
* [Privacy & security](#privacy--security)
* [Performance & reliability](#performance--reliability)
* [Roadmap](#roadmap)
* [Contributing](#contributing)
* [Design & CSS notes](#design--css-notes)
* [FAQ](#faq)
* [License](#license)

---

## Why DevLog

Traditional note apps aren’t optimized for *temporal* developer logs. DevLog focuses on:

* **Speed**: start typing; dates and timestamps appear automatically.
* **Portability**: Markdown on disk (no lock‑in). Use the same folder on multiple machines.
* **Project‑centric**: sidebar lists all projects, sorted by most recently edited.

---

## Key features

* **Auto date & time**

    * Inserts a date header the first time you type on a new day.
    * Every new bullet auto‑prefixes with a timestamp like `[14:32]` (visible but muted in the UI; kept in Markdown).
* **Left sidebar project switcher**

    * Projects sorted by *Last worked on* (top = most recent).
    * Per‑project icons: **Open**, **Info**, **Tasks**, **Export**, **Delete** (with confirm).
* **Markdown‑first**

    * Live preview / split view.
    * Export a single Markdown file or a ZIP of the project.
    * Import an existing folder; DevLog recognizes it.
* **Lightweight editor tools**

    * Add link, todo checkbox `[ ]` `[x]`, indent/outdent with Tab/Shift+Tab, bold/italic/code, code blocks, headings, quotes.
    * Undo/redo, find‑in‑note, quick jump to **Today**.
* **Storage**

    * Choose a root folder on first run. DevLog writes plain `.md` files there.
    * Works offline and saves instantly.

---

## Quick start

> ⚠️ Status: early WIP. Start with the HTML/CSS prototype, then layer in JavaScript and React.

### HTML/CSS prototype (now)

1. Clone the repo and open `/prototype/` in your browser (no build needed).
2. Edit `index.html` and `styles.css` to iterate on layout: left sidebar + main editor + top bar.
3. Use the provided sample Markdown in `/prototype/mock/` to test scrolling and sticky headers.

### Web / PWA

Once you’re ready to add interactivity:

```bash
# prerequisites: Node 18+ (or 20+), pnpm or npm
pnpm install
pnpm dev           # run React dev server
pnpm build         # production build to /dist
pnpm preview       # preview the build locally
```

* **File access**: When run in the browser, DevLog can use the File System Access API to write directly to a chosen folder (Chrome/Edge/Opera). For Safari/Firefox, DevLog will use a download‑based fallback and prompt to re‑select the folder when needed.
* **PWA**: You can install to the desktop via your browser’s *Install App* option.

### Desktop (Tauri)

For best filesystem support and tiny binaries:

```bash
# prerequisites: Rust toolchain + Tauri CLI (see tauri.app docs)
pnpm tauri dev     # run desktop dev build
pnpm tauri build   # produce native app
```

---

## Project structure

```
root/
├─ prototype/                  # HTML/CSS-first playground
├─ apps/
│  └─ web/                     # React app (PWA)
├─ packages/
│  ├─ editor/                  # Editor logic (Markdown, commands, keymaps)
│  └─ fs/                      # File adapters (Web FS API, Tauri FS)
├─ docs/                       # Screenshots, diagrams
└─ scripts/                    # Dev utilities
```

*(Structure may evolve; keep README in sync.)*

---

## Data model & file layout

**Root folder (chosen by user)**

```
/ProjectsRoot/
  manifest.json
  /acme-site/
    project.json
    /logs/
      2025-08.md          # monthly log file
      2025-09.md
    tasks.md              # project tasks (Markdown)
    info.md               # project notes/credentials (no secrets)
```

**Manifest (root)**

```json
{
  "version": 1,
  "projects": [
    {
      "id": "acme-site",
      "name": "Acme Site",
      "path": "acme-site",
      "lastEdited": "2025-08-11T14:32:05Z"
    }
  ]
}
```

**Project config**

```json
{
  "id": "acme-site",
  "name": "Acme Site",
  "createdAt": "2025-06-01T10:00:00Z",
  "timeFormat": "HH:mm",
  "dateFormat": "YYYY-MM-DD (ddd)",
  "logPartition": "monthly"   // also supports "daily" or "single"
}
```

**Monthly log file (Markdown)**

```md
---
projectId: acme-site
month: 2025-08
timezone: Europe/Sofia
generatedBy: DevLog v1
---

# Acme Site — 2025-08

## 2025-08-11 (Mon)
- [09:12] Fix: header z-index on mobile
- [09:40] Add aria-labels to main nav
- [10:05] Deploy to staging https://staging.example.com
- [11:20] Client call summary:
  - Notes about hero image
  - Next steps agreed

## 2025-08-08 (Fri)
- [13:01] Update footer links
```

**Why monthly files?** Keeps files small and fast to load, easy to export/merge. Prefer one file? Set `logPartition: "single"`.

---

## Markdown format

* **Date headers**: `## 2025-08-11 (Mon)` (format configurable per project).
* **Bullets with time**: `- [14:32] Fix header stacking context`
* **Tasks**: `- [ ] Replace homepage video` → `- [x] Replace homepage video`
* **Timestamp rendering**: in preview, timestamps are wrapped in `<span class="timestamp">[14:32]</span>` and styled muted via CSS; in Markdown, they remain `[14:32]`.

---

## Editor behaviors

* **Auto date header**: When you first type each day (or after midnight), DevLog inserts `## YYYY-MM-DD (EEE)` once in the current monthly file.
* **Auto timestamp**: Press **Enter** at end of a bullet; the new line starts with `- [HH:MM] ` automatically. If you backspace, the timestamp is just text and disappears normally.
* **No ghost entries**: DevLog doesn’t create a date section or file until you add non‑whitespace after the timestamp. Autosave fires after you actually type content.
* **Conflict handling**: If a write fails due to external edits (e.g., during sync), DevLog loads both versions and offers a merge view; newest bullets append under the same date by default.
* **Autosave**: Debounced 500–1000ms; writes atomically (temp file then rename).

---

## UI & interactions

**Layout**

* Left **sidebar** (collapsible) with search/filter; each item shows *name* and *last activity time*.
* **Top bar** with project breadcrumb, **Today** jump, and editor mode toggle (**Write / Preview / Split**).
* **Main panel** is the Markdown editor with sticky date headers and a selection toolbar (bold/italic/link/code).
* Status row shows autosave indicator, file path, word count.

**Sidebar buttons per project**

1. **Open** (click the project) → load in main panel.
2. **Info** → open `info.md` modal/editor.
3. **Tasks** → open `tasks.md` (supports checkboxes).
4. **Delete** → confirm dialog: *“Delete project ‘Acme Site’? This moves the folder to your system Trash.”*
5. **Export** → choose **Markdown** (single concatenated) or **Zip** (preserve structure).

**Sorting**

* Projects sort **descending by `lastEdited`** (persisted in `manifest.json`). `lastEdited` updates on save.

---

## Keyboard shortcuts

* **New bullet with timestamp**: `Enter` (contextual)
* **Indent / Outdent**: `Tab` / `Shift+Tab`
* **Toggle task**: `Cmd/Ctrl+Enter` *(or)* `Alt+X`
* **Link**: `Cmd/Ctrl+K`
* **Bold / Italic / Code**: `Cmd/Ctrl+B` / `Cmd/Ctrl+I` / <code>Cmd/Ctrl+\`</code>
* **Find**: `Cmd/Ctrl+F`
* **Jump to Today**: `Cmd/Ctrl+J`
* **Switch project**: `Cmd/Ctrl+P`

---

## Export / Import

**Export**

* **Single Markdown**: Concatenate all monthly logs (newest → oldest), prepend `# Project Name`, and append `## Tasks` and `## Info` sections.
* **Zip**: Include `/logs`, `tasks.md`, `info.md`, and `project.json` unchanged.

**Import**

* Select a folder. If it has `project.json`, adopt it; otherwise scan `/logs/*.md`, infer `id` from folder name, and create a minimal `project.json`.

---

## Settings

* **Projects root folder** (choose/change).
* **Date format** (default `YYYY-MM-DD (ddd)`).
* **Time format** (default 24h `HH:mm`; 12h optional).
* **Log partition**: `monthly` / `daily` / `single`.
* **Timestamp visibility**: *Muted* / *Normal* / *Hidden in preview*.
* **Editor options**: line numbers (toggle), soft wrap (toggle).
* **Keyboard shortcuts reference**.

---

## Privacy & security

* Files are plain text for portability. **Do not store secrets** in `info.md`. Prefer a password manager.
* No cloud backend; syncing is delegated to Dropbox/iCloud/Drive.
* Optional (later): local encryption for `info.md` only (AES‑GCM with user passphrase); store small metadata to indicate encryption.

---

## Performance & reliability

* Lazy‑load the current month’s log and virtualize long documents.
* Keep a rolling local history of saves (JSON patches or copy‑on‑save) for quick restore.
* Handle midnight rollover and timezone changes gracefully (timestamps in local time; include UTC in frontmatter if present).
* Detect and highlight external edits; refresh file hash accordingly.

---

## Roadmap

**MVP (2–3 weeks)**

* [ ] Project picker + sidebar sort by last edited
* [ ] Monthly logs with date headers + auto timestamps
* [ ] Basic editor: bullets, links, tasks, indent/outdent
* [ ] Autosave + atomic writes
* [ ] Export single Markdown
* [ ] Delete → system Trash

**Quality pass**

* [ ] Import existing folders
* [ ] Settings pane (formats, partition, visibility)
* [ ] Keyboard shortcuts help
* [ ] Search across projects
* [ ] Merge on conflict

**Polish**

* [ ] Split view (Write/Preview)
* [ ] Encryption option for `info.md`
* [ ] Quick jump to date, local version history
* [ ] Themes (light/dark), accessibility audit

---

## Contributing

1. **Fork** the repo and create a feature branch.
2. Run `pnpm dev` (web) or `pnpm tauri dev` (desktop).
3. Add/update tests for editor behaviors where possible.
4. Open a PR with a clear description and screenshots/GIFs.

**Commit style**: Conventional Commits are encouraged (e.g., `feat(editor): auto insert timestamp on Enter`).

**Code style**: Prettier + ESLint; TypeScript preferred for app code.

---

## Design & CSS notes

* **Layout**: 3‑pane app — left sidebar (collapsible), top bar, main editor. Keep a compact density in the sidebar.
* **Typography**: Use a mono‑compatible code font for editor content, system UI font elsewhere.
* **Timestamp**: visually muted (≈50–60% of text color). Example preview HTML:

  ```html
  <li><span class="timestamp">[14:32]</span> Fix header stacking context</li>
  ```
* **Color tokens** (CSS custom properties):

  ```css
  :root{
    --bg: #0b0b0c; --fg: #e8e8ea; --muted: #9aa0a6;
    --accent: #7c9cff; --danger: #ff6b6b; --ok: #37d67a;
    --surface: #151518; --border: #2a2a2e;
  }
  .timestamp{ opacity:.56 }
  .sidebar{ width: 280px }
  ```
* **Accessibility**: ensure 4.5:1 contrast for text, focus rings on interactive elements, ARIA labels for toolbar buttons, and `prefers-reduced-motion` support for transitions.

---

## FAQ

**Why not keep everything in one Markdown file per project?**
You can. Set `logPartition: "single"`. Monthly files stay snappier and make merges simpler.

**Does DevLog store anything in the cloud?**
No. All files live where *you* choose. Use a synced folder if you want multi‑device access.

**Can I migrate to/from Obsidian/Logseq?**
Yes. The files are plain Markdown. Import your folder and DevLog will adopt it.

**What about secrets (passwords, API keys)?**
Keep secrets in your password manager. `info.md` is for non‑sensitive notes only.

---

## License

MIT © 2025 DevLog contributors

---

## Changelog

See [`CHANGELOG.md`](./CHANGELOG.md) for notable changes (WIP).

---

## Screenshots (placeholders)

* Sidebar & editor: `docs/screenshots/main.png`
* Export dialog: `docs/screenshots/export.png`

---

## Implementation notes (for maintainers)

* Recommended stack: **Tauri + React** with a Markdown‑aware editor (TipTap/Lexical) and `markdown-it` + plugins (task lists, tables, code fences).
* Search: local full‑text index (MiniSearch/Lunr).
* FS abstraction layer to support both Web FS API and Tauri FS.
* Midnight rollover handler inserts today’s header on first keystroke and scrolls to it.
