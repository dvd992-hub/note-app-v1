# Notes App

A lightweight, browser-based note-taking app. No server, no account, no build step — open `index.html` and start writing.

---

## Project structure

```
notes-app/
├── index.html              Main HTML document
├── css/
│   └── style.css           All styles, CSS variables, responsive rules
├── js/
│   ├── i18n.js             Internationalisation module (auto-detect + toggle)
│   └── script.js           Application logic, state management, persistence
├── i18n/
│   ├── en.json             English UI strings
│   └── it.json             Italian UI strings
├── assets/
│   └── favicon/
│       ├── favicon.svg     SVG favicon (modern browsers)
│       └── favicon.ico     ICO favicon — 16×16, 32×32, 48×48 (fallback)
└── README.md               This file
```

---

## Getting started

Open `index.html` directly in any modern browser — no dependencies to install.

```bash
# Optional: local development server (avoids file:// CORS for XHR)
npx serve .
# or
python3 -m http.server
```

> **Note:** The i18n module uses `XMLHttpRequest` to load JSON files.
> Opening `index.html` directly via `file://` may work in most browsers, but a
> local server (e.g. VS Code Live Server) is recommended to avoid any CORS restrictions.

---

## Features

### Notes

| Action | How |
|--------|-----|
| Create a note | `＋` button in the sidebar or `Ctrl / Cmd + N` |
| Edit a note | Click it in the sidebar, then type |
| Auto-save | 600 ms after the last keystroke (debounced) |
| Delete a note | 🗑 Delete button in the editor toolbar |
| Search | Type in the search bar — filters title and content live |

### Rich-text formatting

The content area is a `contenteditable` div with a formatting toolbar:

| Button | Command | `execCommand` |
|--------|---------|---------------|
| **B** | Bold | `bold` |
| *I* | Italic | `italic` |
| <u>U</u> | Underline | `underline` |
| • List | Bullet list | `insertUnorderedList` |
| 1. Num | Numbered list | `insertOrderedList` |

### Colour tags

Each note can have one colour tag, visible as a left border in the sidebar and as a dot in the note metadata row.

| Tag | Colour | CSS variable |
|-----|--------|-------------|
| Indigo | `#6366F1` | `--tag-a` |
| Green  | `#10B981` | `--tag-b` |
| Amber  | `#F59E0B` | `--tag-c` |
| Pink   | `#EC4899` | `--tag-d` |

### Persistence

Notes are stored in `localStorage` under the key `notes_v1`.  
Each note object has this shape:

```json
{
  "id":      "abc123def456",
  "title":   "My note title",
  "content": "<p>Rich <b>HTML</b> content</p>",
  "tag":     "a",
  "created": 1718000000000,
  "updated": 1718003600000
}
```

---

## Internationalisation (i18n)

The app ships with English (`en`) and Italian (`it`) translations.

### How the language is resolved (hybrid strategy)

1. **localStorage override** — if the user has previously chosen a language via the toggle, that choice is loaded first (`notes_lang` key).
2. **Browser / OS preference** — `navigator.language` is read and matched against supported locales (`en`, `it`). Only the two-letter prefix (`en-US` → `en`) is used.
3. **Fallback** — if neither source yields a supported locale, English is used.

### Manual toggle

The **EN / IT** pill in the sidebar header lets users override the detected language at any time. The selection is saved to `localStorage` and remembered on future visits.

### Adding a new locale

1. Create `i18n/xx.json` (copy `en.json` as a template and translate all values).
2. Add `'xx'` to the `SUPPORTED` array in `js/i18n.js`.
3. Add a button to the language toggle in `index.html`:
   ```html
   <button class="lang-btn" data-lang="xx">XX</button>
   ```

### String keys reference

| Key | Used for |
|-----|----------|
| `appTitle` | Sidebar and topbar title |
| `newNote` | New-note button title attribute |
| `searchPlaceholder` | Search input placeholder |
| `emptyList` | Message when search finds nothing |
| `titlePlaceholder` | Note title input placeholder |
| `bodyPlaceholder` | Content area CSS placeholder |
| `colorLabel` | Tag picker label |
| `btnDelete` | Delete button label |
| `btnBold / btnItalic / btnUnder` | Toolbar button labels |
| `btnBullet / btnOrdered` | List button labels |
| `confirmDelete` | `confirm()` dialog text |
| `placeholderMsg` | Empty-state message |
| `footerModified` | "Modified" prefix in footer |
| `footerWord / footerWords` | Singular/plural word count |
| `tagNone / tagA / tagB / tagC / tagD` | Colour swatch `title` attributes |
| `untitled / emptyNote` | Fallback strings in the list |
| `openMenu` | Hamburger button title attribute |

---

## Design

### Palette — parchment tones

All colours are CSS custom properties in `:root`; changing one variable updates the entire UI.

