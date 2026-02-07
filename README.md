# Synthia 🎨

**AI marketing intelligence that synthesizes legendary frameworks with your unique brand voice.**

Synthia is a single‑page, browser‑only application that combines computer vision, brand strategy, and proven marketing frameworks to turn any image into a complete, executable content ecosystem.

---

## 🌟 What Synthia Does

Synthia helps you go from **brand** → **image** → **strategy** → **content** → **execution** in one flow:

1. Understands your brand and audience (Persona Engine).
2. Analyzes an uploaded image using AI (Vision + Text).
3. Generates a 30‑day marketing calendar around that image.
4. Produces long‑form and channel‑specific content using expert frameworks.
5. Provides a live **Campaign Command Center** with checklist + Kanban board to manage execution.
6. Exports everything to PDFs with your image embedded.

All of this runs locally in your browser with your own Gemini API key.

---

## 🎯 Core Features

### 1. Brand Persona Engine

- **Website Research:** Paste your website URL; Synthia uses Gemini to infer your positioning, audience, and offers.
- **Clarifying Interview:**
  - Asks whether you’re marketing a **Product** or a **Service**.
  - Asks whether you’re doing this for **your own brand** or a **client** (agency mode).
  - Asks follow‑ups to nail voice, offer, and audience.
- **Strategic Frameworks:**
  - Products lean on the **4 Ps** (Product, Price, Place, Promotion).
  - Services lean on the **7 Ps** (People, Process, Physical Evidence, etc.).
- **Persistent Persona:** The generated persona is stored in your browser and reused until you reset it.

### 2. Image‑Driven Content Engine

- **Image Upload:** Drag‑and‑drop or browse any image file (JPEG, PNG, etc., up to ~4 MB).
- **Vision Context:** The image is encoded to base64 and sent to Gemini as inline data, so content is grounded in what’s actually visible.
- **Visual Integration:**
  - The image appears in the UI above each content tab (except persona/tracker).
  - The same image is embedded in exported PDFs.

### 3. Multi‑Framework Content Generation

From one image + persona, Synthia generates multiple content artifacts:

1. **Story**  
   - 3‑act story structure (Robert McKee) and Hero’s Journey style narrative.
   - Tuned for founders’ stories or brand origin stories.

2. **Marketing Copy**  
   - Headline formulas and body copy using PAS (Problem–Agitate–Solution) and classic Ogilvy/Halbert patterns.
   - CTAs adapt:
     - Services → “Book a Call”, “Schedule Consultation”.
     - Products → “Buy Now”, “Shop the Look”.

3. **Instagram / Social Captions**  
   - “Jab, Jab, Jab, Right Hook” structure inspired by Gary Vee.
   - Mix of story, value, question, behind‑the‑scenes, and direct offer posts.

4. **SEO Blog Post**  
   - Long‑form SEO‑oriented post (Skyscraper‑style) with headings, sections, and clear CTAs.

5. **Google Ads Concepts**  
   - High‑intent search ad ideas (headlines + descriptions).
   - Service vs product emphasis (expertise vs price/features).

6. **Podcast Script / Audio Outline**  
   - Episode concept and talking points, with hooks and CTAs.
   - Can be recorded verbatim or used as notes.

7. **AEO (Answer Engine Optimization) Audit**  
   - Recommendations to make your site more “answerable” by AI agents.
   - Focus on structure, schema, FAQs, topical authority.

All prompts inject the active persona and respect product vs service logic.

### 4. Marketing Calendar (30‑Day Plan)

- Generates a **4‑week plan** with:
  - Day, channel, content type, topic/hook, and goal.
  - Weekly themes: Awareness → Consideration → Trust → Conversion.
- Service businesses emphasize:
  - Weekday LinkedIn, webinars, case studies, and outreach.
- Product businesses emphasize:
  - Weekends/evenings, Reels/TikToks, promos, and display ads.
- Output: Markdown table plus a narrative explanation.

### 5. Campaign Command Center (Tracker + Kanban)

This is where Synthia becomes an execution tool, not just a content generator.

#### List View (Checklist)

- Each row from the calendar becomes a **task**:
  - Day, channel, type, topic, goal.
