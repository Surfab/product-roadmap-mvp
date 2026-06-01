# Product Roadmap

An offline-first app to manage a product roadmap — a single HTML file. No build step, no backend, no NPM dependencies. Open and use.

<img width="3060" height="3000" alt="Screenshot 21" src="https://github.com/user-attachments/assets/62c4247d-bbb8-4400-bf0f-e06b371196a3" />

---

## ✨ What it does

- **Two views** of the same roadmap: by **Quarter** (Q1–Q4 of a given year) or by **Now / Next / Later**.
- **Drag-and-drop** to move items between columns or reorder them within the same column.
- **Combinable filters**: by tag (AND / OR logic), by status, by free-text search.
- **Local persistence** via `localStorage` — data survives page refreshes, no cloud required.
- **Import / Export** as JSON, **PNG export** (board snapshot), **PDF print** via the browser's print dialog.
- **Undo** after deletion (toast with 6 seconds to restore).
- **Live validation**, char-counter on fields, tag autocomplete based on existing tags.
- **Dark theme**, glassmorphism, fully responsive (mobile included).

---

## 🚀 How to use it

1. Open `index.html` in a modern browser (Chrome, Firefox, Safari, Edge).
2. Click **+ New** (or press `N`) to add the first item.
3. Drag cards between columns to move them.
4. Click a tag or a status badge to filter.

### Sample data

To test the app with realistic data, import `sample-roadmap.json`:

1. Open the **Export ▾** menu in the top right
2. **Import JSON…**
3. Select `sample-roadmap.json`

It contains 14 items across all quarters, statuses, and owners.

---

## ⌨️ Keyboard shortcuts

| Key | Action |
|---|---|
| `N` | New item (when outside input fields) |
| `⌘K` / `Ctrl+K` | Focus the search bar |
| `⌘⏎` / `Ctrl+Enter` | Save (inside the modal) |
| `Esc` | Close modal / Export menu |

Shortcuts are shown in the footer at the bottom of the board.

---

## 🧩 Data model

Each item has this shape:

```js
{
  id: "uid-string",
  title: "Faster onboarding flow",       // required, max 120 chars
  desc: "Cut signup → first value...",   // optional, max 500 chars
  status: "idea" | "planned" | "in_progress" | "done",
  owner: "Growth",                        // optional, max 80 chars
  tags: ["Retention", "UX", "iOS"],      // 0–3 tags, max 40 chars each
  quarter: "2026-Q2",                     // format YYYY-Q[1-4]
  nnl: "now" | "next" | "later",
  order: { "2026-Q2": 0, "next": 1 },    // position per column
  createdAt: "2026-02-15T11:00:00.000Z",
  updatedAt: "2026-04-22T10:15:00.000Z"
}
```

The full state is saved in `localStorage` under the key `roadmap_v1`:

```js
{
  name: "Roadmap",
  year: 2026,
  view: "quarter" | "nnl",
  search: "...",
  tagFilter: [...],
  tagMode: "and" | "or",
  statusFilter: "...",
  items: [...]
}
```

### Migration

The app also looks for the old key `roadmap_mvp_final_v1` and migrates it automatically on first load.

---

## 🏗 Architecture

**Single-file vanilla JS/HTML/CSS.** No framework, no bundler, no build step.

```
index.html              ~1600 lines
├── <style>            CSS variables + glassmorphism dark theme
├── <body>             static markup: topbar, filter row, board, modal, toast
└── <script>           all logic in an IIFE-like block
    ├── Constants      LS keys, VIEWS, NNL_KEYS, TAG_MODES, STATUSES, LIMITS
    ├── Helpers        uid, escape, debounce, safe localStorage wrappers
    ├── State          single mutable object, persisted on every commit
    ├── Filtering      itemMatchesScope + itemMatchesSearch + matchesFilters
    ├── Rendering      render() rebuilds the board on every state change
    └── Wiring         event listeners at the end
```

### Notable functions

| Function | What it does |
|---|---|
| `sanitizeItem(raw, year)` | Validates and normalises an imported item. Filters out invalid statuses, malformed quarters, duplicate tags. |
| `commitAndRender()` | Unified helper: `save()` + `render()`. Used wherever state changes. |
| `handleDrop(itemId, colKey, insertIndex)` | Drag-drop between/within columns. Computes `(prev + next) / 2` as the new order, then normalises. |
| `showToast(msg, type, opts)` | Bottom-right notifications. Supports `actionLabel + onAction` for undo. |
| `computeTagCounts()` | Single-pass over all filtered items. Returns `Map<tag, count>`. |

### External dependencies

Only **one**, loaded via CDN:

- [html2canvas 1.4.1](https://html2canvas.hertzen.com/) — for PNG export. The app works without it (other exports remain available).

---

## 🎨 Design system

- **Palette**: greyscale on `#0b0c10` background, glassy accents with 3 radial gradients.
- **Status colours**: idea (pale purple), planned (blue), in_progress (amber), done (green). Reflected on the card's left border.
- **Typography**: SF Pro / system font, 11–22 px over 6 steps. No Inter, no Roboto.
- **Radius**: 8/10/12/14/16/18 px for increasing container levels.
- **Density**: two pill sizes (small `4-6px / 9-10px`, normal `8-9px / 10-12px`).
- **Glassmorphism**: `backdrop-filter: blur(8-12px)` on topbar, modal, toast.

---

## ♿ Accessibility

- `aria-modal`, `aria-labelledby`, `aria-live` on toast.
- Focus trap in the modal (Tab/Shift+Tab cycle inside).
- Custom focus-visible (blue outline) on all interactive elements.
- `aria-invalid` on the title input in error state.

---

## 🔒 Limits & validation

| Field | Limit |
|---|---|
| Roadmap name | 120 characters |
| Item title | 120 characters (required) |
| Item description | 500 characters |
| Item owner | 80 characters |
| Tags per item | maximum 3 |
| Tag length | 40 characters |
| Duplicate tags | automatic case-insensitive dedup |

`localStorage` write failures (quota exceeded, Safari private mode) show an error toast instead of failing silently.

---

## 🖨 Print / PDF

The **PDF** button opens the browser's print dialog with a dedicated `@media print` stylesheet: white background, grey borders, no topbar/filter/toast.

---

## 📂 Project files

```
.
├── index.html              ← the app
├── sample-roadmap.json     ← sample data for import
├── README.md               ← Italian readme
├── README.en.md            ← this file
└── images/                 ← screenshots
```

---

## 📜 Project history

Refactor from a ~1100-line script to ~1600 lines (with +6 features, validation, accessibility, mobile responsive). See commit history for the steps:

1. Critical refactor (rename `el`, dedup filter logic, constants, `commitAndRender`)
2. Robustness bug fixes (tag dedup, debounce save, schema migration, defensive validation)
3. GUI polish phase 1 (3-cluster topbar, dedicated filter row, board header, column distinction, status-as-left-border)
4. GUI polish phase 2 (typography, density, focus-visible, drag-handle, loading states, dynamic subtitle)
5. Final features (modal validation, tag autocomplete, shortcuts footer, drag-sort within column, mobile responsive)

---

## 📝 License

MIT — do whatever you want, no warranties.
