# Synthia 🎨

**AI Marketing Intelligence that synthesizes legendary frameworks with your unique brand voice.**

Synthia is a sophisticated single-page web application that combines computer vision, brand strategy, and expert marketing frameworks to generate comprehensive content from images. Built with pure HTML/CSS/JavaScript and powered by Google's Gemini AI.

---

## 🌟 Key Features

### 🎯 Intelligent Brand Persona Generation
- **Automated Research:** Synthia analyzes your website to understand your business
- **Interactive Interview:** Targeted questions ensure accuracy (no more generic personas)
- **Accuracy Lock:** Forces clarification for ambiguous company names or products
- **Persistent Storage:** Personas saved locally for reuse across sessions

### 📸 Image-to-Content Pipeline
- **8 Expert Frameworks:** Each piece uses proven copywriting/storytelling methods
- **Persona-Driven:** All content reflects your brand voice and positioning
- **One-Click Generation:** Upload → Analyze → Get all 8 content types
- **🆕 Visual Integration:** Your uploaded image is displayed with each content piece and embedded in PDF exports

### 🎨 Content Types Generated

1. **Story** (800-1,200 words) - Robert McKee's 3-act structure + Hero's Journey
2. **Marketing Copy** (600-800 words) - David Ogilvy + Gary Halbert frameworks
3. **Instagram Captions** (5 variations) - Gary Vaynerchuk's engagement tactics
4. **Blog Post** (1,800+ words) - Neil Patel + Brian Dean SEO methods
5. **Google Ads Campaign** - Perry Marshall's 80/20 principle (15 headlines + descriptions)
6. **Podcast Script** (2,000 words) - Joe Rogan + Ira Glass storytelling
7. **Persona Summary** - The brand persona used for generation
8. **🆕 AEO Website Audit** - Answer Engine Optimization recommendations

### ✨ v3.1: Image Display & PDF Integration

**What's New:**
- **On-Screen Display:** The image you uploaded now appears at the top of each content tab (except Persona), providing visual context
- **PDF Embedding:** When you export content to PDF, the image is automatically included at the top of the document
- **Smart Sizing:** Images are intelligently scaled to fit the page width while maintaining aspect ratio
- **Professional Layout:** Beautiful glass-effect border and shadow treatment matches the Synthia design language

**Technical Details:**
- Uses jsPDF's `addImage()` method with base64 encoding [web:150]
- Supports JPEG, PNG, and GIF formats automatically
- Calculates optimal dimensions based on aspect ratio
- Respects page boundaries with auto-pagination logic

### ✨ v3.0: Advanced Prompt Engineering

We have completely overhauled the AI prompting engine to ensure deep persona integration:

- **Context Injection:** Every piece of content now explicitly references your brand voice and specific audience pain points.
- **Visual Proof:** Marketing copy now uses the uploaded image as "visual evidence" of your value proposition.
- **How-To Instructions:** Each content result now includes a specific "How to Use" section telling you exactly where to publish or how to implement the generated text.
- **Full PDF Export:** Improved PDF engine now captures long-form content perfectly with auto-paging.

### ✨ Answer Engine Optimization (AEO)

**Get discovered by AI answer engines like ChatGPT, Perplexity, Google AI Overviews, and Gemini.**

Based on frameworks from Contently, HubSpot, Forrester, and leading AI search experts, Synthia generates a comprehensive 10-point AEO audit:

1. **Content Optimization** - Answer-centric structure analysis
2. **Schema Markup** - FAQPage, HowTo, Article implementation guides
3. **Technical Infrastructure** - Core Web Vitals audit (LCP, FID, CLS)
4. **Topical Authority** - E-E-A-T signal recommendations
5. **User Intent Mapping** - Question inventory creation
6. **Content Format** - AI-friendly formatting tactics
7. **Competitive Analysis** - Gap identification vs. competitors
8. **Measurement** - Track AI citation frequency and visibility
9. **Action Plan** - Prioritized roadmap (Quick wins → Long-term)
10. **Risk Mitigation** - Protect against inaccurate AI citations

**Output includes:**
- Current AEO maturity score (1-10)
- Actionable recommendations with code snippets
- Implementation roadmap with priorities
- Schema markup examples ready to use

### 🎨 Premium UI/UX
- **Liquid Glass Design:** Modern, glassmorphic interface with animated gradients
- **Sophisticated Color Palette:** Elegant violet/purple theme with pink accents
- **Markdown Rendering:** Beautiful formatting with headers, lists, and emphasis
- **Image Integration:** Professional display of uploaded images in content and PDFs
- **PDF Export:** One-click export with embedded images for any content piece
- **Reset Controls:** Easy persona and image management
- **Mobile Optimized:** Seamless experience on iPhone and desktop

---

## 🚀 Quick Start

