# Synthia Technical Documentation

**Version:** 3.4  
**Last Updated:** February 6, 2026  
**Architecture:** Single-Page Application (SPA)

---

## Table of Contents

1. [Architecture Overview](#architecture-overview)
2. [Core Technologies](#core-technologies)
3. [Data Flow](#data-flow)
4. [Image Processing Pipeline](#image-processing-pipeline)
5. [Kanban Feature Architecture (NEW v3.4)](#kanban-feature-architecture-new-v34)
6. [API Integration](#api-integration)
7. [State Management](#state-management)
8. [PDF Generation](#pdf-generation)
9. [Security](#security)

---

## Kanban Feature Architecture (NEW v3.4)

v3.4 enhances the Tracker feature by introducing a Kanban Board view using the native HTML5 Drag and Drop API.

### 1. Data Model Evolution
**Old Model (v3.3):**
```javascript
{ completed: boolean, notes: string }
```

**New Model (v3.4):**
```javascript
{ 
  status: 'todo' | 'doing' | 'done', 
  completed: boolean, // Kept for legacy compatibility
  notes: string 
}
```

### 2. View Management
A new state variable `trackerMode` ('list' or 'board') controls the rendering logic.
- **List Mode:** Renders a vertical stack of checkboxes.
- **Board Mode:** Renders 3 columns (`div.kanban-column`) and draggable cards (`div.kanban-card`).

### 3. Drag and Drop Logic
Implemented using native events to ensure zero dependencies:
- `ondragstart`: Sets `dataTransfer` with the task index and adds `.dragging` class.
- `ondragover`: Calls `preventDefault()` to allow dropping and adds `.drag-over` visual cue.
- `ondrop`: Retrieves task index, updates `status` in state, re-renders UI, and saves to `localStorage`.

### 4. Migration Strategy
On application load, the system checks for existing tasks. If a task lacks the `status` property, it is auto-assigned based on the `completed` boolean:
```javascript
status: t.status || (t.completed ? 'done' : 'todo')
```

---

## Architecture Overview

Synthia is a client-side-only application with zero server dependencies. All processing occurs in the user's browser.

```
┌─────────────────────────────────────────────────────────┐
│                    User's Browser                        │
│  ┌────────────────────────────────────────────────────┐ │
│  │              Synthia SPA (index.html)              │ │
│  │                                                    │ │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐        │ │
│  │  │  HTML5   │  │   CSS3   │  │JavaScript│        │ │
│  │  │ Markup   │  │ Styling  │  │  Logic   │        │ │
│  │  └──────────┘  └──────────┘  └──────────┘        │ │
│  │                                                    │ │
│  │  ┌──────────────────────────────────────────────┐ │ │
│  │  │        State Management (Variables)          │ │ │
│  │  │  • campaignTasks (Array of Objects)          │ │ │
│  │  │  • trackerMode ('list' | 'board') ◄── v3.4   │ │ │
│  │  └──────────────────────────────────────────────┘ │ │
│  └────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────┘
```

---

## Data Flow

### 1. Drag & Drop Interaction
```
User drags Card A from 'To Do' to 'In Progress'
  ↓
`drop` event fires
  ↓
Update Task A: { status: 'doing' }
  ↓
`saveTrackerState()` -> writes to localStorage
  ↓
`renderTrackerUI()` -> Re-renders columns
```

---

## Appendix: Code Statistics

### v3.4 Codebase
- **Total Lines:** ~1,600 (HTML + CSS + JS)
- **JavaScript:** ~1,100 lines
- **CSS:** ~400 lines
- **HTML:** ~100 lines

### Kanban Feature Addition
- **New CSS:** Grid layout for board, glassmorphism for cards, drag states
- **New JS:** Drag event handlers, state migration, view toggling
- **Lines Added:** ~150
- **Complexity:** Medium (DOM manipulation + Event handling)

---

**Documentation Maintained By:** Vivek Pandit  
**Last Code Review:** February 6, 2026  
**Next Review:** March 2026
