# Product Roadmap

App offline-first per gestire una roadmap di prodotto, in un unico file HTML. Niente build step, niente backend, niente dipendenze NPM. Apri e usa.

![Screenshot](screenshots/gui-v4-final.png)

---

## ✨ Cosa fa

- **Due viste** della stessa roadmap: per **Quarter** (Q1-Q4 di un anno) o per **Now / Next / Later**.
- **Drag-and-drop** per spostare gli item tra colonne o riordinarli dentro la stessa colonna.
- **Filtri combinabili**: per tag (con logica AND / OR), per status, per testo libero.
- **Persistenza locale** in `localStorage` — i dati restano tra refresh, niente cloud.
- **Bilingue IT / EN** — toggle in topbar, scelta persistita, auto-detect della lingua del browser al primo avvio.
- **Import / Export** in JSON, **export PNG** (snapshot della board), **stampa PDF** via dialog di stampa del browser.
- **Undo** dopo eliminazione (toast con 6 secondi per ripristinare).
- **Validazione live**, char-counter sui campi, autocomplete dei tag basato sui tag esistenti.
- **Tema dark**, glassmorphism, completamente responsive (mobile incluso).

---

## 🚀 Come usarla

1. Apri `index.html` in un browser moderno (Chrome, Firefox, Safari, Edge).
2. Clicca **+ New** (o premi `N`) per aggiungere il primo item.
3. Trascina le card tra le colonne per spostarle.
4. Clicca su un tag o sul badge di status per filtrare.

### Dati di esempio

Per testare l'app con dati realistici, importa `sample-roadmap.json`:

1. Apri il menu **Export ▾** in alto a destra
2. **Import JSON…**
3. Seleziona `sample-roadmap.json`

Contiene 14 item su tutti i quarter, status e owner.

---

## ⌨️ Scorciatoie da tastiera
| Tasto | Azione |
|---|---|
| `N` | Nuovo item (quando fuori dai campi di input) |
| `⌘K` / `Ctrl+K` | Focus sulla barra di ricerca |
| `⌘⏎` / `Ctrl+Enter` | Salva (dentro il modal) |
| `Esc` | Chiudi modal / menu Export |

Le scorciatoie sono visibili nel footer in basso alla board.

---

## 🌐 Lingua

L'interfaccia è disponibile in **italiano** e **inglese**. Cambi lingua con il toggle `IT / EN` in alto a destra nella topbar.

- La scelta è salvata in `localStorage` (chiave `roadmap_lang`) e persiste tra le sessioni.
- Al primo avvio, se non c'è una preferenza salvata, l'app prova a dedurre la lingua dal browser (`navigator.language`), con fallback su italiano.
- Viene tradotta solo la **chrome** (bottoni, label, menu, toast, scorciatoie). Il **contenuto degli item** (titoli, descrizioni, owner, tag) resta come l'hai scritto — non viene tradotto.

### Aggiungere una lingua

Le stringhe vivono in un unico oggetto `STRINGS` dentro `index.html`:

```js
const STRINGS = {
  it: { "action.new": "Nuovo", /* ... */ },
  en: { "action.new": "New",   /* ... */ }
};
```

