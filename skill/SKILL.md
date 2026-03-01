---
name: brand-doc
description: Create a professionally branded Word document (.docx) for either Automate Accelerator or Inspra AI. Use when the user asks to create a document, report, proposal, letter, brief, or any client deliverable for either brand.
allowed-tools: Bash, Write, Read, Glob
---

# Brand Document Generator

You create professionally branded Word documents (.docx) for two brands:

- **Automate Accelerator** (AA)
- **Inspra AI**

## Brand Assets Location

All logos are stored at: `~/.claude/skills/brand-doc/assets/`

| Brand | Logo File | Use Case |
|-------|-----------|----------|
| Automate Accelerator | `AA-logo-dark.png` | Full-colour logo on white/light backgrounds |
| Automate Accelerator | `AA-logo-white.png` | White logo on dark/black backgrounds |
| Inspra AI | `Inspra-logo-dark.png` | Dark logo on light/white backgrounds |
| Inspra AI | `Inspra-logo-light.png` | White/light logo on dark backgrounds |

In Python scripts, resolve the path like this:
```python
import os
ASSETS_DIR = os.path.expanduser("~/.claude/skills/brand-doc/assets")
# Then reference logos as:
# os.path.join(ASSETS_DIR, "AA-logo-dark.png")    ← default (light backgrounds)
# os.path.join(ASSETS_DIR, "AA-logo-white.png")   ← dark backgrounds only
# os.path.join(ASSETS_DIR, "Inspra-logo-dark.png")
# os.path.join(ASSETS_DIR, "Inspra-logo-light.png")
```

## Brand Colors & Styles

### Automate Accelerator
- **Primary orange:** `#F47920` → RGB(244, 121, 32)
- **Primary purple:** `#412F8F` → RGB(65, 47, 143)
- **Background:** Always white `#FFFFFF` — AA brand is light/white background, NOT dark
- **Logo to use:** `AA-logo-dark.png` (full-colour orange + purple) — always on white background
- **NEVER use:** `AA-logo-white.png` or dark `#1A1A1A` header strips for standard documents
- **Body text:** `#1A1A1A`
- **Accent/rule color:** `#F47920`
- **Font:** Gilroy (all weights — Thin, Light, Regular, Medium, SemiBold, ExtraBold)
- **Footer:** "Automate Accelerator  |  automateaccelerator.com.au"

### Inspra AI
- **Primary green:** `#39E100` → RGB(57, 225, 0)
- **Header background:** White `#FFFFFF`
- **Body text:** `#1A1A1A`
- **Accent/rule color:** `#39E100`
- **Font:** Calibri
- **Footer:** "Inspra AI  |  inspra.ai"

## Workflow

### Step 1 — Gather Info

Ask the user (you can ask all at once):

1. **Which brand?** — Automate Accelerator or Inspra AI?
2. **Document type?** — Letter / Proposal / Report / Brief / Meeting Notes / Other?
3. **Document title** — e.g. "Proposal for XYZ Client"
4. **Recipient** — Name and company (if applicable)
5. **Content** — What should go in the document? Ask them to describe the sections and content, or paste it in. If it is a blank letterhead template, say so.
6. **Output filename** — suggest `[Brand]_[Title]_[Date].docx` as default, saved to Desktop.

### Step 2 — Generate & Run Python Script

Write a Python script using `python-docx` and run it with Bash. Install if needed:
```bash
pip3 install python-docx -q
```

#### Document Structure

**Automate Accelerator:**
- Header: WHITE background with `AA-logo-dark.png` (full-colour orange + purple logo, left-aligned, width ~2 inches) — NO dark header strips
- Orange horizontal rule line below logo
- Document title: Bold, 16pt, orange (`#F47920`)
- Recipient / date block: 11pt, dark grey
- Body content: Calibri 11pt, `#1A1A1A`, 1.15 line spacing
- Section headings: Bold 13pt, orange
- Footer: 9pt, grey, centered

#### PDF Page-Break Rules (MANDATORY — apply to every AA PDF)

These rules prevent tables and section headers from splitting awkwardly across pages:

1. **`sec()` — always keep header + first item only**
   — wrapping the whole section in `KeepTogether` causes large white gaps when the block
     is bigger than remaining page space. Only glue the header to its first item.

2. **`make_table()` — only wrap tables with ≤4 data rows in `KeepTogether`**
   — larger tables must flow freely to avoid white gaps. Never wrap a 6+ row table.

3. **Never wrap single Flowables (SloganBox, CoreMsgBox, QuoteBlock) in `KeepTogether`**
   — a single Flowable cannot split across pages anyway. Wrapping adds overhead with no benefit
     and can trigger the same white-gap jump.

4. **Never use `canvas.rect()` for decorative bars at y=0 or y≤5mm**
   — they bleed off the page edge. Footer canvas calls must stay within `10–15mm` from bottom.

**Standard helper pattern (copy exactly for every AA PDF):**

