# Synthia Technical Documentation

**Version:** 3.4  
**Last Updated:** February 7, 2026  
**Author:** Vivek Pandit  

---

## 1. Overview

Synthia is a **single-page web application (SPA)** that helps marketers and creators turn a single image into a full, executable marketing campaign. It runs entirely in the browser and uses Google Gemini for AI-powered analysis and content generation.

Key pillars:

- Brand Persona Engine (Product vs Service, Client vs Own).
- Image-driven content generation with multiple expert frameworks.
- 30-day marketing calendar.
- Campaign Command Center (Checklist + Kanban).
- PDF export with embedded images.
- Persistent state via browser storage.

---

## 2. Architecture

### 2.1 High-Level Diagram

```
┌───────────────────────────────────────────────┐
│                 User Browser                  │
│                                               │
│  ┌─────────────────────────────────────────┐  │
│  │         Synthia SPA (index.html)       │  │
│  │                                         │ │
│  │  HTML5  •  CSS3  •  Vanilla JS (ES6+)   │ │
│  │                                         │ │
│  │  UI Layers:                             │ │
│  │   • Brand Config & Persona              │ │
│  │   • Image Upload & Preview              │ │
│  │   • Content Tabs (Story/Copy/etc.)      │ │
│  │   • Calendar                            │ │
│  │   • Tracker + Kanban Board              │ │
│  │                                         │ │
│  │  State & Logic:                         │ │
│  │   • customPersona                       │ │
│  │   • generatedContent (per tab)          │ │
│  │   • selectedFile, imageBase64           │ │
│  │   • campaignTasks (array of tasks)      │ │
│  │   • trackerMode ('list'|'board')        │ │
│  │   • workingModel (Gemini model cache)   │ │
│  └─────────────────────────────────────────┘  │
│                     │                         │
│                     ▼                         │
│  ┌─────────────────────────────────────────┐  │
│  │          Browser Storage (localStorage) │  │
│  │   • geminiApiKey                        │  │
│  │   • customPersona                       │  │
│  │   • personaWebsite/personaClients       │  │
│  │   • synthiaTrackerTasks                 │  │
│  └─────────────────────────────────────────┘  │
└────────────────────────────┬──────────────────┘
                             ▼
               ┌─────────────────────────┐
               │ Google Gemini API       │
               │  • Text + Vision        │
               │  • generateContent      │
               └─────────────────────────┘
```

### 2.2 Technology Choices

- **HTML5:** semantic structure, file inputs, drag-and-drop.
- **CSS3:** glassmorphism, Flexbox/Grid, responsive design.
- **JavaScript (ES6+):** async/await, arrow functions, template literals.
- **Libraries:**
  - `marked.js` for Markdown -> HTML.
  - `jsPDF` for PDF generation.
- **No Frameworks:** No React/Vue; everything is written in plain JS for portability.

---

## 3. Core Modules & Data Flow

### 3.1 Persona Generation

**Variables:**

```js
let customPersona = null;
let generatedQuestions = [];
let isRefining = false;
```

**Flow:**

1. User enters:
   - `websiteUrl`
   - Optional `clientWebsites`
   - `geminiApiKey`
2. `startPersonaResearch(refining)` builds a `researchPrompt` that:
   - Asks for 3-5 questions.
   - Explicitly requires:
     - Product vs Service.
     - Client vs Own Company.
3. Gemini returns questions -> rendered into the interview modal.
4. On submit:
   - `finalizePersonaGeneration()` concatenates Q&A into `interviewTranscript`.
   - Builds a persona `fullPrompt` with strategic instructions:
     - Products use 4 Ps.
     - Services use 7 Ps.
   - Gemini returns full persona text.
5. Persona is:
   - Displayed in the UI (`displayPersona()` with `marked.parse`).
   - Saved to `localStorage`.

### 3.2 Image Processing

**Variables:**

```js
let selectedFile = null;
let imageBase64 = null;
```

**Flow:**

1. File input or drag-and-drop triggers `handleFile(file)`:
   - Ensures `file.type.startsWith('image/')`.
   - Ensures `file.size <= 4MB`.
   - Uses `FileReader.readAsDataURL(file)` and strips the base64 string.
2. Stores:
   - `selectedFile` (for MIME type and filename).
   - `imageBase64` (for Gemini, UI, and PDF).