| Variable | Value | Purpose |
|----------|-------|---------|
| `--bg` | `#FAF6E9` | Main background |
| `--surface` | `#FDF9EE` | Elevated surfaces (cards, inputs) |
| `--sidebar` | `#F2ECD6` | Sidebar background |
| `--border` | `#E5DDB8` | Borders and dividers |
| `--text` | `#1C1B1F` | Primary text |
| `--muted` | `#79767E` | Secondary text / placeholders |
| `--accent` | `#6366F1` | Buttons, focus indicators |
| `--accent-lt` | `#EEF2FF` | Active-state backgrounds |
| `--red` | `#EF4444` | Destructive actions |

### Typography

| Font | Source | Used for |
|------|--------|---------|
| Instrument Serif | Google Fonts | Note titles, app title |
| Inter | Google Fonts | All UI chrome, note body text |

### Visual signature

The 3 px coloured left border on sidebar note cards is the app's only ornament — an immediate visual cue for the tag with zero extra noise.

---

## Favicon

Two formats are provided for maximum compatibility:

| File | Format | Used by |
|------|--------|---------|
| `favicon.svg` | SVG | Chrome 80+, Firefox 83+, Safari 15+, Edge 80+ |
| `favicon.ico` | ICO (16×16, 32×32, 48×48) | Older browsers, Windows taskbar, bookmark icons |

The `<head>` loads SVG first so modern browsers use it; the ICO `<link>` acts as fallback.

---

## Responsive layout

| Viewport | Behaviour |
|----------|-----------|
| `> 680 px` (desktop) | Sidebar fixed on the left; editor fills the right |
| `≤ 680 px` (mobile) | Sidebar is an off-canvas drawer; topbar shows hamburger + new-note btn |

On mobile, the sidebar opens via the hamburger icon and closes by tapping the overlay or selecting a note.

---

## Accessibility

- `prefers-reduced-motion`: all CSS transitions disabled when the OS "Reduce Motion" setting is on.
- ARIA attributes on interactive elements: `role="list"`, `role="toolbar"`, `role="group"`, `role="textbox"`, `aria-multiline`, `aria-label`, `aria-live`.
- `document.documentElement.lang` updated when the locale changes (screen-reader support).
- Keyboard shortcut: `Ctrl + N` / `Cmd + N` creates a new note.

---

## Customisation

### Change the colour palette

Edit the variables in `:root` inside `css/style.css`:

```css
:root {
  --bg:     #FAF6E9;   /* try #F5F0E8 for a warmer tone */
  --accent: #6366F1;   /* try #E11D48 for a rose theme  */
}
```

### Change the sidebar width

```css
:root {
  --sidebar-w: 320px;  /* default: 280px */
}
```

### Change the auto-save delay

In `js/script.js`, inside `scheduleSave()`:

```js
saveTimer = setTimeout(flushSave, 600);  // milliseconds
```

### Add a new colour tag

1. Add a CSS variable in `:root`: `--tag-e: #8B5CF6;`
2. Add dot and border rules in `css/style.css`:
   ```css
   .tag-dot.tag-e   { background: var(--tag-e); }
   .note-item.tag-e { border-left-color: var(--tag-e); }
   .tag-option[data-tag="e"] { background: var(--tag-e); }
   ```
3. Add a swatch to the picker in `index.html`:
   ```html
   <div class="tag-option" data-tag="e" data-i18n-title="tagE"></div>
   ```
4. Add the `tagE` key to both `i18n/en.json` and `i18n/it.json`.

---

## Browser compatibility

| Browser | Version | Notes |
|---------|---------|-------|
| Chrome / Edge | 80+ | Full support |
| Firefox | 83+ | Full support |
| Safari | 15+ | Full support (SVG favicon requires 15+) |
| Safari | 12–14 | Works; ICO favicon used instead of SVG |
| IE 11 | — | Not supported (`const`, `arrow functions`, `fetch`) |

---

## Known limitations

- **`document.execCommand`** (rich-text formatting) is deprecated in the W3C spec but still universally supported. For a production app, replace it with a library such as [Tiptap](https://tiptap.dev/) or [Quill](https://quilljs.com/).
- **`localStorage`** has a ~5 MB per-origin limit. For large volumes of notes, consider IndexedDB.
- Data is local to the browser and device — no cross-device sync.

---

## External dependencies

| Resource | Purpose | Loaded via |
|----------|---------|-----------|
| Google Fonts — Instrument Serif | Display / title font | CDN `<link>` |
| Google Fonts — Inter | UI / body font | CDN `<link>` |

No JavaScript libraries. No CSS framework. No build tool.

---

## Changelog

### v1.0.0
- Initial release
- Multi-file structure: `css/`, `js/`, `i18n/`, `assets/favicon/`
- Hybrid i18n system: `navigator.language` auto-detect + EN/IT manual toggle, persisted in `localStorage`
- SVG + ICO favicon (16×16, 32×32, 48×48)
- Parchment colour palette
- Rich-text editor with toolbar (bold, italic, underline, lists)
- Colour tags with left-border visual
- Live search, auto-save (debounced 600 ms), word count footer
- Responsive layout with mobile sidebar drawer
- `prefers-reduced-motion` support
- Keyboard shortcut: `Ctrl / Cmd + N` for new note