```python
from reportlab.platypus import KeepTogether

def make_table(data, col_widths, style_fn=None):
    t = Table(data, colWidths=col_widths)
    t.setStyle(style_fn() if style_fn else tbl2())
    num_data_rows = len(data) - 1
    return KeepTogether(t) if num_data_rows <= 4 else t

def sec(divider_text, content_items, colour=AA_PURPLE, gap=3*mm):
    from reportlab.platypus import Table as RLTable
    div = SectionDivider(divider_text, CW, colour=colour)
    sp  = Spacer(1, gap)
    items = list(content_items)
    # If first item is a Table, only glue div+spacer together (not the table).
    # Including a table in KeepTogether forces the whole block to jump pages,
    # leaving a large white gap. Tables manage their own keep logic via make_table().
    if items and isinstance(items[0], RLTable):
        return [KeepTogether([div, sp])] + items
    # For non-table first items, glue header to first item to prevent orphaned headers
    return [KeepTogether([div, sp, items[0]])] + items[1:]
```

**Inspra AI:**
- Header: White background with Inspra dark logo (left-aligned, width ~2 inches)
- Green horizontal rule line below header
- Document title: Bold, 16pt, dark (`#1A1A1A`)
- Recipient / date block: 11pt, dark grey
- Body content: Calibri 11pt, `#1A1A1A`, 1.15 line spacing
- Section headings: Bold 13pt, green (`#39E100`)
- Footer: 9pt, grey, centered

#### Python Script Pattern

```python
from docx import Document
from docx.shared import Inches, Pt, RGBColor, Cm
from docx.enum.text import WD_ALIGN_PARAGRAPH
from docx.oxml.ns import qn
from docx.oxml import OxmlElement
from datetime import date
import os

ASSETS_DIR = os.path.expanduser("~/.claude/skills/brand-doc/assets")
OUTPUT_PATH = os.path.expanduser("~/Desktop/Document.docx")  # adjust per user

def add_horizontal_rule(doc, r, g, b):
    """Add a colored horizontal rule paragraph."""
    p = doc.add_paragraph()
    p.paragraph_format.space_before = Pt(0)
    p.paragraph_format.space_after = Pt(6)
    pPr = p._p.get_or_add_pPr()
    pBdr = OxmlElement('w:pBdr')
    bottom = OxmlElement('w:bottom')
    bottom.set(qn('w:val'), 'single')
    bottom.set(qn('w:sz'), '12')
    bottom.set(qn('w:space'), '1')
    color_hex = '{:02X}{:02X}{:02X}'.format(r, g, b)
    bottom.set(qn('w:color'), color_hex)
    pBdr.append(bottom)
    pPr.append(pBdr)
    return p

def set_cell_background(cell, hex_color):
    """Set table cell background color."""
    tc = cell._tc
    tcPr = tc.get_or_add_tcPr()
    shd = OxmlElement('w:shd')
    shd.set(qn('w:val'), 'clear')
    shd.set(qn('w:color'), 'auto')
    shd.set(qn('w:fill'), hex_color)
    tcPr.append(shd)

def create_aa_document(title, recipient, content_sections, output_path):
    doc = Document()
    logo_path = os.path.join(ASSETS_DIR, "AA-logo-dark.png")  # full-colour on white

    for section in doc.sections:
        section.top_margin = Cm(1.5)
        section.bottom_margin = Cm(2)
        section.left_margin = Cm(2.5)
        section.right_margin = Cm(2.5)

    # Header: White background with full-colour AA logo — NO dark strip
    header = doc.sections[0].header
    header.paragraphs[0].alignment = WD_ALIGN_PARAGRAPH.LEFT
    header.paragraphs[0].paragraph_format.space_before = Pt(4)
    header.paragraphs[0].paragraph_format.space_after = Pt(4)
    run = header.paragraphs[0].add_run()
    run.add_picture(logo_path, width=Inches(2.0))

    # Orange rule — official AA orange #F47920
    add_horizontal_rule(doc, 244, 121, 32)

    # Date & Recipient
    if recipient:
        p = doc.add_paragraph()
        p.paragraph_format.space_after = Pt(4)
        run = p.add_run(f"To: {recipient}")
        run.font.size = Pt(10)
        run.font.color.rgb = RGBColor(100, 100, 100)

    p = doc.add_paragraph()
    p.paragraph_format.space_after = Pt(12)
    run = p.add_run(f"Date: {date.today().strftime('%d %B %Y')}")
    run.font.size = Pt(10)
    run.font.color.rgb = RGBColor(100, 100, 100)

    # Title
    p = doc.add_paragraph()
    p.paragraph_format.space_after = Pt(12)
    run = p.add_run(title)
    run.bold = True
    run.font.size = Pt(18)
    run.font.color.rgb = RGBColor(244, 121, 32)

    # Body Content
    for section_heading, section_body in content_sections:
        if section_heading:
            p = doc.add_paragraph()
            p.paragraph_format.space_before = Pt(12)
            p.paragraph_format.space_after = Pt(4)
            run = p.add_run(section_heading)
            run.bold = True
            run.font.size = Pt(13)
            run.font.color.rgb = RGBColor(244, 121, 32)
        if section_body:
            p = doc.add_paragraph(section_body)
            p.paragraph_format.space_after = Pt(8)
            for run in p.runs:
                run.font.size = Pt(11)
                run.font.color.rgb = RGBColor(26, 26, 26)

    # Footer
    footer = doc.sections[0].footer
    footer.paragraphs[0].clear()
    footer.paragraphs[0].alignment = WD_ALIGN_PARAGRAPH.CENTER
    run = footer.paragraphs[0].add_run("Automate Accelerator  |  automateaccelerator.com.au")
    run.font.size = Pt(8)
    run.font.color.rgb = RGBColor(150, 150, 150)

    doc.save(output_path)
    print(f"Document saved: {output_path}")


def create_inspra_document(title, recipient, content_sections, output_path):
    doc = Document()
    logo_path = os.path.join(ASSETS_DIR, "Inspra-logo-dark.png")

    for section in doc.sections:
        section.top_margin = Cm(1.5)
        section.bottom_margin = Cm(2)
        section.left_margin = Cm(2.5)
        section.right_margin = Cm(2.5)

    # Header: White background with dark logo
    header = doc.sections[0].header
    header.paragraphs[0].alignment = WD_ALIGN_PARAGRAPH.LEFT
    header.paragraphs[0].paragraph_format.space_before = Pt(6)
    header.paragraphs[0].paragraph_format.space_after = Pt(6)
    run = header.paragraphs[0].add_run()
    run.add_picture(logo_path, width=Inches(2.0))

    # Green rule
    add_horizontal_rule(doc, 57, 225, 0)

    # Date & Recipient
    if recipient:
        p = doc.add_paragraph()
        p.paragraph_format.space_after = Pt(4)
        run = p.add_run(f"To: {recipient}")
        run.font.size = Pt(10)
        run.font.color.rgb = RGBColor(100, 100, 100)

    p = doc.add_paragraph()
    p.paragraph_format.space_after = Pt(12)
    run = p.add_run(f"Date: {date.today().strftime('%d %B %Y')}")
    run.font.size = Pt(10)
    run.font.color.rgb = RGBColor(100, 100, 100)

    # Title
    p = doc.add_paragraph()
    p.paragraph_format.space_after = Pt(12)
    run = p.add_run(title)
    run.bold = True
    run.font.size = Pt(18)
    run.font.color.rgb = RGBColor(26, 26, 26)

    # Body Content
    for section_heading, section_body in content_sections:
        if section_heading:
            p = doc.add_paragraph()
            p.paragraph_format.space_before = Pt(12)
            p.paragraph_format.space_after = Pt(4)
            run = p.add_run(section_heading)
            run.bold = True
            run.font.size = Pt(13)
            run.font.color.rgb = RGBColor(57, 225, 0)
        if section_body:
            p = doc.add_paragraph(section_body)
            p.paragraph_format.space_after = Pt(8)
            for run in p.runs:
                run.font.size = Pt(11)
                run.font.color.rgb = RGBColor(26, 26, 26)

    # Footer
    footer = doc.sections[0].footer
    footer.paragraphs[0].clear()
    footer.paragraphs[0].alignment = WD_ALIGN_PARAGRAPH.CENTER
    run = footer.paragraphs[0].add_run("Inspra AI  |  inspra.ai")
    run.font.size = Pt(8)
    run.font.color.rgb = RGBColor(150, 150, 150)

    doc.save(output_path)
    print(f"Document saved: {output_path}")
```

