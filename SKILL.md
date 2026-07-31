# CreateNotes Skill

## Trigger
`/CreateNotes`

## Purpose
Read any uploaded document (PDF, image, video frame) → extract all content → produce structured, aesthetic HTML study notes → offer PDF conversion.

---

## Package Installation (run once on first use)

```bash
pip install weasyprint pdfplumber pillow --break-system-packages -q
apt-get install -y poppler-utils ffmpeg 2>/dev/null || true
```

---

## Activation Sequence

When user types `/CreateNotes`:

**Step 1 — Collect inputs (single message, no back-and-forth)**
Ask ONE combined question:
> "Please provide: (1) your uploaded file(s), and (2) your UI idea for the notes layout — e.g. 'cards with purple highlights', 'timeline style', 'Cornell format', 'minimal dark', etc. Also paste your syllabus/topic list if you want selective coverage."

**Step 2 — Read files silently** (no narration)

**Step 3 — CODA Loop** (Context → Objective → Details → Action) until notes are complete

**Step 4 — Output HTML, then ask about PDF**

---

## CODA Loop (Internal — never narrate to user)

```
CONTEXT:   What is the document about? Subject, level, scope.
OBJECTIVE: What must the notes cover? (syllabus filter if provided, else everything)
DETAILS:   What UI style did the user specify? Map to CSS components below.
ACTION:    Generate the HTML. Review: any section thin/missing? If yes, loop back to DETAILS and fill. If complete, output.
```

**Loop condition:** Loop only if a major syllabus point has zero coverage in the draft. Max 2 loops. Never tell the user you are looping.

---

## File Reading Rules (token-efficient)

### PDF
```python
import pdfplumber, subprocess, base64

# Try text extraction first (cheap)
with pdfplumber.open(path) as pdf:
    text = "\n".join(p.extract_text() or "" for p in pdf.pages)

# If text < 100 chars, it's scanned — rasterize
if len(text.strip()) < 100:
    subprocess.run(["pdftoppm","-jpeg","-r","150", path, "/tmp/pg"])
    # then view each /tmp/pg-N.jpg with view() tool
```

### Image
Use `view()` tool directly. No conversion needed.

### Video
```bash
ffmpeg -i input.mp4 -vf "fps=1/10" /tmp/frame_%03d.jpg -q:v 3 2>/dev/null
# view key frames with view() tool
```

**Token rule:** Extract text first. Only rasterize if text extraction yields < 100 chars. View only as many frames/pages as needed to cover all syllabus points — skip duplicate/blank pages.

---

## HTML Output Rules

### Always
- White background (`#fff`) — NO ruled lines, NO beige body
- Self-contained single HTML file — all CSS inline in `<style>`
- Use system fonts: `Arial, Helvetica, sans-serif` — no Google Fonts import (PDF rendering)
- Max width `820px`, padding `48px 56px`

### Never
- `repeating-linear-gradient` ruled lines
- `@import url(...)` Google Fonts
- `localStorage` or JS storage
- External CSS/JS links

### Purple Highlight (standard)
```css
.hl {
  background: linear-gradient(180deg, transparent 52%, rgba(164,130,222,0.5) 52%);
  padding: 0 2px;
  font-weight: 700;
}
```

### Section Heading (standard)
```css
h2.section {
  font-size: 12px; font-weight: 900; letter-spacing: 0.1em;
  text-transform: uppercase; color: #111;
  margin-top: 30px; margin-bottom: 12px;
  display: flex; align-items: center; gap: 10px;
}
h2.section::after { content:''; flex:1; height:1.5px; background:#111; }
```

---

## UI Component Library

Map user's UI idea to these components. Mix as needed.

### CALLOUT BOX
```html
<div style="background:#f0ecfa;border-left:4px solid #7c5cbf;border-radius:0 8px 8px 0;padding:13px 17px;margin:16px 0;font-size:13.5px;">
  <strong style="color:#5a3fa0;">Label:</strong> content
</div>
```

### DEFINITION BOX
```html
<div style="border:2px solid #7c5cbf;border-radius:10px;padding:16px 20px;margin:14px 0;background:#fdfcff;">
  <div style="font-size:10px;font-weight:900;letter-spacing:.12em;text-transform:uppercase;color:#7c5cbf;margin-bottom:6px;">Definition</div>
  <p style="font-size:14px;color:#222;line-height:1.65;">content</p>
</div>
```

### CAUSES GRID (2-col numbered cards)
```html
<div style="display:grid;grid-template-columns:repeat(2,1fr);gap:9px;margin:12px 0;">
  <div style="display:flex;gap:10px;background:#fdfcff;border:1px solid #e0d4f5;border-radius:8px;padding:10px 12px;">
    <div style="width:24px;height:24px;min-width:24px;background:#7c5cbf;color:#fff;font-size:11px;font-weight:900;border-radius:50%;display:flex;align-items:center;justify-content:center;">1</div>
    <div style="font-size:12.5px;color:#222;line-height:1.5;"><strong>Title</strong> Detail text</div>
  </div>
</div>
```

### PERSON CARD
```html
<div style="border:1.5px solid #c9b8e8;border-radius:10px;padding:14px 18px;margin:10px 0;background:#fdfcff;">
  <div style="font-size:14px;font-weight:900;color:#5a3fa0;margin-bottom:3px;">Name</div>
  <div style="font-size:10px;font-weight:700;letter-spacing:.1em;text-transform:uppercase;color:#aaa;margin-bottom:7px;">Field · Era</div>
  <p style="font-size:13px;color:#333;line-height:1.6;">content</p>
</div>
```