Per aggiungere p.es. lo spagnolo: duplica un blocco, traduci i valori, aggiungi `"es"` a `LANGS`, e un bottone nel toggle. Il resto (il walk dei `data-i18n`, l'helper `t()`) funziona già.

Nel markup, gli elementi statici sono annotati con attributi:

| Attributo | Cosa traduce |
|---|---|
| `data-i18n` | `textContent` |
| `data-i18n-html` | `innerHTML` (per stringhe con markup) |
| `data-i18n-ph` | `placeholder` |
| `data-i18n-title` | `title` (tooltip) |
| `data-i18n-aria` | `aria-label` |

Le stringhe dinamiche (toast, titoli della board, tempi relativi) passano da `t(key, vars)` con interpolazione `{placeholder}`.

---

## 🧩 Modello dati

Ogni item ha questa shape:

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
  order: { "2026-Q2": 0, "next": 1 },    // posizione per colonna
  createdAt: "2026-02-15T11:00:00.000Z",
  updatedAt: "2026-04-22T10:15:00.000Z"
}
```

Lo state completo salvato in `localStorage` sotto la chiave `roadmap_v1`:

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

La lingua è salvata a parte, sotto la chiave `roadmap_lang` (`"it"` | `"en"`).

### Migrazione

L'app cerca anche la vecchia chiave `roadmap_mvp_final_v1` e la migra automaticamente al primo load.

---

## 🏗 Architettura

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

### Funzioni notevoli

| Funzione | Cosa fa |
|---|---|
| `sanitizeItem(raw, year)` | Valida e normalizza un item importato. Filtra status invalidi, quarter mal formati, tag duplicati. |
| `commitAndRender()` | Helper unificato: `save()` + `render()`. Usato ovunque cambi lo state. |
| `handleDrop(itemId, colKey, insertIndex)` | Drag-drop tra/dentro colonne. Calcola `(prev + next) / 2` come nuovo order, poi normalizza. |
| `showToast(msg, type, opts)` | Notifiche bottom-right. Supporta `actionLabel + onAction` per gli undo. |
| `computeTagCounts()` | Single-pass su tutti gli item filtrati. Restituisce `Map<tag, count>`. |
| `t(key, vars)` | Traduzione con interpolazione `{placeholder}`. Fallback: lingua corrente → en → chiave stessa. |
| `applyStaticI18n()` | Walk dei nodi `[data-i18n*]` e applica la lingua corrente a testo, placeholder, title, aria-label. |

### Dipendenze esterne

Solo **una**, caricata via CDN:

- [html2canvas 1.4.1](https://html2canvas.hertzen.com/) — per l'export PNG. L'app funziona anche senza (gli altri export restano disponibili).

---

## 🎨 Design system

- **Palette**: scala di grigi su sfondo `#0b0c10`, accenti vetrosi con 3 radial-gradient.
- **Status color**: idea (viola pallido), planned (azzurro), in_progress (ambra), done (verde). Riflessi sul left-border delle card.
- **Tipografia**: SF Pro / system font, 11→22px su 6 step. Niente Inter, niente Roboto.
- **Radius**: 8/10/12/14/16/18px per livelli crescenti di container.
- **Densità**: due taglie di pill (small `4-6px / 9-10px`, normal `8-9px / 10-12px`).
- **Glassmorphism**: `backdrop-filter: blur(8-12px)` su topbar, modal, toast.

---

## ♿ Accessibilità

- `aria-modal`, `aria-labelledby`, `aria-live` su toast.
- Focus trap nel modal (Tab/Shift+Tab ciclano dentro).
- Focus-visible custom (outline blu) su tutti gli interactive.
- `aria-invalid` sul title input in stato errore.

---

## 🔒 Limiti e validazione

| Campo | Limite |
|---|---|
| Roadmap name | 120 caratteri |
| Item title | 120 caratteri (obbligatorio) |
| Item description | 500 caratteri |
| Item owner | 80 caratteri |
| Tag per item | massimo 3 |
| Lunghezza tag | 40 caratteri |
| Tag duplicati | dedup case-insensitive automatico |

`localStorage` write failures (quota piena, modalità privata Safari) mostrano un toast d'errore invece di fallire silenziosamente.

---

## 🖨 Stampa / PDF

Il bottone **PDF** apre il dialog di stampa del browser con uno stylesheet `@media print` dedicato: sfondo bianco, bordi grigi, niente topbar/filter/toast.

---

## 📂 File del progetto

```
.
├── index.html              ← l'app
├── sample-roadmap.json     ← dati di esempio per import
├── README.md               ← questo file
└── screenshots/            ← screenshot
```

---

## 📜 Storia del progetto

Refactor da uno script di ~1100 righe a ~1600 righe (con +6 feature, validazione, accessibilità, mobile responsive). Vedi commit history per i passaggi:

1. Refactor critico (rinomina `el`, dedup filter logic, costanti, `commitAndRender`)
2. Bug fix di robustezza (dedup tag, debounce save, schema migration, validation defensiva)
3. Polish GUI fase 1 (3-cluster topbar, filter row dedicata, board header, distinguere colonne, status-as-left-border)
4. Polish GUI fase 2 (tipografia, densità, focus-visible, drag-handle, loading states, subtitle dinamico)
5. Feature finali (validation modal, autocomplete tag, shortcuts footer, drag-sort dentro colonna, mobile responsive)
6. Internazionalizzazione (i18n IT/EN con toggle, `STRINGS` + `t()`, persistenza lingua)

---

## 📝 Licenza

MIT — fai quello che vuoi, niente garanzie.
