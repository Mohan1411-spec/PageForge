# PageForge — Personal Content Builder

> A browser-based, block-style page editor. No backend. No account. Just write.

![React](https://img.shields.io/badge/React-19-61DAFB?logo=react&logoColor=white)
![Vite](https://img.shields.io/badge/Vite-8-646CFF?logo=vite&logoColor=white)
![Tailwind CSS](https://img.shields.io/badge/Tailwind_CSS-4-38BDF8?logo=tailwindcss&logoColor=white)
![Netlify](https://img.shields.io/badge/Deployed_on-Netlify-00C7B7?logo=netlify&logoColor=white)
![License](https://img.shields.io/badge/license-MIT-green)

---

## What is PageForge?

PageForge is a personal page builder that lets you construct rich, multi-block documents in the browser — dragging, dropping, and editing content blocks (text, headings, images, code, callouts, and more) with every change automatically saved to `localStorage`. No server, no sign-up, no sync delay.

Think of it as a lightweight personal Notion that lives entirely in your browser.

---

## Features

- **Block-based editing** — Text, Headings (H1–H3), Images, Code, Dividers, Quotes, Callouts
- **Drag-and-drop reordering** — Powered by [dnd-kit](https://dnd-kit.com/) with accessible keyboard support
- **Markdown rendering** — Text and Quote blocks render full Markdown via `react-markdown`
- **Multiple pages** — Create, rename, and delete named pages
- **Zero-server persistence** — All data lives in `localStorage`, instantly restored on next visit
- **Dark mode** — System-aware with manual toggle
- **Static deploy** — Ships as a pure `dist/` folder; works on Netlify, Vercel, GitHub Pages

---

## Getting Started

```bash
# 1. Clone
git clone https://github.com/your-username/pageforge.git
cd pageforge

# 2. Install dependencies
npm install

# 3. Start dev server
npm run dev
# → http://localhost:5173
```

### Build for production

```bash
npm run build     # outputs to /dist
npm run preview   # preview the production build locally
```

### Deploy to Netlify

Push to any Git remote connected to Netlify. The included `netlify.toml` handles everything:

```toml
[build]
  command = "npm run build"
  publish = "dist"

[[redirects]]
  from = "/*"
  to   = "/index.html"
  status = 200
```

The catch-all redirect ensures page refreshes and direct URL access work correctly for this SPA.

---

## Tech Stack

| Package | Version | Role |
|---|---|---|
| [React](https://react.dev) | 19.x | UI framework |
| [Vite](https://vite.dev) | 8.x | Dev server & bundler |
| [Tailwind CSS](https://tailwindcss.com) | 4.x (Vite plugin) | Utility-first styling |
| [@dnd-kit/core](https://dnd-kit.com) + sortable | 6.x / 10.x | Accessible drag-and-drop |
| [react-markdown](https://github.com/remarkjs/react-markdown) | 10.x | Markdown block rendering |
| [lucide-react](https://lucide.dev) | 1.x | Icon set |
| [nanoid](https://github.com/ai/nanoid) | 5.x | Collision-resistant block IDs |

---

## Project Structure

```
src/
├── components/
│   ├── blocks/          # BlockRenderer, BlockEditor, per-type block components
│   ├── canvas/          # Canvas, BlockList, DragOverlay
│   ├── sidebar/         # PageList, BlockOutline
│   ├── toolbar/         # AddBlock menu, formatting toolbar
│   └── ui/              # Button, Input, Icon, Modal primitives
├── context/
│   ├── PageContext.jsx   # Global state provider
│   ├── reducer.js        # Pure reducer + action creators
│   └── persistence.js    # initState() + schema migration helpers
├── hooks/
│   ├── useActiveBlock.js
│   ├── useKeyboardShortcuts.js
│   └── usePage.js
├── lib/
│   ├── defaults.js       # Default page & block templates
│   └── blockTypes.js     # Block type registry
├── App.jsx
└── main.jsx
```

---

## UI / UX Design

### Design Philosophy

The interface is guided by **calm technology**: the canvas should recede so content stays prominent. There are no dashboards, analytics panels, or distracting chrome — just an outline sidebar and a wide editing canvas.

### Typefaces

| Font | Usage |
|---|---|
| **DM Sans** | All UI chrome — sidebars, toolbars, headings |
| **Inter** | Form inputs, metadata labels, property panels |
| **JetBrains Mono** | Code blocks exclusively |

### Layout

The editor is split into two panes: a narrow left sidebar (block outline + page list) and a wide central canvas. Blocks are a **flat, ordered list** — no tree hierarchy — which keeps the mental model simple and reordering predictable.

### Micro-interactions

- Blocks animate in on creation with a short scale + fade transition
- The active block shows a subtle left-border accent and card elevation
- Drag previews match the exact rendered size of the dragged block
- Toolbar icons use Lucide at 16 px — dense without being cramped

---

## Component Design

### Block Types

Every block is a plain JavaScript object:

```js
{ id: string, type: BlockType, content: string, meta: object }
```

| Type | Content | Rendered As |
|---|---|---|
| `text` | Markdown string | `react-markdown` output |
| `heading` | Plain string + level (1–3) | `<h1>` / `<h2>` / `<h3>` |
| `image` | URL + alt text | `<img>` with lazy loading |
| `code` | Code string + language | Syntax-highlighted `<pre><code>` |
| `divider` | *(none)* | `<hr>` |
| `quote` | Markdown string | Styled blockquote |
| `callout` | Markdown + icon + color | Colored callout card |

### BlockEditor vs BlockRenderer

**`BlockRenderer`** is a pure display component — it takes a block object and returns the appropriate React subtree. It is memoised with `React.memo` so it only re-renders when the block object reference changes.

**`BlockEditor`** mounts in place of `BlockRenderer` when a block enters the "active" state. It renders the appropriate input widget (textarea for text/quote, controlled inputs for heading/code, URL field + preview for image). On blur or Escape, the editor commits the change and the block reverts to its rendered view.

This double-buffer pattern keeps the render tree shallow and avoids unified-component mode flags.

### Drag-and-Drop

dnd-kit was chosen over react-beautiful-dnd because it is actively maintained, works with React 19's concurrent renderer, and ships with zero default CSS opinions.

```
DndContext              → at BlockList level, provides the drag event bus
  SortableContext       → verticalListSortingStrategy for the ordered list
    Block (useSortable) → transform styles + drag handle ref per block
  DragOverlay           → portal clone that floats above other content
```

On `dragEnd`, `arrayMove` from `@dnd-kit/sortable` reorders the blocks array in a single state update, which propagates to `localStorage` automatically via the persistence layer.

---

## State Management

### Global State Shape

All application state lives in a single React Context (`PageContext`) produced by a custom hook (`usePageStore`) over `useReducer`:

```js
{
  pages: Page[],           // all named pages
  activePageId: string,    // which page is visible
  activeBlockId: string,   // which block is in edit mode
  theme: 'light' | 'dark'  // global colour scheme
}
```

Each `Page`:

```js
{ id, name, blocks: Block[], createdAt: number, updatedAt: number }
```

### Why Context + useReducer

For a single-user, offline-first project of this scale, Context + useReducer provides the right fidelity without an external dependency:

| Option | Decision |
|---|---|
| **Context + useReducer** | ✅ Chosen — zero dependencies, colocated logic, easy to test |
| Zustand | Rejected — minimal API is nice, but overkill here |
| Redux Toolkit | Rejected — too much ceremony for one user, no DevTools need |
| Jotai / Recoil | Rejected — atom abstraction adds a second mental layer |

Re-render risk is mitigated by splitting into two contexts: a **data context** (pages, activePageId) and an **actions context** (dispatch functions). Toolbar buttons that only call actions never re-render on data changes.

### Reducer Actions

| Action | Payload | Effect |
|---|---|---|
| `ADD_BLOCK` | type, position | Inserts new block with nanoid |
| `UPDATE_BLOCK` | id, partial block | Merges partial into existing block |
| `DELETE_BLOCK` | id | Removes block; shifts focus |
| `MOVE_BLOCK` | fromIndex, toIndex | Reorders via arrayMove |
| `SET_ACTIVE_BLOCK` | id \| null | Updates focus state |
| `ADD_PAGE` | name | Creates new empty page |
| `DELETE_PAGE` | id | Removes page; falls back to first |
| `RENAME_PAGE` | id, name | Updates name + updatedAt |
| `SET_THEME` | `'light'` \| `'dark'` | Toggles root theme class |

---

## Persistence

### Strategy: Versioned localStorage

All data is written to a single key — `pageforge_v1` — on every state change. The `v1` suffix enables clean schema migration: if a future release changes the shape, the app detects the missing key, reads a legacy key, migrates, and writes the new schema.

### Write Path

A `useEffect` in `PageContext` serialises state on every change:

```js
useEffect(() => {
  localStorage.setItem('pageforge_v1', JSON.stringify(state));
}, [state]);
```

Because the reducer returns a new reference on every genuine mutation (immutable updates), this effect never fires spuriously.

### Read Path (Hydration)

The initial state is derived from a lazy initialiser — meaning the `localStorage` read happens exactly once, at mount:

```js
const [state, dispatch] = useReducer(reducer, undefined, initState);

function initState() {
  try {
    const raw = localStorage.getItem('pageforge_v1');
    if (raw) return JSON.parse(raw);
  } catch (_) {}
  return DEFAULT_STATE;
}
```

### Current Limitations

- **~5 MB cap** — heavy use of base64 image data URLs will hit the `localStorage` limit; a future release should store image URLs or use the [Origin Private File System (OPFS)](https://developer.mozilla.org/en-US/docs/Web/API/File_System_API/Origin_private_file_system) API
- **No conflict resolution** — last write wins if the same origin is open in two tabs
- **No undo / redo** — planned via an action history stack in the reducer

---

## Roadmap

- [ ] Undo / Redo (`Ctrl+Z` / `Ctrl+Shift+Z`) via command history
- [ ] Slash command palette — insert blocks by typing `/`
- [ ] Block templates — save an arrangement as a reusable template
- [ ] Export page as Markdown or PDF
- [ ] Optional cloud sync via Cloudflare Workers + KV
- [ ] Mobile drag-to-reorder using dnd-kit touch sensors
- [ ] JSON export / import for manual backups

---

## Contributing

1. Fork the repo
2. Create a feature branch: `git checkout -b feat/my-feature`
3. Commit your changes: `git commit -m "feat: add my feature"`
4. Push and open a pull request

Please run `npm run lint` before submitting.

---

## License

MIT © 2025 — see [LICENSE](./LICENSE) for details.