### COMPARE TABLE (2-col)
```html
<div style="border:1.5px solid #7c5cbf;border-radius:10px;overflow:hidden;margin:14px 0;font-size:13px;">
  <div style="display:grid;grid-template-columns:1fr 1fr;">
    <div style="background:#7c5cbf;color:#fff;font-weight:800;font-size:12px;letter-spacing:.07em;text-transform:uppercase;padding:9px 14px;text-align:center;">A</div>
    <div style="background:#7c5cbf;color:#fff;font-weight:800;font-size:12px;letter-spacing:.07em;text-transform:uppercase;padding:9px 14px;text-align:center;">B</div>
  </div>
  <div style="display:grid;grid-template-columns:1fr 1fr;border-top:.5px solid #e0d4f5;">
    <div style="padding:7px 13px;background:#faf8ff;">Cell A</div>
    <div style="padding:7px 13px;background:#fff;">Cell B</div>
  </div>
</div>
```

### IMPACT GRID (3-col icon cards)
```html
<div style="display:grid;grid-template-columns:repeat(3,1fr);gap:9px;margin:12px 0;">
  <div style="background:#f0ecfa;border-radius:8px;padding:11px 12px;text-align:center;">
    <div style="font-size:18px;margin-bottom:5px;">🔬</div>
    <div style="font-size:11px;font-weight:800;color:#5a3fa0;text-transform:uppercase;letter-spacing:.05em;">Title</div>
    <div style="font-size:11px;color:#555;margin-top:4px;line-height:1.45;">Desc</div>
  </div>
</div>
```

### TIMELINE STRIP
```html
<div style="display:flex;gap:0;margin:14px 0;position:relative;">
  <div style="position:absolute;top:20px;left:24px;right:24px;height:2px;background:#7c5cbf;z-index:0;"></div>
  <div style="flex:1;text-align:center;position:relative;z-index:1;">
    <div style="width:14px;height:14px;background:#7c5cbf;border-radius:50%;margin:14px auto 6px;border:2.5px solid #fff;box-shadow:0 0 0 2px #7c5cbf;"></div>
    <div style="font-size:12px;font-weight:700;color:#5a3fa0;">YEAR</div>
    <div style="font-size:11px;color:#555;line-height:1.3;padding:0 4px;">Event</div>
  </div>
</div>
```

### QUESTION BLOCK
```html
<!-- Group label -->
<div style="font-size:10px;font-weight:900;letter-spacing:.1em;text-transform:uppercase;color:#7c5cbf;margin:20px 0 8px;padding-bottom:5px;border-bottom:1px solid #e0d4f5;">Topic Name</div>
<!-- Single question -->
<div style="display:flex;gap:12px;padding:11px 0;border-bottom:.5px solid #ede6f8;align-items:flex-start;">
  <div style="min-width:28px;height:28px;background:#f0ecfa;border:1.5px solid #c9b8e8;color:#5a3fa0;font-weight:800;font-size:12px;border-radius:50%;display:flex;align-items:center;justify-content:center;">1</div>
  <div style="font-size:13px;color:#222;line-height:1.6;padding-top:4px;">Question text <span style="font-size:9.5px;font-weight:700;letter-spacing:.06em;text-transform:uppercase;padding:2px 6px;border-radius:4px;margin-left:7px;background:#f3e5f5;color:#6a1b9a;">key</span></div>
</div>
```
Badge colours: oral=`#e8f5e9/#2e7d32` · test=`#fff3e0/#e65100` · key=`#f3e5f5/#6a1b9a`

### PART HEADER (section divider)
```html
<div style="background:#7c5cbf;color:#fff;font-size:13px;font-weight:900;letter-spacing:.12em;text-transform:uppercase;padding:10px 18px;border-radius:6px;margin-top:38px;margin-bottom:18px;">Part A — Topic Name</div>
```

### BULLET LIST
```html
<ul style="list-style:none;padding:0;margin:8px 0 12px;">
  <li style="position:relative;padding-left:20px;margin-bottom:5px;font-size:13.5px;line-height:1.6;">
    <span style="position:absolute;left:0;color:#7c5cbf;font-weight:700;">→</span>
    Content
  </li>
</ul>
```

### TAG STRIP
```html
<div style="display:flex;flex-wrap:wrap;gap:7px;margin:10px 0 16px;">
  <span style="background:#f0ecfa;color:#5a3fa0;font-size:11.5px;font-weight:600;padding:3px 10px;border-radius:20px;border:1px solid #c9b8e8;">Tag</span>
</div>
```

---

## PDF Conversion (after HTML output)

After presenting the HTML file, ask exactly:
> "Would you like this converted to a PDF as well?"

If yes:
```python
import weasyprint
weasyprint.HTML(filename='notes.html').write_pdf('/mnt/user-data/outputs/notes.pdf')
```
Then `present_files(['/mnt/user-data/outputs/notes.pdf'])`.

---

## Output Checklist (verify before presenting HTML)

- [ ] White background, no ruled lines
- [ ] Every syllabus point covered with at least one section
- [ ] Key terms use `.hl` highlight
- [ ] Definitions in definition boxes
- [ ] Causes/points in grid cards (not plain paragraphs)
- [ ] Notable people in person cards
- [ ] Questions block present if requested
- [ ] No Google Fonts import
- [ ] File saved to `/mnt/user-data/outputs/`

---

## Token Efficiency Rules

1. **Read text-layer PDFs with pdfplumber** — never rasterize a searchable PDF
2. **Rasterize scanned PDFs at 150dpi** (not 300) — sufficient for reading
3. **Skip blank/duplicate pages** — check page character count before viewing
4. **Write HTML in one `create_file` call** — no iterative appending
5. **No debug prints** — silent execution
6. **No narration of internal steps** — only show user the final HTML and the PDF question
