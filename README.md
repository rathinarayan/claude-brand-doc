# claude-brand-doc

A Claude Code skill that generates professionally branded Word documents (`.docx`) for two brands:
- **Inspra AI**
- **Automate Accelerator**

## What it does

Invoke this skill whenever you need to produce a client-facing document — proposals, reports, letters, briefs, meeting notes, or any other deliverable. It handles layout, brand colours, logo placement, fonts, and footer automatically.

**Supported document types:** Letter · Proposal · Report · Brief · Meeting Notes · Blank Letterhead

## Brand specs built in

| Brand | Primary Colour | Font |
|-------|---------------|------|
| Inspra AI | Green `#39E100` | Calibri |
| Automate Accelerator | Orange `#F47920` + Purple `#412F8F` | Gilroy |

## Installation

1. Clone this repository:
   ```bash
   git clone https://github.com/rathinarayan/claude-brand-doc.git
   ```

2. Copy the skill folder into your Claude Code skills directory:
   ```bash
   cp -r claude-brand-doc/skill ~/.claude/skills/brand-doc
   ```

3. Restart Claude Code. The skill loads automatically.

## How to invoke

Type `/brand-doc` in any Claude Code session, or describe what you need:

> "Create an Inspra AI proposal for SkyMesh covering the Phase 1 integration scope."

> "Draft an Automate Accelerator cover letter to ABC Company."

Claude will ask for:
1. Which brand (Inspra AI or Automate Accelerator)
2. Document type
3. Title and recipient
4. Content (paste or describe)
5. Output filename

The `.docx` file is saved to your Desktop by default.

## Writing standards applied automatically

- No contractions (`do not` not `don't`)
- No em/en dashes in email or document body
- Australian English spelling throughout
- Professional tone consistent with each brand

## Folder structure

```
skill/
  SKILL.md          ← Skill definition and instructions
  assets/
    AA-logo-dark.png
    AA-logo-white.png
    Inspra-logo-dark.png
    Inspra-logo-light.png
```

## Requirements

- Claude Code (latest)
- Python 3 with `python-docx` installed: `pip3 install python-docx`
