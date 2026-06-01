# Product Roadmap

[🇮🇹 Italiano](#italiano) · [🇬🇧 English](#english)

---

## Italiano

App offline-first per gestire una roadmap di prodotto, in un unico file HTML. Niente build step, niente backend, niente dipendenze NPM. Apri e usa.

<img width="2238" height="2108" alt="Screenshot" src="https://github.com/user-attachments/assets/be4f707f-7245-4350-b58f-8e4e208ec3b8" />

### ✨ Cosa fa

- **Due viste** della stessa roadmap: per **Quarter** (Q1-Q4 di un anno) o per **Now / Next / Later**.
- **Drag-and-drop** per spostare gli item tra colonne o riordinarli dentro la stessa colonna.
- **Filtri combinabili**: per tag (con logica AND / OR), per status, per testo libero.
- **Persistenza locale** in `localStorage` — i dati restano tra refresh, niente cloud.
- **Bilingue IT / EN** — toggle in topbar, scelta persistita, auto-detect della lingua del browser al primo avvio.
- **Import / Export** in JSON, **export PNG** (snapshot della board), **stampa PDF** via dialog di stampa del browser.
- **Undo** dopo eliminazione (toast con 6 secondi per ripristinare).
- **Validazione live**, char-counter sui campi, autocomplete dei tag basato sui tag esistenti.
- **Tema dark**, glassmorphism, completamente responsive (mobile incluso).

### 🚀 Come usarla

1. Apri `index.html` in un browser moderno (Chrome, Firefox, Safari, Edge).
2. Clicca **+ New** (o premi `N`) per aggiungere il primo item.
3. Trascina le card tra le colonne per spostarle.
4. Clicca su un tag o sul badge di status per filtrare.

#### Dati di esempio

Per testare l'app con dati realistici, importa `sample-roadmap.json`:

1. Apri il menu **Export ▾** in alto a destra
2. **Import JSON…**
3. Seleziona `sample-roadmap.json`

Contiene 14 item su tutti i quarter, status e owner.

### ⌨️ Scorciatoie da tastiera

| Tasto | Azione |
|---|---|
| `N` | Nuovo item (quando fuori dai campi di input) |
| `⌘K` / `Ctrl+K` | Focus sulla barra di ricerca |
| `⌘⏎` / `Ctrl+Enter` | Salva (dentro il modal) |
| `Esc` | Chiudi modal / menu Export |

### 🌐 Lingua

L'interfaccia è disponibile in **italiano** e **inglese**. Cambi lingua con il toggle `IT / EN` in alto a destra nella topbar.

- La scelta è salvata in `localStorage` (chiave `roadmap_lang`) e persiste tra le sessioni.
- Al primo avvio, se non c'è una preferenza salvata, l'app prova a dedurre la lingua dal browser (`navigator.language`), con fallback su italiano.
- Viene tradotta solo la **chrome** (bottoni, label, menu, toast, scorciatoie). Il **contenuto degli item** (titoli, descrizioni, owner, tag) resta come l'hai scritto — non viene tradotto.

### 🧩 Modello dati

```js
{
  id: "uid-string",
  title: "Faster onboarding flow",       // required, max 120 char
  desc: "Cut signup → first value...",   // optional, max 500 char
  status: "idea" | "planned" | "in_progress" | "done",
  owner: "Growth",                        // optional, max 80 char
  tags: ["Retention", "UX", "iOS"],      // 0-3 tag, max 40 char ciascuno
  quarter: "2026-Q2",                     // formato YYYY-Q[1-4]
  nnl: "now" | "next" | "later",
  order: { "2026-Q2": 0, "next": 1 },
  createdAt: "2026-02-15T11:00:00.000Z",
  updatedAt: "2026-04-22T10:15:00.000Z"
}
```

### 🏗 Architettura

**Single-file vanilla JS/HTML/CSS.** Nessun framework, nessun bundler, nessun build step.

```
index.html              ~1600 righe
├── <style>            CSS variables + glassmorphism dark theme
├── <body>             markup statico: topbar, filter row, board, modal, toast
└── <script>           tutto il logic in un IIFE-like blocco
    ├── Constants      LS keys, VIEWS, NNL_KEYS, TAG_MODES, STATUSES, LIMITS
    ├── i18n           STRINGS (it/en), t(), applyStaticI18n(), setLang()
    ├── Helpers        uid, escape, debounce, safe localStorage wrappers
    ├── State          single mutable object, persisted on every commit
    ├── Filtering      itemMatchesScope + itemMatchesSearch + matchesFilters
    ├── Rendering      render() rebuilds the board on every state change
    └── Wiring         event listeners at the end
```

| Funzione | Cosa fa |
|---|---|
| `sanitizeItem(raw, year)` | Valida e normalizza un item importato. Filtra status invalidi, quarter mal formati, tag duplicati. |
| `commitAndRender()` | Helper unificato: `save()` + `render()`. Usato ovunque cambi lo state. |
| `handleDrop(itemId, colKey, insertIndex)` | Drag-drop tra/dentro colonne. Calcola `(prev + next) / 2` come nuovo order, poi normalizza. |
| `showToast(msg, type, opts)` | Notifiche bottom-right. Supporta `actionLabel + onAction` per gli undo. |
| `t(key, vars)` | Traduzione con interpolazione `{placeholder}`. Fallback: lingua corrente → en → chiave stessa. |

Dipendenza esterna: [html2canvas 1.4.1](https://html2canvas.hertzen.com/) via CDN — solo per l'export PNG.

### 🎨 Design system

- **Palette**: scala di grigi su sfondo `#0b0c10`, accenti vetrosi con 3 radial-gradient.
- **Status color**: idea (viola pallido), planned (azzurro), in_progress (ambra), done (verde). Riflessi sul left-border delle card.
- **Tipografia**: SF Pro / system font, 11→22px su 6 step.
- **Glassmorphism**: `backdrop-filter: blur(8-12px)` su topbar, modal, toast.

### 🔒 Limiti

| Campo | Limite |
|---|---|
| Item title | 120 caratteri (obbligatorio) |
| Item description | 500 caratteri |
| Tag per item | massimo 3 |
| Lunghezza tag | 40 caratteri |

### 📜 Storia del progetto

1. Refactor critico (rinomina `el`, dedup filter logic, costanti, `commitAndRender`)
2. Bug fix di robustezza (dedup tag, debounce save, schema migration, validation defensiva)
3. Polish GUI fase 1 (3-cluster topbar, filter row dedicata, board header, status-as-left-border)
4. Polish GUI fase 2 (tipografia, densità, focus-visible, drag-handle, loading states)
5. Feature finali (validation modal, autocomplete tag, shortcuts footer, drag-sort dentro colonna, mobile)
6. Internazionalizzazione (i18n IT/EN con toggle, `STRINGS` + `t()`, persistenza lingua)

### 📝 Licenza

MIT — fai quello che vuoi, niente garanzie.

---

## English

An offline-first app to manage a product roadmap — a single HTML file. No build step, no backend, no NPM dependencies. Open and use.

<img width="2238" height="2108" alt="Screenshot" src="https://github.com/user-attachments/assets/be4f707f-7245-4350-b58f-8e4e208ec3b8" />

### ✨ What it does

- **Two views** of the same roadmap: by **Quarter** (Q1–Q4 of a given year) or by **Now / Next / Later**.
- **Drag-and-drop** to move items between columns or reorder them within the same column.
- **Combinable filters**: by tag (AND / OR logic), by status, by free-text search.
- **Local persistence** via `localStorage` — data survives page refreshes, no cloud required.
- **Bilingual IT / EN** — toggle in the topbar, persisted choice, auto-detects browser language on first launch.
- **Import / Export** as JSON, **PNG export** (board snapshot), **PDF print** via the browser's print dialog.
- **Undo** after deletion (toast with 6 seconds to restore).
- **Live validation**, char-counter on fields, tag autocomplete based on existing tags.
- **Dark theme**, glassmorphism, fully responsive (mobile included).

### 🚀 How to use it

1. Open `index.html` in a modern browser (Chrome, Firefox, Safari, Edge).
2. Click **+ New** (or press `N`) to add the first item.
3. Drag cards between columns to move them.
4. Click a tag or a status badge to filter.

#### Sample data

To test the app with realistic data, import `sample-roadmap.json`:

1. Open the **Export ▾** menu in the top right
2. **Import JSON…**
3. Select `sample-roadmap.json`

It contains 14 items across all quarters, statuses, and owners.

### ⌨️ Keyboard shortcuts

| Key | Action |
|---|---|
| `N` | New item (when outside input fields) |
| `⌘K` / `Ctrl+K` | Focus the search bar |
| `⌘⏎` / `Ctrl+Enter` | Save (inside the modal) |
| `Esc` | Close modal / Export menu |

### 🌐 Language

The interface is available in **Italian** and **English**. Switch with the `IT / EN` toggle in the top right of the topbar.

- The choice is saved in `localStorage` (key `roadmap_lang`) and persists across sessions.
- On first launch, the app auto-detects the browser language (`navigator.language`), falling back to Italian.
- Only the **chrome** is translated (buttons, labels, menus, toasts, shortcuts). **Item content** stays as written.

### 🧩 Data model

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
  order: { "2026-Q2": 0, "next": 1 },
  createdAt: "2026-02-15T11:00:00.000Z",
  updatedAt: "2026-04-22T10:15:00.000Z"
}
```

### 🏗 Architecture

**Single-file vanilla JS/HTML/CSS.** No framework, no bundler, no build step.

```
index.html              ~1600 lines
├── <style>            CSS variables + glassmorphism dark theme
├── <body>             static markup: topbar, filter row, board, modal, toast
└── <script>           all logic in an IIFE-like block
    ├── Constants      LS keys, VIEWS, NNL_KEYS, TAG_MODES, STATUSES, LIMITS
    ├── i18n           STRINGS (it/en), t(), applyStaticI18n(), setLang()
    ├── Helpers        uid, escape, debounce, safe localStorage wrappers
    ├── State          single mutable object, persisted on every commit
    ├── Filtering      itemMatchesScope + itemMatchesSearch + matchesFilters
    ├── Rendering      render() rebuilds the board on every state change
    └── Wiring         event listeners at the end
```

| Function | What it does |
|---|---|
| `sanitizeItem(raw, year)` | Validates and normalises an imported item. Filters invalid statuses, malformed quarters, duplicate tags. |
| `commitAndRender()` | Unified helper: `save()` + `render()`. Used wherever state changes. |
| `handleDrop(itemId, colKey, insertIndex)` | Drag-drop between/within columns. Computes `(prev + next) / 2` as new order, then normalises. |
| `showToast(msg, type, opts)` | Bottom-right notifications. Supports `actionLabel + onAction` for undo. |
| `t(key, vars)` | Translation with `{placeholder}` interpolation. Fallback: current lang → en → key itself. |

External dependency: [html2canvas 1.4.1](https://html2canvas.hertzen.com/) via CDN — PNG export only.

### 🎨 Design system

- **Palette**: greyscale on `#0b0c10` background, glassy accents with 3 radial gradients.
- **Status colours**: idea (pale purple), planned (blue), in_progress (amber), done (green). Reflected on the card's left border.
- **Typography**: SF Pro / system font, 11–22 px over 6 steps.
- **Glassmorphism**: `backdrop-filter: blur(8-12px)` on topbar, modal, toast.

### 🔒 Limits

| Field | Limit |
|---|---|
| Item title | 120 characters (required) |
| Item description | 500 characters |
| Tags per item | maximum 3 |
| Tag length | 40 characters |

### 📜 Project history

1. Critical refactor (rename `el`, dedup filter logic, constants, `commitAndRender`)
2. Robustness bug fixes (tag dedup, debounce save, schema migration, defensive validation)
3. GUI polish phase 1 (3-cluster topbar, dedicated filter row, board header, status-as-left-border)
4. GUI polish phase 2 (typography, density, focus-visible, drag-handle, loading states)
5. Final features (modal validation, tag autocomplete, shortcuts footer, drag-sort within column, mobile)
6. Internationalisation (i18n IT/EN with toggle, `STRINGS` + `t()`, language persistence)

### 📝 License

MIT — do whatever you want, no warranties.