Adapt the script pattern above for the specific document. For complex documents with tables, bullets, and multiple heading levels, extend the pattern as needed — the key is to maintain brand colors, fonts, logo placement, and footer consistently.

### Step 3 — Confirm Output

After running, tell the user:
- Where the file was saved
- Offer to make adjustments (change content, add tables, add signature block, etc.)

## Writing Rules (Apply to ALL output: documents, emails, drafts)

These rules are non-negotiable. Apply them every time you write text for the user, whether it is a .docx document, an email draft, or plain text output.

### No contractions
NEVER use contractions in any written output. Write "do not" instead of "don't", "it is" instead of "it's", "we are" instead of "we're", etc. This applies to documents, emails, and all drafts without exception.

### No dashes in emails
NEVER use em dashes, en dashes, or hyphens as punctuation in email drafts. Replace them with commas wherever it flows naturally. The only acceptable use of a hyphen is inside compound words (e.g., "non-refundable", "client-facing", "tick-to-accept").

**Bad:** "The terms were written for a platform business — we now deliver managed services."
**Good:** "The terms were written for a platform business, we now deliver managed services."

**Bad:** "Three priority areas for the solicitor -- website terms, proposals, alignment."
**Good:** "Three priority areas for the solicitor, website terms, proposals, and alignment."

### Australian English
Use Australian English spelling in all output: organisation, colour, recognise, optimise, analyse, defence, licence (noun), practise (verb).

## Tips

- If the user says "blank letterhead", create the header/footer structure with a placeholder body paragraph only.
- If the user pastes content, break it into sections intelligently.
- Always save to Desktop by default unless told otherwise.
- Write the Python script to a temp file, run it, then delete the temp file.
