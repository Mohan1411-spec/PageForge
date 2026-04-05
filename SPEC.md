# Personal Content Page Builder - Specification

## 1. Project Overview

- **Project Name**: PageForge
- **Type**: Interactive web application (React + Vite)
- **Core Functionality**: A drag-and-drop page builder allowing users to compose personal content pages from configurable blocks
- **Target Users**: Anyone wanting to create simple customizable content pages

---

## 2. Visual & Rendering Specification

### Layout Structure

```
┌─────────────────────────────────────────────────────────────┐
│  Header: "PageForge" branding + Save indicator              │
├──────────────────────┬──────────────────────────────────────┤
│                      │                                      │
│   Block Palette      │         Canvas Area                  │
│   (Left Sidebar)     │         (Main Content)               │
│                      │                                      │
│   - Header Block     │   [Dropped blocks arranged           │
│   - Rich Text Block  │    vertically, each configurable]    │
│   - Image Block      │                                      │
│   - Markdown Block   │                                      │
│                      │                                      │
│                      │                                      │
└──────────────────────┴──────────────────────────────────────┘
```

- **Header**: Fixed top bar, ~60px height, dark theme
- **Palette Sidebar**: Fixed left, 280px width, scrollable block list
- **Canvas Area**: Flexible width, scrollable, light grid background

### Color Palette

| Role        | Color     | Hex       |
|-------------|-----------|-----------|
| Primary     | Deep Indigo | `#4338ca` |
| Secondary   | Slate     | `#475569` |
| Accent      | Amber     | `#f59e0b` |
| Background  | Cool Gray | `#f1f5f9` |
| Surface     | White     | `#ffffff` |
| Text Primary | Slate 900 | `#0f172a` |
| Text Muted  | Slate 500 | `#64748b` |
| Border      | Slate 200 | `#e2e8f0` |
| Drag Active | Indigo 100 | `#e0e7ff` |

### Typography

- **Font Family**: "DM Sans" (headings), "Inter" (body) from Google Fonts
- **Scale**: 12px (xs), 14px (sm), 16px (base), 20px (lg), 24px (xl), 32px (2xl)

### Visual Effects

- **Block Cards**: Subtle shadow (`0 1px 3px rgba(0,0,0,0.1)`), 8px border-radius
- **Drag State**: Elevated shadow (`0 8px 25px rgba(0,0,0,0.15)`), slight scale (1.02)
- **Drop Zone**: Dashed border animation, indigo highlight on hover
- **Transitions**: 200ms ease-out for all interactive states

---

## 3. Component Specification

### Block Types

#### 1. Header Block
- **Purpose**: Display customizable headings
- **Properties**:
  - Text content (string)
  - Header level (h1, h2, h3)
  - Text alignment (left, center, right)
- **Default**: "Your Heading Here" at h1, left-aligned

#### 2. Rich Text Block
- **Purpose**: Freeform text editing with formatting
- **Properties**:
  - HTML content (bold, italic, lists, links)
  - Font size (sm, base, lg)
- **Implementation**: ContentEditable div with toolbar
- **Default**: "Start typing your content..."

#### 3. Image Block
- **Purpose**: Display images from URL
- **Properties**:
  - Image URL (string, validated)
  - Alt text (string)
  - Caption (optional string)
  - Fit mode (cover, contain, fill)
- **Default**: Placeholder gradient with "Add Image" prompt

#### 4. Markdown Block
- **Purpose**: Render markdown content
- **Properties**:
  - Markdown source text
  - Live preview of rendered output
- **Implementation**: textarea + react-markdown preview
- **Default**: "# Hello World\n\nWrite your markdown here..."

### Block Card Wrapper

Each block on canvas includes:
- **Drag Handle**: 6-dot grip icon on left edge
- **Block Type Badge**: Small label showing block type
- **Configuration Panel**: Expandable settings (gear icon)
- **Delete Button**: X icon to remove block
- **Visibility Toggle**: Eye icon to show/hide block

---

## 4. Interaction Specification

### Drag and Drop

- **Palette → Canvas**: Drag block from palette, drop onto canvas
- **Canvas Reorder**: Drag existing blocks to reorder vertically
- **Library**: @dnd-kit/core + @dnd-kit/sortable
- **Drag Overlay**: Ghost preview follows cursor
- **Drop Animation**: 200ms smooth repositioning

### Configuration Flow

1. Click gear icon on block card
2. Configuration panel slides open below block
3. Edit properties via form controls
4. Changes apply immediately (live preview)
5. Click outside or gear again to close

### Persistence

- **Auto-save**: Debounced 500ms after any change
- **Storage Key**: `pageforge_state`
- **Format**: JSON stringified state object
- **Load**: On app mount, hydrate from localStorage
- **Clear**: Button in header to reset to empty state

---

## 5. State Management

### State Shape

```typescript
interface Block {
  id: string;           // nanoid generated
  type: 'header' | 'richtext' | 'image' | 'markdown';
  properties: Record<string, any>;
  visible: boolean;
}

interface AppState {
  blocks: Block[];
  selectedBlockId: string | null;
}
```

### Actions

- `ADD_BLOCK(type, index?)` - Insert new block
- `REMOVE_BLOCK(id)` - Delete block
- `UPDATE_BLOCK(id, props)` - Modify properties
- `REORDER_BLOCKS(ids[])` - Update block order
- `TOGGLE_VISIBILITY(id)` - Show/hide block
- `LOAD_STATE(blocks[])` - Hydrate from storage

---

## 6. Technical Stack

- **Framework**: React 18 with Vite
- **Language**: JavaScript (ES6+)
- **Styling**: Tailwind CSS v3
- **Drag & Drop**: @dnd-kit/core, @dnd-kit/sortable
- **Markdown**: react-markdown
- **ID Generation**: nanoid
- **Icons**: Lucide React

---

## 7. Acceptance Criteria

1. [ ] User can drag each block type from palette to canvas
2. [ ] Blocks can be reordered via drag-and-drop on canvas
3. [ ] Each block type has a functioning configuration panel
4. [ ] Header block: text, level, alignment all work
5. [ ] Rich text block: text editing with basic formatting
6. [ ] Image block: URL input with live preview, alt text, caption
7. [ ] Markdown block: textarea input with live rendered preview
8. [ ] State persists to localStorage and restores on reload
9. [ ] Reset button clears all blocks
10. [ ] UI is responsive and visually polished
11. [ ] No console errors during normal operation