### Prerequisites
- Modern web browser (Chrome, Firefox, Safari, Edge)
- Google Gemini API key ([Get one free](https://aistudio.google.com/app/apikey))
- Your business website URL

### Installation

1. **Download the files:**
   ```bash
   # Download these files to the same folder:
   # - index.html
   # - synthia_icon_transparent.png
   ```

2. **Open in browser:**
   ```bash
   # Double-click index.html or right-click → Open With → Browser
   ```

3. **Configure:**
   - Enter your Gemini API key (stored locally, never sent anywhere but Google)
   - Add your website URL
   - Click "Start Research & Generate Persona"

That's it! No installation, no dependencies, no server required.

---

## 📚 How It Works

### Phase 1: Persona Research
```
Your Website → AI Analysis → Intelligent Questions → Interview → Final Persona
```

Synthia first analyzes your website, then asks 3-5 targeted questions to clarify:
- Exact company name (critical accuracy feature)
- Specific products/services
- Target audience details
- Competitive positioning
- Brand voice preferences

### Phase 2: Content Generation
```
Upload Image → AI Vision Analysis → Apply Persona + Framework → 8 Content Pieces (with Image)
```

For each content type, Synthia:
1. Analyzes the image in detail (objects, people, setting, mood)
2. Applies your brand persona
3. Uses the corresponding expert framework
4. Generates publication-ready content
5. Appends "How to Use" instructions
6. **🆕 Displays the image alongside the content**

### Phase 3: PDF Export
```
Content + Image → jsPDF Processing → Professional PDF with Embedded Image
```

When you click "PDF":
- Image is added at the top (maintaining aspect ratio)
- Content follows with proper formatting
- Auto-pagination ensures nothing gets cut off
- Result: Professional, print-ready document

### Phase 4: AEO Audit
```
Website Context + Image Analysis → 10-Point AEO Framework → Actionable Recommendations
```

Synthia audits your website's readiness for AI answer engines and provides:
- Schema markup implementation guides
- Content structure optimizations
- Technical infrastructure improvements
- Prioritized action plan

---

## 🔐 Privacy & Security

- **100% Local Processing:** Runs entirely in your browser
- **No Server Required:** No data sent to third parties (except Google Gemini API)
- **Local Storage Only:** API keys and personas stored in browser localStorage
- **No Tracking:** No analytics, no cookies, no telemetry
- **Image Privacy:** Images processed client-side, only sent to Google Gemini for analysis

---

## 🛠️ Technical Stack

- **Frontend:** Pure HTML5, CSS3 (Glassmorphism), Vanilla JavaScript
- **AI Engine:** Google Gemini 2.0 Flash / 1.5 Pro (auto-detection)
- **Markdown:** marked.js for rich text rendering
- **PDF Export:** jsPDF 2.5.1 with image embedding support
- **Image Processing:** FileReader API for base64 encoding
- **Architecture:** Single-page application, no build tools required
- **Fonts:** Inter via Google Fonts for premium typography

---

## 💡 Use Cases

### Marketing Agencies
- Generate client-specific content from photo assets
- **🆕 Export professional PDFs with images for client presentations**
- Maintain brand consistency across channels
- Accelerate campaign development
- Optimize client websites for AI discoverability

### Solopreneurs / SMBs
- Create professional marketing materials without a copywriter
- Repurpose photos into blogs, ads, and social content
- **🆕 Generate PDF reports with visual context for stakeholders**
- Build cohesive brand messaging
- Compete with larger companies in AI search results

### Content Creators
- Transform behind-the-scenes photos into engaging stories
- Generate multiple content angles from one image
- **🆕 Create professional PDF portfolios with images and copy**
- Maintain consistent voice across platforms
- Ensure content is AI-engine friendly

---

## 🎯 Best Practices

### For Optimal Personas
1. Use your main website (not social media profiles)
2. Answer interview questions thoroughly
3. If company name is missed, use the "Refine" button
4. Review generated persona before image analysis

### For Best Content
1. Use high-quality, well-lit images (under 4MB)
2. Choose images that tell a story
3. Images with people/action generate richer narratives
4. Product photos work great for ads and marketing copy
5. **🆕 Landscape images work best for PDF layouts (wider aspect ratios)**

### For AEO Success
1. Implement recommended schema markup first (highest ROI)
2. Focus on question-answer content format
3. Optimize Core Web Vitals for better AI crawling
4. Monitor AI citations monthly in ChatGPT and Perplexity

### For PDF Exports
1. **🆕 Images are automatically sized to fit page width**
2. **🆕 Content text auto-paginates to avoid cutoff**
3. **🆕 Use "Persona" tab PDF for brand guidelines without image**
4. **🆕 All other tabs include image at the top**

---

## 🐛 Troubleshooting

### "API Error" or "Failed to generate"
- Verify API key is correct
- Check you have Gemini API quota remaining
- Try refreshing the page

### "Company name is wrong"
- Click "Refine" to update persona
- Manually answer "What is your exact company name?"
- Use the "Reset" button to start fresh

### Content is too generic
- Generate a more detailed persona first
- Answer interview questions with specifics
- Use the "Refine" feature to add depth

### AEO recommendations seem off
- Ensure website URL is correct during persona generation
- Upload an image that represents your actual business
- Review persona accuracy first

### 🆕 Image not appearing in PDF
- Ensure image is under 4MB
- Supported formats: JPEG, PNG, GIF
- Try re-uploading the image
- Check browser console for errors

### 🆕 PDF text is cut off
- This should be fixed in v3.1 with auto-pagination
- If issue persists, try a different browser
- Report the issue with content length details

---

## 🎨 Design Philosophy

Synthia embodies **sophisticated AI marketing-as-a-service** through:
- **Premium aesthetics:** Liquid glass design with violet/purple gradients
- **Apple Human Interface Guidelines:** Clean, intuitive, accessibility-focused
- **Effortless power:** Complex AI workflows hidden behind elegant simplicity
- **Mobile-first:** Touch-friendly controls, responsive layouts
- **🆕 Visual Storytelling:** Images integrated seamlessly with generated content

---

## 📄 License

Copyright © 2026 Vivek Pandit. All rights reserved.

This software is provided for demonstration and educational purposes.

---

## 🙏 Credits

**Designed and Developed by Vivek Pandit using AI**

Frameworks inspired by:
- **Story:** Robert McKee (Story structure)
- **Marketing:** David Ogilvy, Gary Halbert, Eugene Schwartz (Advertising)
- **Social:** Gary Vaynerchuk (Social media)
- **Blog:** Neil Patel, Brian Dean (SEO/Content)
- **Ads:** Perry Marshall (Google Ads)
- **Podcast:** Joe Rogan, Ira Glass, Malcolm Gladwell (Audio storytelling)
- **AEO:** Contently, HubSpot, Forrester, AI search experts

**Technical References:**
- jsPDF image embedding: Stack Overflow community [web:150][web:151]
- Base64 encoding best practices [web:152][web:154]

---

## 📞 Support

For questions or issues, please refer to:
- **Quick Start Guide** (quickstart.md)
- **Full Documentation** (documentation.md)
- **AEO Test Report** (AEO_TEST_REPORT.md)
- **🆕 Image Integration Guide** (below)

---

## 🖼️ Image Integration Technical Guide

### How Images Are Processed

1. **Upload:** FileReader API converts image to base64
2. **Display:** Base64 data rendered as `<img>` element
3. **Analysis:** Base64 sent to Gemini AI for vision analysis
4. **Content Generation:** Image context informs all 8 frameworks
5. **PDF Export:** jsPDF's `addImage()` embeds base64 in document

### Supported Formats

| Format | Extension | PDF Support | Notes |
|--------|-----------|-------------|-------|
| JPEG   | .jpg, .jpeg | ✅ Excellent | Best compression, recommended |
| PNG    | .png      | ✅ Excellent | Transparency supported |
| GIF    | .gif      | ✅ Good      | Animated GIFs show first frame only |

### Image Best Practices

**For On-Screen Display:**
- Max file size: 4MB (enforced)
- Recommended dimensions: 1200-2400px width
- Aspect ratio: 16:9 or 4:3 works best

**For PDF Export:**
- Images auto-scale to page width (170mm)
- Height calculated to maintain aspect ratio
- Images placed before text content
- Pagination ensures image never splits across pages

### Code Example (for developers)

```javascript
// How Synthia handles image embedding in PDFs
const imgData = 'data:image/jpeg;base64,' + imageBase64;
const imgFormat = 'JPEG'; // Auto-detected from file type
const pageWidth = 210; // A4 width in mm
const margin = 20;
const imgWidth = pageWidth - (margin * 2);

// Calculate height maintaining aspect ratio
const aspectRatio = img.width / img.height;
const imgHeight = imgWidth / aspectRatio;

// Add to PDF
doc.addImage(imgData, imgFormat, margin, cursorY, imgWidth, imgHeight);
```

---

## 🆕 Version History

### v3.1 (February 2026) - Current
- 🖼️ **Image Display:** Uploaded images now appear at top of each content tab
- 📄 **PDF Integration:** Images automatically embedded in exported PDFs
- 📐 **Smart Sizing:** Aspect ratio preservation with intelligent scaling
- 🎨 **Visual Polish:** Glass-effect borders and professional image treatment

### v3.0 (February 2026)
- ✨ Advanced Prompt Engineering: Deep context injection for all content types
- 📖 "How to Use" instructions added to every generation
- 📄 PDF Export Fix: Now supports full-length content with pagination
- 🧠 AEO Audit V2: Specific role-based instructions (Dev/SEO/Content)

### v2.0 (February 2026)
- ✨ NEW: Answer Engine Optimization (AEO) audit feature
- 📱 Mobile responsive design improvements
- 🎨 Updated branding: "Synthia" with new logo
- 🔤 Premium typography with Inter font family
- ✅ All existing features preserved and enhanced

### v1.0 (January 2026)
- Initial release with 7 content types
- Brand persona generation with interview mode
- Image analysis with Gemini AI
- PDF export and markdown rendering

---

**Synthia: Where brand synthesis meets marketing intelligence. Made with ❤️ for marketers, creators, and entrepreneurs who deserve better tools.**