3. Updates UI:
   - Preview image.
   - Shows “Analyze & Generate Content” button.

---

## 4. Content Generation

**Variables:**

```js
let generatedContent = {
  persona: '',
  calendar: '',
  tracker: '',
  story: '',
  marketing: '',
  instagram: '',
  blog: '',
  googleads: '',
  podcast: '',
  aeo: ''
};
```

**Key Function: `analyzeImage()`**

1. Validates API key and image presence.
2. Collapses config panel, shows loading overlay.
3. Builds `pContext` string if `customPersona` exists, injecting into prompts.
4. Creates `prompts` object with tailored instructions for:
   - `calendar`, `story`, `marketing`, `instagram`,
   - `blog`, `googleads`, `podcast`, `aeo`.
   - Each prompt embeds product vs service logic and persona context.
5. Calls `callGeminiAPI(apiKey, prompts.calendar, imageBase64)` first:
   - Saves result to `generatedContent.calendar`.
   - Immediately passes it to `parseMarkdownTable` to bootstrap `campaignTasks`.
6. Iterates through remaining prompts:
   - `callGeminiAPI(...)` for each.
   - Adds slight delay to avoid hitting rate limits.
   - Saves into `generatedContent[...]`.
7. Shows `resultsSection`, auto-switches to the `tracker` tab, and scrolls into view.

---

## 5. Tracker & Kanban Engine

### 5.1 Data Model

**Task Object:**

```js
{
  id: number,          // unique (Date.now() + Math.random)
  day: string,         // from calendar table
  channel: string,     // e.g., "Instagram", "Email"
  type: string,        // e.g., "Reel", "Newsletter"
  topic: string,       // hook or subject
  goal: string,        // "Awareness", "Conversion", etc.
  completed: boolean,  // kept for legacy compatibility
  status: 'todo' | 'doing' | 'done',
  notes: string
}
```

**State:**

```js
let campaignTasks = [];
let trackerMode = 'list'; // or 'board'
```

### 5.2 Calendar -> Tasks (Markdown Parsing)

**Function: `parseMarkdownTable(markdown: string) => Task[]`**

- Splits by newline.
- Filters lines starting with `|`.
- Skips header and separator line.
- Splits each row by `|`, trims, and maps to `day`, `channel`, `type`, `topic`, `goal`.
- Returns an array of task objects with default `status: 'todo'` and `completed: false`.

### 5.3 List View Rendering

**Function: `renderTrackerUI()` (List branch)**

- Computes `total`, `completed`, and `progress` percentage.
- Renders a `tracker-dashboard` with:
  - Completion (%)
  - `completed/total`
  - Remaining count.
- Renders `.task-list` with `.task-item` rows:
  - Checkbox -> `toggleTask(index)` toggles `status` between `'todo'` and `'done'`.
  - Notes `textarea` -> `updateTaskNote(index, value)`.

### 5.4 Kanban View Rendering

**Function: `renderTrackerUI()` (Board branch)**

- Groups `campaignTasks` into three arrays based on `status`:
  - `todo`, `doing`, `done`.
- Renders `.kanban-board` with three `.kanban-column` containers:
  - Each column has a header and count.
  - Inside, cards (`.kanban-card`) are rendered with minimal details (day, channel, topic, notes indicator).

### 5.5 Drag & Drop Implementation

Browser-native API (no external library).

Handlers:

```js
window.drag = function(ev, index) {
  ev.dataTransfer.setData("text", index);
  ev.target.classList.add('dragging');
};

window.allowDrop = function(ev) {
  ev.preventDefault();
  ev.currentTarget.classList.add('drag-over');
};

window.drop = function(ev, newStatus) {
  ev.preventDefault();
  document.querySelectorAll('.kanban-column').forEach(c => c.classList.remove('drag-over'));
  const index = ev.dataTransfer.getData("text");
  if (index !== '') {
    const task = campaignTasks[index];
    task.status = newStatus;
    task.completed = (newStatus === 'done');
    saveTrackerState();
    renderTrackerUI();
  }
};
```

Global listeners remove styling on `dragend`/`dragleave` to keep the UI clean.

### 5.6 Persistence & Reset

**Persistence:**