- For each task you can:
  - Tick a checkbox when done.
  - Add free‑form notes (“Re‑shot reel”, “Client requested edits”, etc.).
- Dashboard at top:
  - Completion percentage.
  - Tasks done vs total.
  - Remaining count.

#### Kanban Board View (Drag & Drop)

- Switch between **List** and **Kanban** with a simple toggle.
- Three status columns:
  - **To Do**
  - **In Progress**
  - **Done**
- Drag & drop cards between columns using the native HTML5 drag‑and‑drop API.
- Status and completion state stay in sync with the checklist view.

#### Persistence & Reset

- **Auto‑Save:** Tasks, status (“todo/doing/done”), and notes are saved to `localStorage` as you go.
- **Resume Later:** Close the browser and come back to see your board exactly as you left it.
- **Reset:** One‑click reset of tracker state while keeping persona/API key intact.

### 6. PDF Export (With Image)

- Any tab’s content can be exported as a PDF:
  - Header with title and date.
  - Embedded image (for content tabs that use the image).
  - Auto‑paginated text.
- Tracker export:
  - Exports a readable status report (TODO / IN PROGRESS / DONE with notes).

---

## 🧱 Architecture Overview

- **Type:** Single‑Page Application (SPA).
- **Stack:** HTML5 + CSS3 + vanilla JavaScript (ES6+).
- **No Build Step:** Single `index.html` file plus assets.
- **No Backend:** All logic runs client‑side in the user’s browser.
- **AI Engine:** Google Gemini (2.0 Flash / 1.5, auto‑discovery of best model).
- **Libraries:**
  - `marked.js` – render Markdown to HTML.
  - `jsPDF` – generate and download PDFs.
  - Google Fonts (Inter) – for typography.

---

## 🔑 Setup & Usage

### Requirements

- Modern browser (Chrome, Edge, Firefox, Safari).
- Gemini API key from Google AI Studio.
- A website URL for persona generation.
- One or more marketing images.

### Installation

1. Place the files together:
   - `index.html`
   - `synthia_icon_transparent.png`
2. Open `index.html` in your browser (double‑click or `open`/`start`/`xdg-open` from a terminal).

### Flow

1. **Configure Brand**
   - Paste your Gemini API key.
   - Enter your website URL and optional client URLs.
   - Generate persona and answer the clarifying questions.

2. **Upload Image & Generate**
   - Drag‑and‑drop an image.
   - Click “Analyze & Generate Content”.
   - Wait while Synthia:
     - Builds your 30‑day calendar.
     - Generates all content types.
     - Populates the tracker and Kanban board.

3. **Manage Campaign**
   - Use the **Tracker** tab:
     - Check off tasks or drag cards across Kanban columns.
     - Add notes as you execute.
   - Export any view to PDF if needed.

---

## 🔐 Privacy & Security

- Runs entirely on the client; no Synthia‑side backend.
- Only outbound network calls go to the official Gemini API endpoint.
- API key and persona are stored in `localStorage` on your device.
- No analytics, cookies, or third‑party trackers are included.

---

## 🛠️ Technical Summary

- **UI Design:** Glassmorphism, responsive layout, mobile‑friendly Kanban.
- **State:**
  - In‑memory JS variables for active session.
  - `localStorage` for persona, website, and tracker tasks.
- **Networking:** `fetch` with JSON bodies to Gemini `generateContent`.
- **Drag & Drop:** Native HTML Drag and Drop API for Kanban cards.
- **Markdown Rendering:** `marked.parse(...)` into the content area.
- **PDF:** `jsPDF` with `addImage` and `splitTextToSize`.

---

## 🧾 Version History (High‑Level)

- **v3.4:** Kanban board for campaign tasks; dual view (List + Board); drag‑and‑drop and migration of tracker data.
- **v3.3:** Campaign tracker (checklist with progress and notes) and persistent tracker state.
- **v3.2:** Marketing calendar generation with product vs service strategy logic and client vs owner mode.
- **v3.1:** Image display in content tabs and image embedding in PDFs.
- **v3.0 and earlier:** Core persona engine, multi‑framework content types, AEO audit, PDF export, responsive UI.

---

**Synthia: From image to executed campaign, in one glassy interface.**
