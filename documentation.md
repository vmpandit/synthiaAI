# Technical Documentation - AI Scene Detective

Complete technical reference for developers, power users, and system integrators.

---

## Table of Contents

1. [System Architecture](#system-architecture)
2. [File Structure](#file-structure)
3. [API Integration](#api-integration)
4. [State Management](#state-management)
5. [UI Components](#ui-components)
6. [Prompt Engineering](#prompt-engineering)
7. [Data Flow](#data-flow)
8. [Security](#security)
9. [Browser Compatibility](#browser-compatibility)
10. [Troubleshooting](#troubleshooting)
11. [API Reference](#api-reference)

---

## System Architecture

### High-Level Overview

```
┌─────────────────────────────────────────────────────────────┐
│                        User Browser                          │
│  ┌───────────────────────────────────────────────────────┐  │
│  │                    index.html                         │  │
│  │  ┌─────────────┐  ┌──────────────┐  ┌─────────────┐ │  │
│  │  │   HTML/UI   │  │  JavaScript  │  │     CSS     │ │  │
│  │  │  Structure  │  │    Logic     │  │   Styling   │ │  │
│  │  └─────────────┘  └──────────────┘  └─────────────┘ │  │
│  └───────────────────────────────────────────────────────┘  │
│                            ↓↑                                │
│  ┌───────────────────────────────────────────────────────┐  │
│  │               localStorage (Persistence)              │  │
│  │  - API Key  - Persona  - Website URL  - Settings     │  │
│  └───────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
                            ↓↑
┌─────────────────────────────────────────────────────────────┐
│                    Google Gemini API                         │
│  - Model Selection (2.0 Flash / 1.5 Pro / 1.5 Flash)       │
│  - Vision + Text Processing                                  │
│  - Rate Limiting: ~60 requests/min (free tier)              │
└─────────────────────────────────────────────────────────────┘
```

### Technology Stack

| Component | Technology | Version | Purpose |
|-----------|-----------|---------|---------|
| Frontend | HTML5 | Latest | Structure |
| Styling | CSS3 | Latest | Glassmorphism UI |
| Scripting | JavaScript (ES6+) | Latest | Logic & API calls |
| Markdown | marked.js | 4.0+ | Content rendering |
| PDF Export | jsPDF | 2.5+ | Document generation |
| AI Engine | Google Gemini | 2.0/1.5 | Content generation |
| Storage | localStorage | HTML5 | Persistence |

---

## File Structure

### Single-File Architecture

```
index.html
├── <head>
│   ├── <meta> tags (viewport, charset)
│   ├── <title>
│   ├── <script> marked.js CDN
│   └── <style> Complete CSS (inline)
│
└── <body>
    ├── Background layer (.liquid-background)
    ├── App container (.app-container)
    │   ├── Header (title + subtitle)
    │   ├── Config Panel (persona generation)
    │   ├── Upload Panel (image handling)
    │   ├── Results Section (7 content tabs)
    │   └── Footer
    ├── Modals (API key help, interview)
    ├── Toast (notifications)
    ├── <script> jsPDF CDN
    └── <script> Main application logic (inline)
```

### Why Single-File?

✅ **Advantages:**
- Zero installation (download and open)
- No build process needed
- Easy to share/distribute
- Self-contained (works offline after first load)

❌ **Trade-offs:**
- Large file size (~50KB)
- No code splitting
- Harder to maintain long-term

---

## API Integration

### Google Gemini API

**Base URL:**
```
https://generativelanguage.googleapis.com/v1beta/
```

**Endpoints Used:**

1. **List Models** (Model Discovery)
```http
GET /models?key={API_KEY}

Response:
{
  "models": [
    {
      "name": "models/gemini-2.0-flash",
      "supportedGenerationMethods": ["generateContent"],
      ...
    }
  ]
}
```

2. **Generate Content** (Main Endpoint)
```http
POST /models/{MODEL_NAME}:generateContent?key={API_KEY}

Request Body:
{
  "contents": [
    {
      "parts": [
        {"text": "Your prompt here"},
        {"inline_data": {"mime_type": "image/jpeg", "data": "base64..."}}
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

Response:
{
  "candidates": [
    {
      "content": {
        "parts": [
          {"text": "Generated content here..."}
        ]
      }
    }
  ]
}
```

### Model Selection Strategy

```javascript
// Priority order (highest to lowest)
const MODEL_PRIORITY = [
  'gemini-2.0-flash',     // Fastest, newest (preferred)
  'gemini-1.5-pro',       // Most capable (fallback)
  'gemini-1.5-flash'      // Most compatible (last resort)
];

async function callGeminiAPI(apiKey, prompt, imageBase64) {
  // Try auto-discovery first
  if (workingModel) {
    return executeGeneration(apiKey, workingModel, prompt, imageBase64);
  }

  // Discovery phase
  const models = await fetch(`/v1beta/models?key=${apiKey}`);
  const sorted = sortByPriority(models);

  // Try each model until one works
  for (const model of sorted) {
    try {
      const result = await executeGeneration(apiKey, model, prompt, imageBase64);
      workingModel = model; // Cache successful model
      return result;
    } catch (e) {
      continue; // Try next model
    }
  }

  // Fallback to hardcoded list
  return fallbackHardcodedGeneration(apiKey, prompt, imageBase64);
}
```

### Error Handling

```javascript
// Error types and responses
const ERROR_CODES = {
  400: 'Invalid request (check prompt format)',
  401: 'Invalid API key',
  403: 'API key lacks permissions',
  429: 'Rate limit exceeded (wait 60s)',
  500: 'Gemini server error (retry)',
  503: 'Service temporarily unavailable'
};

// Retry logic with exponential backoff
async function executeGeneration(apiKey, model, prompt, image, retries = 3) {
  for (let i = 0; i < retries; i++) {
    try {
      const response = await fetch(/* ... */);
      if (response.ok) return await response.json();

      if (response.status === 429) {
        await sleep(Math.pow(2, i) * 1000); // 1s, 2s, 4s
        continue;
      }

      throw new Error(ERROR_CODES[response.status] || 'Unknown error');
    } catch (e) {
      if (i === retries - 1) throw e;
    }
  }
}
```

---

## State Management

### Global State Object

```javascript
// Application state (in-memory)
const state = {
  // File handling
  selectedFile: null,          // File object
  imageBase64: null,           // Base64 encoded image

  // Persona
  customPersona: null,         // Full persona markdown

  // Generated content
  generatedContent: {
    persona: '',
    story: '',
    marketing: '',
    instagram: '',
    blog: '',
    googleads: '',
    podcast: ''
  },

  // UI state
  activeTab: 'persona',        // Current content tab
  workingModel: null,          // Cached Gemini model
  generatedQuestions: [],      // Interview questions
  isRefining: false            // Refinement mode flag
};
```

### Persistence (localStorage)

```javascript
// Saved to browser (survives page reloads)
localStorage Schema:
{
  "geminiApiKey": "AIza...",                    // String
  "customPersona": "## 1. COMPANY...",          // String (markdown)
  "personaWebsite": "https://example.com",      // String (URL)
  "personaClients": "https://client1.com
..."  // String (newline-separated)
}

// Save operations
function savePersona(persona) {
  localStorage.setItem('customPersona', persona);
  localStorage.setItem('personaWebsite', websiteUrl);
  localStorage.setItem('personaClients', clientWebsites);
}

// Load operations
window.addEventListener('load', () => {
  const savedPersona = localStorage.getItem('customPersona');
  if (savedPersona) {
    customPersona = savedPersona;
    displayPersona(savedPersona);
  }

  els.websiteUrl.value = localStorage.getItem('personaWebsite') || '';
  els.apiKey.value = localStorage.getItem('geminiApiKey') || '';
});
```

### State Transitions

```javascript
// State machine for persona workflow
State: INITIAL
  → [User enters website] → State: RESEARCHING

State: RESEARCHING
  → [AI generates questions] → State: INTERVIEWING

State: INTERVIEWING
  → [User submits answers] → State: SYNTHESIZING

State: SYNTHESIZING
  → [Persona complete] → State: PERSONA_READY

State: PERSONA_READY
  → [User uploads image] → State: ANALYZING_IMAGE

State: ANALYZING_IMAGE
  → [Content generated] → State: CONTENT_READY

State: CONTENT_READY
  → [User clicks "Upload Different Photo"] → State: PERSONA_READY
  → [User clicks "Refine"] → State: RESEARCHING (refinement mode)
  → [User clicks "Reset"] → State: INITIAL
```

---

## UI Components

### Component Hierarchy

```
App
├── LiquidBackground (animated gradients)
├── Header (title + subtitle)
├── ConfigPanel (collapsible)
│   ├── PersonaInputSection
│   │   ├── ApiKeyInput
│   │   ├── WebsiteUrlInput
│   │   ├── ClientWebsitesTextarea
│   │   └── GeneratePersonaButton
│   └── PersonaSection (hidden until generated)
│       ├── PersonaContent (markdown)
│       └── ActionButtons (Refine, Copy, PDF)
├── UploadPanel
│   ├── UploadZone (drag-drop area)
│   ├── PreviewContainer
│   │   ├── PreviewImage
│   │   └── RemoveButton (X)
│   ├── LoadingOverlay
│   └── ActionButtons (Analyze, Reset Image)
├── ResultsSection (hidden until content generated)
│   ├── SegmentedControl (tabs)
│   ├── ContentArea
│   │   ├── ContentHeader
│   │   └── ContentText (markdown)
│   └── ActionButtons (Copy, PDF)
├── Modals
│   ├── ApiKeyModal
│   └── InterviewModal
│       ├── InterviewQuestions (dynamic)
│       └── SubmitButton
└── Toast (notifications)
```

### CSS Architecture

**Design System Variables:**
```css
:root {
  /* Colors */
  --system-blue: #0A84FF;
  --system-indigo: #5E5CE6;
  --system-purple: #BF5AF2;
  --system-pink: #FF375F;
  --system-teal: #64D2FF;
  --system-green: #32D74B;

  /* Glassmorphism */
  --glass-thick: rgba(30, 30, 30, 0.6);
  --glass-thin: rgba(60, 60, 60, 0.3);
  --glass-border: rgba(255, 255, 255, 0.1);
  --glass-shadow: 0 12px 40px rgba(0, 0, 0, 0.4);

  /* Typography */
  --font-primary: -apple-system, BlinkMacSystemFont, "SF Pro Display", system-ui;
  --text-primary: #FFFFFF;
  --text-secondary: rgba(255, 255, 255, 0.65);
  --text-tertiary: rgba(255, 255, 255, 0.45);

  /* Spacing */
  --radius-xl: 28px;
  --radius-l: 20px;
  --radius-m: 14px;
}
```

**Glassmorphism Effect:**
```css
.glass-panel {
  background: var(--glass-thick);
  backdrop-filter: blur(40px) saturate(180%);
  -webkit-backdrop-filter: blur(40px) saturate(180%);
  border: 1px solid var(--glass-border);
  border-radius: var(--radius-xl);
  box-shadow: var(--glass-shadow);
}
```

**Animations:**
```css
/* Liquid background movement */
@keyframes liquid-move {
  0% { transform: scale(1) translate(0, 0); }
  50% { transform: scale(1.1) translate(20px, -20px); }
  100% { transform: scale(1) translate(0, 0); }
}

/* Modal slide-up */
@keyframes slideUp {
  from { opacity: 0; transform: translateY(20px); }
  to { opacity: 1; transform: translateY(0); }
}

/* Spinner rotation */
@keyframes spin {
  to { transform: rotate(360deg); }
}
```

---

## Prompt Engineering

### Prompt Template Structure

```javascript
const promptTemplate = `
[ROLE DEFINITION]
You are [specific expert persona] with [credentials/experience].

[CONTEXT INJECTION]
${personaContext ? `=== BRAND PERSONA ===
${customPersona}
===` : ''}

[TASK DESCRIPTION]
Analyze this image and [specific task].

[FRAMEWORK SPECIFICATION]
Use these [named frameworks/methods]:
1. [Framework 1 with specific steps]
2. [Framework 2 with specific rules]
3. [Framework 3 with examples]

[OUTPUT REQUIREMENTS]
- Format: [markdown/plain text]
- Length: [word count]
- Structure: [sections/headings]
- Tone: [conversational/professional]

[CONSTRAINTS]
- Must include: [specific elements]
- Must avoid: [prohibited content]
- Quality bar: [specific criteria]
`;
```

### Example: Story Generation Prompt

```javascript
const storyPrompt = `
You are Robert McKee, master of story structure, combined with the observational skills of a CSI detective.

${personaContext}

Analyze this image and create a COMPELLING NARRATIVE STORY (800-1,200 words) using these frameworks:

1. THREE-ACT STRUCTURE (McKee):
   - Act 1: Setup (Who, What, Where, When - establish context)
   - Act 2: Confrontation (The challenge, conflict, or journey)
   - Act 3: Resolution (The outcome, transformation, or revelation)

2. HERO'S JOURNEY ELEMENTS:
   - Ordinary world → Call to adventure → Crossing threshold
   - Tests and trials → Transformation → Return with wisdom

3. CINEMATIC OBSERVATION:
   - Notice EVERY detail (lighting, colors, textures, expressions)
   - Environmental context and atmosphere
   - Emotional undertones and subtext
   - Background elements that tell a story

4. STORYTELLING TECHNIQUES:
   - Use sensory details (what you see, implied sounds/smells)
   - Create character depth (motivations, backstory hints)
   - Build tension and pacing
   - Include dialogue if people are present
   - Use metaphors and vivid language

OUTPUT FORMAT:
- Write in flowing narrative prose
- 800-1,200 words
- Divided into clear beginning, middle, end
- Make it engaging and visual
- Connect details to larger meaning
`;
```

### Prompt Optimization Techniques

1. **Persona Injection:**
   ```javascript
   const personaContext = customPersona 
     ? `

=== BRAND PERSONA ===
${customPersona}
=== END PERSONA ===

CRITICAL: Use this persona.

`
     : '';
   ```

2. **Example-Driven Learning:**
   ```javascript
   Examples of good headlines:
   - "How to [Benefit] Even If [Objection]"
   - "The Secret of [Desire]"

   Now create 5 headlines for this image.
   ```

3. **Constraint Framing:**
   ```javascript
   CRITICAL: Do not use generic phrases.
   REQUIRED: Include specific numbers/data.
   FORBIDDEN: Marketing clichés like "cutting-edge" or "revolutionary".
   ```

4. **Output Format Specification:**
   ```javascript
   Return ONLY raw JSON: ["Q1", "Q2", "Q3"]
   Do not include markdown code fences.
   Do not include explanatory text.
   ```

---

## Data Flow

### Persona Generation Flow

```
┌─────────────────────────────────────────────────────────────┐
│ 1. User Input                                                │
│    - Website URL: "https://networkscr.com"                  │
│    - Client URLs: "https://client1.com"                     │
└────────────────────────┬────────────────────────────────────┘
                         ↓
┌─────────────────────────────────────────────────────────────┐
│ 2. Research API Call                                         │
│    - Model: gemini-2.0-flash                                │
│    - Prompt: "Analyze website + generate questions"         │
│    - Response: ["Q1?", "Q2?", "Q3?"]                        │
└────────────────────────┬────────────────────────────────────┘
                         ↓
┌─────────────────────────────────────────────────────────────┐
│ 3. Interview Modal Display                                   │
│    - Render questions as textareas                          │
│    - User types answers                                      │
│    - Collect answers: {Q1: A1, Q2: A2, Q3: A3}             │
└────────────────────────┬────────────────────────────────────┘
                         ↓
┌─────────────────────────────────────────────────────────────┐
│ 4. Synthesis API Call                                        │
│    - Model: gemini-2.0-flash                                │
│    - Prompt: "Website + Interview → Persona"                │
│    - Response: "## 1. COMPANY PROFILE
..."                 │
└────────────────────────┬────────────────────────────────────┘
                         ↓
┌─────────────────────────────────────────────────────────────┐
│ 5. Storage & Display                                         │
│    - Save to localStorage.customPersona                      │
│    - Render with marked.parse()                             │
│    - Show "Persona Ready" toast                             │
└─────────────────────────────────────────────────────────────┘
```

### Content Generation Flow

```
┌─────────────────────────────────────────────────────────────┐
│ 1. Image Upload                                              │
│    - User drags/clicks to upload                            │
│    - Validate: type (image/*), size (<4MB)                  │
│    - Convert to base64                                       │
└────────────────────────┬────────────────────────────────────┘
                         ↓
┌─────────────────────────────────────────────────────────────┐
│ 2. Sequential API Calls (7 total)                           │
│    For each content type:                                    │
│      - Load persona from localStorage                        │
│      - Build prompt (persona + framework + image)           │
│      - Call Gemini API with vision                          │
│      - Store result in generatedContent[type]               │
│      - Update UI loading state                              │
│      - Wait 1.2 seconds (rate limit buffer)                 │
└────────────────────────┬────────────────────────────────────┘
                         ↓
┌─────────────────────────────────────────────────────────────┐
│ 3. Display Results                                           │
│    - Show results section                                    │
│    - Activate "Story" tab by default                        │
│    - Render content with marked.parse()                     │
│    - Enable Copy/PDF buttons                                │
└─────────────────────────────────────────────────────────────┘
```

---

## Security

### API Key Handling

```javascript
// Stored in localStorage (browser-local, not synced)
localStorage.setItem('geminiApiKey', userInputKey);

// Never logged to console in production
// Never sent to any server except Google Gemini
// Never embedded in HTML/JavaScript source

// Auto-save on blur
els.apiKey.addEventListener('blur', (e) => {
  if (e.target.value.trim()) {
    localStorage.setItem('geminiApiKey', e.target.value.trim());
  }
});
```

### XSS Prevention

```javascript
// Use textContent instead of innerHTML for user input
element.textContent = userInput; // Safe

// Markdown is sanitized by marked.js (default settings)
els.activeContent.innerHTML = marked.parse(generatedContent); // Safe (marked escapes HTML)
```

### CORS & CSP

```html
<!-- No external resources except CDNs -->
<script src="https://cdn.jsdelivr.net/npm/marked/marked.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>

<!-- All API calls to Gemini are same-origin from browser perspective -->
<!-- No server-side proxy needed -->
```

### Data Privacy

- ✅ No analytics/tracking scripts
- ✅ No third-party cookies
- ✅ No telemetry to developer
- ✅ All data stays in user's browser (except API calls to Google)
- ✅ User can clear localStorage to remove all data

---

## Browser Compatibility

### Supported Browsers

| Browser | Minimum Version | Notes |
|---------|----------------|-------|
| Chrome | 90+ | Full support |
| Firefox | 88+ | Full support |
| Safari | 14+ | Full support |
| Edge | 90+ | Full support |
| Opera | 76+ | Full support |

### Required Features

- ✅ ES6+ JavaScript (const, let, arrow functions, async/await)
- ✅ CSS Grid & Flexbox
- ✅ CSS Custom Properties (variables)
- ✅ Backdrop filter (glassmorphism)
- ✅ FileReader API (image upload)
- ✅ localStorage API
- ✅ Fetch API

### Polyfills Not Needed

Modern browsers (2021+) natively support all features.

### Mobile Support

**Responsive Design:**
```css
@media (max-width: 768px) {
  h1 { font-size: 2.5rem; }
  .glass-panel { padding: 24px; }
  .flex-row { flex-direction: column; }
  .btn-secondary { width: 100%; }
}
```

**Touch Optimization:**
- Buttons are 44px+ touch targets
- Drag-drop fallback to click
- Modals are fullscreen on mobile

---

## Troubleshooting

### Common Issues & Solutions

#### Issue: "API Error" or Generation Fails

**Diagnosis:**
```javascript
// Check browser console for detailed error
console.error(error.message);
```

**Solutions:**
1. Verify API key is correct (starts with "AIza")
2. Check Gemini API quota: [console.cloud.google.com](https://console.cloud.google.com)
3. Try different model (falls back automatically)
4. Check internet connection

---

#### Issue: localStorage Quota Exceeded

**Diagnosis:**
```javascript
try {
  localStorage.setItem('test', 'value');
} catch (e) {
  if (e.name === 'QuotaExceededError') {
    console.error('localStorage full');
  }
}
```

**Solutions:**
1. Clear old personas: `localStorage.removeItem('customPersona')`
2. Export to PDF before clearing
3. Use Incognito mode (fresh storage)

---

#### Issue: Image Won't Upload

**Diagnosis:**
```javascript
console.log('File type:', file.type);
console.log('File size:', file.size);
```

**Solutions:**
1. Check file type is image/jpeg, image/png, or image/webp
2. Ensure file size < 4MB (4194304 bytes)
3. Try converting to JPG
4. Resize image to 1920px width

---

#### Issue: Persona Shows Wrong Company Name

**Solutions:**
1. Click "Refine" button
2. Explicitly answer: "What is your exact company name?"
3. Be specific: "NetworkSCR" not "Network company"
4. If still wrong, click "Reset" and try again

---

## API Reference

### JavaScript Functions

#### Core Functions

```javascript
/**
 * Call Gemini API with auto model selection
 * @param {string} apiKey - Gemini API key
 * @param {string} prompt - Text prompt
 * @param {string|null} imageBase64 - Base64 encoded image (optional)
 * @returns {Promise<string>} Generated text
 */
async function callGeminiAPI(apiKey, prompt, imageBase64 = null)

/**
 * Execute generation with specific model
 * @param {string} apiKey - API key
 * @param {string} modelName - Model ID (e.g., 'gemini-2.0-flash')
 * @param {string} prompt - Text prompt
 * @param {string|null} imageBase64 - Image data
 * @returns {Promise<string>} Generated text
 */
async function executeGeneration(apiKey, modelName, prompt, imageBase64)

/**
 * Start persona research workflow
 * @param {boolean} refining - Is this a refinement?
 */
async function startPersonaResearch(refining = false)

/**
 * Generate final persona from interview answers
 */
async function finalizePersonaGeneration()

/**
 * Analyze uploaded image and generate all content types
 */
async function analyzeImage()
```

#### UI Functions

```javascript
/**
 * Display persona with markdown rendering
 * @param {string} text - Markdown text
 */
function displayPersona(text)

/**
 * Show loading overlay with message
 * @param {boolean} isLoading - Show or hide
 * @param {string} text - Loading message
 */
function setLoading(isLoading, text = '')

/**
 * Show toast notification
 * @param {string} msg - Message text
 */
function showToast(msg)

/**
 * Copy current content to clipboard
 */
function copyCurrentContent()

/**
 * Export current content to PDF
 */
function exportCurrentToPDF()

/**
 * Reset image upload state
 */
function resetImage()
```

### Event Handlers

```javascript
// File upload
els.fileInput.addEventListener('change', (e) => handleFile(e.target.files[0]))
els.uploadArea.addEventListener('drop', handleDrop)

// Persona generation
els.generatePersonaBtn.addEventListener('click', () => startPersonaResearch(false))
els.submitInterviewBtn.addEventListener('click', finalizePersonaGeneration)

// Image analysis
els.analyzeBtn.addEventListener('click', analyzeImage)
els.resetImageBtn.addEventListener('click', resetImage)

// Tab switching
els.tabs.addEventListener('click', handleTabClick)

// Collapse panel
els.configCollapseBtn.addEventListener('click', () => {
  els.configPanel.classList.toggle('collapsed')
})
```

---

## Performance Optimization

### Bundle Size Optimization

Current size: ~50KB (uncompressed)
- HTML: ~5KB
- CSS: ~10KB
- JavaScript: ~35KB

**Could be reduced with:**
- Minification: ~30% reduction
- Code splitting: Load marked.js/jsPDF on demand
- Tree-shaking: Remove unused library features

### API Call Optimization

**Current:**
- Sequential generation (7 API calls)
- 1.2s delay between calls
- Total time: ~70 seconds

**Could be improved with:**
- Parallel generation (but risks rate limits)
- Batch API (if Gemini supports in future)
- Caching common persona patterns

### Image Optimization

**Client-side resize:**
```javascript
function resizeImage(file, maxWidth = 1920) {
  return new Promise((resolve) => {
    const img = new Image();
    img.onload = () => {
      const canvas = document.createElement('canvas');
      const ratio = maxWidth / img.width;
      canvas.width = maxWidth;
      canvas.height = img.height * ratio;
      const ctx = canvas.getContext('2d');
      ctx.drawImage(img, 0, 0, canvas.width, canvas.height);
      resolve(canvas.toDataURL('image/jpeg', 0.8));
    };
    img.src = URL.createObjectURL(file);
  });
}
```

---

## Deployment

### Hosting Options

**Option 1: GitHub Pages (Recommended)**
```bash
# 1. Create repo on GitHub
# 2. Upload index.html
# 3. Settings → Pages → Enable
# 4. Access at: https://username.github.io/repo-name
```

**Option 2: Netlify Drop**
```bash
# 1. Visit drop.netlify.com
# 2. Drag index.html into browser
# 3. Get instant URL
```

**Option 3: Self-Hosted**
```bash
# Any web server (Apache, Nginx, etc.)
# Just serve index.html as a static file
```

### CDN Configuration

**For production, consider:**
- Hosting marked.js and jsPDF locally (remove CDN dependency)
- Implementing Service Worker for offline support
- Adding versioning to cache-bust updates

---

## License & Credits

**Copyright:** © 2026 Vivek Pandit

**Technologies:**
- Google Gemini AI
- marked.js (markdown parser)
- jsPDF (PDF generation)

**Frameworks Referenced:**
- Robert McKee (Story)
- David Ogilvy (Advertising)
- Gary Vaynerchuk (Social Media)
- Neil Patel & Brian Dean (SEO)
- Perry Marshall (Google Ads)
- Joe Rogan, Ira Glass, Malcolm Gladwell (Podcasting)

---

**For additional support, refer to:**
- [README.md](README.md) - Project overview
- [quickstart.md](quickstart.md) - Getting started
- [workflows.md](workflows.md) - Process documentation
- [persona_feature.md](persona_feature.md) - Persona deep dive
