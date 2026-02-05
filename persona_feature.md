# Persona Feature Deep Dive

The persona generation system is the foundation of AI Scene Detective's content quality. This document explains the architecture, prompts, and best practices.

---

## Table of Contents

1. [Why Personas Matter](#why-personas-matter)
2. [Architecture Overview](#architecture-overview)
3. [The Accuracy Lock System](#the-accuracy-lock-system)
4. [Interview Question Generation](#interview-question-generation)
5. [Persona Synthesis Logic](#persona-synthesis-logic)
6. [Refinement System](#refinement-system)
7. [Storage & Persistence](#storage--persistence)
8. [Best Practices](#best-practices)

---

## Why Personas Matter

### The Problem with Generic AI Content

Without a persona:
- ❌ Generic, forgettable copy
- ❌ Wrong tone for audience
- ❌ Misaligned value propositions
- ❌ No brand consistency

With a persona:
- ✅ On-brand, distinctive content
- ✅ Audience-specific messaging
- ✅ Consistent voice across all 7 content types
- ✅ Competitive differentiation

### Real Example: NetworkSCR

**Without Persona:**
> "We provide IT services to help businesses with their technology needs."

**With Persona:**
> "NetworkSCR delivers 24/7 managed IT services to Chicago-area MSPs and businesses, eliminating downtime with our ConnectWise-integrated support desk and HIPAA-compliant infrastructure."

**Why it's better:**
- Exact company name (not "Network Security" or "Network Services")
- Specific service offering (managed IT, not generic "IT services")
- Geographic target (Chicago area)
- Technical credibility (ConnectWise, HIPAA)

---

## Architecture Overview

### Three-Phase System

```
Phase 1: RESEARCH        Phase 2: INTERVIEW       Phase 3: SYNTHESIS
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│ Website URL     │────→│ Generated       │────→│ Final Persona   │
│ (User Input)    │     │ Questions       │     │ (10 Sections)   │
│                 │     │ (AI-created)    │     │                 │
│ Client Examples │     │ User Answers    │     │ Saved to        │
│ (Optional)      │     │ (User Input)    │     │ localStorage    │
└─────────────────┘     └─────────────────┘     └─────────────────┘
```

### Data Flow

1. **User Input → Research Prompt**
   ```javascript
   const researchPrompt = `
   Analyze: ${websiteUrl}
   Context: ${clientWebsites}
   Goal: Generate 3-5 interview questions
   Constraint: If company name unclear, MUST ask
   `;
   ```

2. **AI Response → Question Array**
   ```json
   ["What is your exact company name?", "Who is your ideal customer?", ...]
   ```

3. **User Answers → Synthesis Prompt**
   ```javascript
   const synthesisPrompt = `
   Website: ${websiteUrl}
   Interview: ${Q1}: ${A1}, ${Q2}: ${A2}...
   Goal: Create 1000+ word persona
   Priority: User answers > Web assumptions
   `;
   ```

4. **AI Response → Formatted Persona**
   ```markdown
   ## 1. COMPANY PROFILE
   - Company Name: NetworkSCR
   - Tagline: 24/7 IT Security & Support
   ...
   ```

---

## The Accuracy Lock System

### Problem Statement
AI models can "hallucinate" company details when the website URL is ambiguous:
- "greatescapepools.com" → AI assumes "Great Escape Pools"
- "networkscr.com" → AI might guess "Network Security & Recovery"

### Solution: Forced Clarification

**In Research Prompt:**
```javascript
CRITICAL INSTRUCTIONS:
1. IDENTIFY THE EXACT COMPANY NAME. Do not guess.
2. If the name is not 100% clear from the URL, YOU MUST ASK THE USER.
3. If products/services are ambiguous, YOU MUST ASK.

If you are unsure of the Company Name, Question #1 MUST be:
"What is the exact official name of your company?"
```

### Detection Logic

The AI checks for:
- **Domain ambiguity:** Does URL clearly state company name?
  - ✅ Clear: `www.tesla.com` → Tesla
  - ❌ Unclear: `www.networkscr.com` → Network...?

- **Product ambiguity:** Are offerings obvious?
  - ✅ Clear: `www.teslamotors.com` → Electric vehicles
  - ❌ Unclear: `www.greatescapes.com` → Escape rooms? Pools? Travel?

### Example Output

**Ambiguous Case (NetworkSCR):**
```json
[
  "What is the exact official name of your company?",
  "What are your primary products or services?",
  "Who is your ideal customer?",
  "What is your main competitive advantage?"
]
```

**Clear Case (Tesla):**
```json
[
  "Who is your ideal customer segment (Model S/3/X/Y buyers)?",
  "How do you position against traditional luxury brands?",
  "What's your Supercharger network's role in your value prop?"
]
```

---

## Interview Question Generation

### AI Strategy

The research phase analyzes:
1. **Homepage content:** Hero section, headlines, CTAs
2. **About page:** Company history, mission, team
3. **Services/Products page:** Offerings, pricing signals
4. **Target audience signals:** Case studies, testimonials, industries served
5. **Brand voice:** Tone, language style, formality

### Question Categories

#### 1. Clarification Questions (Mandatory if ambiguous)
```
"What is the exact official name of your company?"
"What are your primary products or services?"
```

#### 2. Audience Questions
```
"Who is your ideal customer? (demographics, firmographics, psychographics)"
"What are the biggest pain points your customers face?"
"What objections do prospects typically have before buying?"
```

#### 3. Positioning Questions
```
"What is your main competitive advantage?"
"How do you differ from [detected competitor]?"
"What results/outcomes do clients typically achieve?"
```

#### 4. Brand Voice Questions
```
"What tone best represents your brand? (professional, casual, authoritative, friendly)"
"Are there any words or phrases you avoid?"
"How do you want customers to feel after interacting with your brand?"
```

#### 5. Context Questions
```
"What percentage of your business comes from [detected channel]?"
"What's your typical sales cycle length?"
"Do you serve any specific industries or niches?"
```

### Dynamic Question Selection

The AI selects 3-5 questions based on:
- **Certainty level:** More ambiguity = more clarification questions
- **Website depth:** Shallow site = broader questions, deep site = tactical questions
- **Industry:** B2B = firmographics, B2C = demographics
- **Refining mode:** If refining, ask deeper/different questions

---

## Persona Synthesis Logic

### Input Sources (Priority Order)

1. **User Interview Answers** (Highest Priority)
   - Explicit, verified information
   - Used verbatim for company name, products

2. **Website Content Analysis**
   - Extracted facts and claims
   - Used to fill gaps in interview answers

3. **Industry Best Practices**
   - Standard positioning for industry
   - Used when both above are insufficient

### Synthesis Prompt Structure

```javascript
const fullPrompt = `
You are a world-class marketing strategist.

WEBSITE: ${website}
INTERVIEW TRANSCRIPT:
Q: What is your exact company name?
A: NetworkSCR

Q: What are your primary products?
A: Managed IT services, 24/7 helpdesk, cybersecurity, cloud management

Q: Who is your ideal customer?
A: Small-medium businesses in Chicago, 20-200 employees, limited in-house IT

TASK: Create a COMPREHENSIVE BUYER PERSONA (1000+ words).

CRITICAL RULES:
1. ACCURACY IS PARAMOUNT. Use the exact Company Name from interview answers.
2. Prioritize user's answers over web assumptions.
3. Do not hallucinate details not provided.

OUTPUT STRUCTURE:
## 1. COMPANY PROFILE
- Exact Company Name: [From interview answer]
- Tagline: [Create from products/positioning]
- Industry & Niche: [From context]

## 2. BRAND VOICE & PERSONALITY
[How the company should sound]

## 3. TARGET AUDIENCE
[Detailed customer avatar]

## 4. UNIQUE VALUE PROPOSITION
[Why customers choose this company]

## 5. CONTENT STRATEGY
[What topics to cover, tone to use]

## 6. COMPETITIVE LANDSCAPE
[Who they compete with, how they differ]

## 7. MARKETING CHANNELS
[Where to reach audience]

## 8. CUSTOMER JOURNEY
[Awareness → Consideration → Decision stages]

## 9. OBJECTIONS & SOLUTIONS
[Common concerns and how to address]

## 10. BRAND GUIDELINES
[Dos and don'ts for content creation]

Use markdown formatting. Be specific. Use examples.
`;
```

### Quality Checks

Before saving, the system validates:
- ✅ Company name is present and not generic
- ✅ Persona is >800 words (comprehensive)
- ✅ All 10 sections are populated
- ✅ No placeholder text (e.g., "[Insert company name]")

---

## Refinement System

### When Users Click "Refine"

**Step 1: Analyze Current Persona**
```javascript
const refinePrompt = `
EXISTING PERSONA:
${customPersona}

TASK: Identify gaps or areas needing clarification.
Generate 3-5 DEEPER questions to improve this persona.

Focus on:
- Missing competitive details
- Vague target audience descriptions
- Unclear value propositions
- Inconsistent brand voice
`;
```

**Step 2: Generate Different Questions**
- Avoid repeating original questions
- Focus on nuance and edge cases
- Ask for quantitative data if possible

**Example Refined Questions:**
```
"What percentage of your clients are in the healthcare vs. legal sector?"
"What's your average client retention rate?"
"How does your pricing compare to [competitor name]?"
"What specific compliance certifications do you hold?"
```

**Step 3: Merge or Replace**

Current implementation: **Replace completely**
- Simpler logic
- Ensures consistency
- User can review before accepting

Future enhancement: **Smart merge**
- Keep sections that were accurate
- Update only sections with new information
- Track change history

---

## Storage & Persistence

### localStorage Schema

```javascript
{
  "geminiApiKey": "AIza...",           // API key
  "customPersona": "## 1. COMPANY...", // Full persona markdown
  "personaWebsite": "https://...",     // Original website
  "personaClients": "https://..."      // Client examples (optional)
}
```

### Persistence Behavior

- **On persona generation:** All 4 keys saved
- **On page load:** API key, website, clients auto-populated
- **On persona display:** Rendered from localStorage
- **On reset:** `customPersona` deleted (others remain for convenience)

### Cross-Session Usage

**Session 1 (Tuesday):**
```
1. Generate persona for Client A
2. Create content from 3 images
3. Close browser
```

**Session 2 (Wednesday):**
```
4. Open browser → Persona still loaded
5. Upload 5 more images → Same persona applied
6. Export all content
```

### Multi-Device Sync

**Not currently supported** (localStorage is browser-local)

**Workaround:**
1. Export persona to PDF
2. Copy markdown text to notes app (synced)
3. On new device, paste back into persona section

**Future enhancement:** Cloud sync with user account

---

## Best Practices

### For Initial Persona Generation

✅ **DO:**
- Use your main website (most comprehensive)
- Answer ALL interview questions thoroughly
- Be specific with numbers ("20-50 employees" not "small businesses")
- Mention actual competitor names
- Include industry jargon/terminology

❌ **DON'T:**
- Use social media profiles (limited info)
- Skip questions that seem obvious
- Use generic descriptions
- Assume AI will "figure it out"

### For Refinement

✅ **DO:**
- Refine after generating 1-2 pieces of content
- Review what's inaccurate or missing
- Add data-driven details (percentages, timelines)

❌ **DON'T:**
- Refine immediately (see how it performs first)
- Answer with contradictory information
- Over-refine (3+ times rarely improves quality)

### For Multi-Client Usage

✅ **DO:**
- Use browser tabs (one per client)
- Export persona to PDF for client records
- Keep consistent persona across campaign

❌ **DON'T:**
- Mix client personas in same session
- Forget to click "Reset" between clients
- Reuse persona for fundamentally different brands

---

## Troubleshooting Persona Issues

### Issue: Company Name is Wrong

**Diagnosis:**
- AI couldn't find it on website
- Interview question was skipped
- User didn't provide exact name

**Fix:**
1. Click "Refine"
2. Explicitly answer "What is your exact company name?"
3. Review new persona
4. If still wrong, click "Reset" and start over

---

### Issue: Target Audience Too Broad

**Example:** "Businesses that need IT services"

**Fix:**
1. Click "Refine"
2. When asked about ideal customer, provide:
   - Industry (e.g., "Healthcare and legal")
   - Size (e.g., "10-50 employees")
   - Geography (e.g., "Chicago metro area")
   - Pain points (e.g., "Compliance concerns, limited budget")

---

### Issue: Brand Voice Doesn't Match

**Example:** Persona sounds too formal for a casual brand

**Fix:**
1. Click "Refine"
2. Add specific voice guidance:
   - "We use casual, conversational language"
   - "We avoid corporate jargon"
   - "Think Gary Vaynerchuk, not McKinsey"

---

### Issue: Persona is Too Generic

**Diagnosis:** Not enough specific details in answers

**Fix:** Answer interview questions with:
- Exact numbers ("$5K-$15K deal size")
- Named competitors ("We compete with IBM and Accenture")
- Specific use cases ("law firms preparing for audits")
- Unique differentiators ("only MSP with 24/7 human support")

---

## Advanced Techniques

### Technique 1: Industry-Specific Personas

For specialized industries (healthcare, legal, finance):
1. Mention compliance requirements (HIPAA, SOC 2, GDPR)
2. Use industry acronyms (MSP, EHR, CRM)
3. Reference industry publications/events
4. Mention certifications/partnerships

**Result:** Content feels native to that industry

---

### Technique 2: Persona Versioning

For A/B testing different positioning:
1. Generate Persona A (emphasizing speed)
2. Export to PDF as "Persona_Speed_v1.pdf"
3. Click Reset
4. Generate Persona B (emphasizing quality)
5. Export to PDF as "Persona_Quality_v1.pdf"
6. Test content from both personas
7. Keep winner for ongoing use

---

### Technique 3: Segment-Specific Personas

For companies serving multiple segments:
1. Generate "Enterprise Persona" (large clients)
2. Use for big-ticket content
3. Generate "SMB Persona" (small clients)
4. Use for volume content
5. Keep both in separate browser sessions/profiles

---

## Technical Implementation Notes

### Model Selection for Persona

Prefers: **Gemini 2.0 Flash** (fastest, most recent training)
Fallback: **Gemini 1.5 Pro** (more comprehensive reasoning)

### Token Budget

- Research prompt: ~300 tokens
- Interview questions: ~50-200 tokens output
- Synthesis prompt: ~500-1000 tokens
- Persona output: ~2000-4000 tokens

**Total per persona:** ~3000-5000 tokens (~$0.01-0.02 on paid tier)

### Rate Limiting

Persona generation involves:
- 1 research call
- 1 synthesis call

**Total API calls:** 2
**Time required:** ~60-90 seconds

---

## Future Enhancements

### Planned Features

1. **Persona Templates**
   - Pre-built personas for common industries
   - Quick-start for standard business types

2. **Persona Library**
   - Save multiple personas locally
   - Switch between them without reset

3. **Smart Merge on Refinement**
   - Update only changed sections
   - Track version history

4. **Persona Import/Export**
   - JSON format for sharing
   - Restore from previous export

5. **Visual Persona Builder**
   - Drag-and-drop section editor
   - Live preview as you build

---

**Next:** [Full Technical Documentation](documentation.md)