```js
function saveTrackerState() {
  localStorage.setItem('synthiaTrackerTasks', JSON.stringify(campaignTasks));
}

// On load:
const savedTasks = localStorage.getItem('synthiaTrackerTasks');
if (savedTasks) {
  const tasks = JSON.parse(savedTasks);
  campaignTasks = tasks.map(t => ({
    ...t,
    status: t.status || (t.completed ? 'done' : 'todo')
  }));
}
```

**Reset:**

```js
function resetTracker() {
  if (confirm("Are you sure you want to reset all progress?")) {
    campaignTasks.forEach(t => { t.status = 'todo'; t.completed = false; t.notes = ''; });
    saveTrackerState();
    renderTrackerUI();
  }
}
```

---

## 6. PDF Generation

### 6.1 Library

- `jsPDF` 2.5.1 (UMD build loaded from CDN).

### 6.2 Export Logic

**Function: `exportCurrentToPDF()`**

1. Initializes `jsPDF` in A4 portrait.
2. Writes title based on `contentTitle`.
3. If active tab is a content type (not persona/tracker) and `imageBase64` is present:
   - Constructs data URI `data:${mime};base64,${imageBase64}`.
   - Calculates dimensions to fit within margins while preserving aspect ratio.
   - Calls `doc.addImage(...)`.
4. Chooses content:
   - For tracker:
     - Renders each task with status and notes as plain text.
   - For other tabs:
     - Takes `activeContent.innerText`.
5. Uses `splitTextToSize` + pagination to write text across pages.
6. Calls `doc.save(...)`.

---

## 7. API Integration (Gemini)

### 7.1 Model Discovery

**Function: `callGeminiAPI(apiKey, prompt, imageBase64?)`**

- If `workingModel` cached:
  - Calls `executeGeneration(apiKey, workingModel, ...)`.
- Else:
  - Fetches model list from `v1beta/models`.
  - Filters for models with `generateContent`.
  - Sorts by preference (Gemini 2.x, then 1.5).
  - Iterates through candidates until one works.
  - Caches winning model in `workingModel`.

### 7.2 Request Shape

```json
{
  "contents": [
    {
      "parts": [
        { "text": "prompt here" },
        {
          "inline_data": {
            "mime_type": "image/jpeg",
            "data": "BASE64_STRING"
          }
        }
      ]
    }
  ],
  "generationConfig": {
    "temperature": 0.9,
    "topK": 40,
    "topP": 0.95,
    "maxOutputTokens": 8192
  }
}
```

### 7.3 Error Handling

- If model listing fails:
  - Falls back to a hardcoded small list of model names.
- On non-OK responses:
  - Throws Error with API message if available.
- The UI shows a toast message and restores buttons.

---

## 8. State Management

### 8.1 Global Variables

```js
let selectedFile, imageBase64;
let customPersona;
let generatedContent;
let campaignTasks;
let trackerMode;
let activeTab;
let workingModel;
let generatedQuestions;
let isRefining;
```

### 8.2 Storage Keys

- `geminiApiKey`
- `customPersona`
- `personaWebsite`
- `personaClients`
- `synthiaTrackerTasks`

Images are **not** persisted to avoid bloating localStorage.

---

## 9. Security & Privacy

- No backend, no database, no server-side session.
- API key is scoped to the domain and only sent to `generativelanguage.googleapis.com` over HTTPS.
- Content Security Policy (recommended if self-hosting) can restrict scripts, fonts, and connections to known CDNs and Gemini only.
- No user-generated HTML is injected via `innerHTML` without sanitization; Markdown rendering is handled by `marked`, and inputs are not directly executed.

---

## 10. Browser Compatibility

- Designed for modern evergreen browsers.
- Requires:
  - `fetch`
  - `async/await`
  - `FileReader`
  - `localStorage`
  - HTML5 Drag and Drop (for Kanban)

Mobile:

- Uses responsive layouts for small screens.
- Kanban board becomes horizontally scrollable with each column near full width.

---

## 11. Future Enhancements

- Multiple images per campaign.
- Image compression before sending to Gemini and embedding in PDFs.
- Export/import of tracker state as JSON for collaboration.
- Optional integration with external analytics or CRMs (via webhooks/API) while keeping core app local-first.

---

Synthia is designed to be **inspectable**, **hackable**, and **self-contained**: a single HTML file and a logo that gives you a full AI-assisted marketing operating system in your browser.
