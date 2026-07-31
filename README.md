# Claude_MakeNotes

**A Claude skill that reads any document — PDF, image, or video — and produces structured, aesthetic HTML study notes with a single command.**

---

## What It Does

| Input | Output |
|---|---|
| PDF (text or scanned) | Structured HTML notes |
| Image (JPG, PNG) | Structured HTML notes |
| Video (MP4, MOV) | Structured HTML notes from key frames |
| + Your UI style idea | Notes styled exactly to your preference |
| + Optional: syllabus | Only syllabus topics included, nothing extra |

After the HTML is ready, Claude asks if you want a PDF version too.

---

## Installation

### Step 1 — Install Claude Desktop

Download from [claude.ai/download](https://claude.ai/download) and sign in.

### Step 2 — Locate your skills folder

| OS | Path |
|---|---|
| macOS | `~/Library/Application Support/Claude/skills/` |
| Windows | `%APPDATA%\Claude\skills\` |
| Linux | `~/.config/Claude/skills/` |

If the `skills/` folder does not exist, create it.

### Step 3 — Add this skill

```bash
# Option A — Clone directly into the skills folder
cd ~/Library/Application\ Support/Claude/skills/
git clone https://github.com/Agrazel1459/Claude_MakeNotes.git

# Option B — Download ZIP from GitHub and unzip into the skills folder
```

Your skills folder should look like:
```
skills/
└── Claude_MakeNotes/
    ├── SKILL.md
    └── README.md
```

### Step 4 — Install required system packages

Open Terminal and run:

```bash
# Python packages
pip install weasyprint pdfplumber pillow --break-system-packages

# System tools (macOS with Homebrew)
brew install poppler ffmpeg

# System tools (Ubuntu/Debian)
sudo apt-get install -y poppler-utils ffmpeg
```

> **Windows users:** Install [Poppler for Windows](https://github.com/oschwartz10612/poppler-windows/releases) and add it to PATH. Install ffmpeg from [ffmpeg.org](https://ffmpeg.org/download.html).

### Step 5 — Restart Claude Desktop

Fully quit and reopen Claude Desktop. The skill is now active.

---

## How to Use

### Basic usage

1. Open Claude Desktop
2. Upload your file(s) — PDF, image, or video
3. Type the trigger command:

```
/CreateNotes
```

4. Claude will ask you two things in one message:
   - Your **UI style idea** for the notes
   - Your **syllabus/topic list** (optional — if you only want specific topics covered)

5. Claude reads the document, generates the HTML notes, and presents the file.

6. Claude then asks: *"Would you like this converted to a PDF as well?"* — reply Yes or No.

---

## UI Style Ideas (what to tell Claude)

You can describe any visual style. Examples:

| What you say | What you get |
|---|---|
| `purple highlights, card layout` | Purple marker highlights on key terms, content in bordered cards |
| `Cornell format` | Two-column layout with cues on left, notes on right, summary at bottom |
| `timeline style` | Events laid out on a horizontal or vertical timeline |
| `minimal dark` | Dark background, light text, clean lines |
| `comparison tables` | Side-by-side comparison grids for contrasting topics |
| `person cards for each figure` | Dedicated styled card per historical/notable person |
| `icon grid for impacts` | 3-column grid with emoji icons for each impact/consequence |

You can mix styles: *"purple highlights, causes in numbered cards, comparison table for capitalism vs socialism, 25 questions at the end tagged oral/test/key"*

---

## Syllabus Filtering

If you only want specific topics covered (and want the rest excluded), paste your syllabus:

```
/CreateNotes

Syllabus:
3. Jainism and Buddhism
   - Sources: Angas, Tripitikas, Jatakas
   - Causes for rise in 6th century BC
   - Doctrines

UI: purple highlights, numbered cause cards, definition boxes, 25 questions at end
```

Claude will include **only** those topics and exclude everything else from the document.

---

## Multiple Files

You can upload multiple files at once:

```
/CreateNotes
[upload: chapter3.pdf, diagram.png]

UI: clean white, card layout, timeline for dates, 20 questions
```

Claude reads all files and merges the content into one set of notes.

---

## Output Location

All generated files are saved to:
```
/mnt/user-data/outputs/
```

Both the `.html` and `.pdf` (if requested) appear as downloadable links in Claude's response.

---

## Design Defaults

All notes use these design rules by default (you can override with your UI idea):

- **Background:** Pure white — no ruled lines, no coloured paper
- **Accent colour:** Purple `#7c5cbf`
- **Highlight:** Purple marker effect on key terms
- **Headings:** Bold black, uppercase with extending line
- **Font:** System fonts (Arial/Helvetica) — works correctly in PDF
- **Max width:** 820px, properly padded for A4 printing

---

## Strict Rules (do not ask Claude to break these)

- Notes will **always** use white background — the ruled-paper effect is permanently disabled
- Google Fonts are **never** imported — they break PDF rendering
- Only the **syllabus topics** will be covered if a syllabus is provided
- Claude will **not** narrate its internal reading/processing steps — it works silently and presents the finished notes
- The PDF conversion step is **always** optional — Claude will ask before converting

---

## Troubleshooting

| Problem | Fix |
|---|---|
| `/CreateNotes` not recognized | Restart Claude Desktop; confirm `SKILL.md` is inside `Claude_MakeNotes/` folder |
| PDF is blank / no text | The PDF is scanned — Claude will automatically rasterize it at 150dpi |
| `weasyprint` PDF conversion fails | Run `pip install weasyprint --break-system-packages` in Terminal |
| Video frames not reading | Install ffmpeg: `brew install ffmpeg` (mac) or `sudo apt install ffmpeg` (linux) |
| Notes missing a topic | Re-run with a more explicit syllabus list specifying exactly what to include |

---

## Repository

[github.com/Agrazel1459/Claude_MakeNotes](https://github.com/Agrazel1459/Claude_MakeNotes)

---

## License

MIT License — free to use, modify and distribute.
