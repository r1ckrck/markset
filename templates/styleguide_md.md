# Markdown Styleguide

## Frontmatter
Required fields — all must be present:
```yaml
---
title: "Document Title"
author: "Arnesh Mandal"
version: "1.0"
date: "2026-01-30"
include-before: |
  \begin{lstlisting}[style=coverasciiart]
  Your ASCII art here (max 60 chars wide, 16 lines tall)
  \end{lstlisting}
---
```

Optional — theme customisation:
```yaml
theme_overrides:
  palette.accent.primary: "#ff6600"   # one-off accent colour
  spacing.unit: 5pt                    # bump the rhythm
  blocks.heading.h1.before: 36pt       # more air above H1
  layout.cover.style: title-top        # swap cover layout
```
Dot-path keys address any theme field — see `themes/SCHEMA.md` for the full map.

## Structure
- One sentence per line (source readability)
- Consecutive sentences combine into paragraphs in PDF
- Use bullets for separate points that should not combine into paragraphs
- Blank line between paragraphs or sections
- No indentation

## Headings
- Max depth: H4
- Title Case: `# Getting Started With the API`
- Auto-numbered (never manual)
- No blank line after heading, blank line before next heading

## Lists
- Max 2 levels nesting
- No callouts or complex blocks inside

```markdown
- Parent
  - Child
1. First
2. Second
```

## Tables
Caption mandatory (`:` line immediately after, no blank):
Use dash count to control the colomn width ratios
Distribute based on content length (short columns = fewer dashes, long columns = more)
```markdown
| Col A | Col B |
|-------|-------|
| Data  | Data  |
: Table caption
```

## Code
- Inline: `` `code` ``
- Blocks: always specify language, aim <80 chars, no blank line before

````markdown
```python
def hello():
    print("world")
```
````

## ASCII Art
- Use `ascii` class for diagrams/art
- max width of cover ascii art: 82
- max height of cover ascii art: 

````markdown
```ascii
   +---------+
   |  Box    |
   +---------+
```
````
## Callouts
Types: `note`, `tip`, `warning`, `important`
Never inside lists/tables or nested.

```markdown
::: note
Content here.
:::
```

## Images
Use only when necessary. Formats: PNG, JPG, PDF.

Two variants — pick based on role:

**Supporting image** (default, 60% column width — sits inside the reading column):
```markdown
![Caption](path.png)
```

**Full-bleed image** (100% column width — hero / diagram / feature shot):
```markdown
![Caption](path.png){width=100%}
```

Any width override is accepted (e.g. `{width=80%}`), but prefer the two variants above for consistency.

Placeholder (when the image is not yet available):
```markdown
::: image-placeholder
Figure: Title
Description: What it shows
Dimensions: supporting | full-bleed
:::
```

## Links
`[descriptive text](url)` - never "click here" or raw URLs

## Other Elements
- **Block quotes**: `> text` (use sparingly)
- **Horizontal rules**: `---` (use sparingly)
- **Footnotes**: `text[^1]` + `[^1]: Note` at end (use sparingly)
- **Icons/Emojis**: Use simple ASCII characters

## Prohibited
No raw LaTeX (`\newpage`), HTML (`<div>`), or manual styling (fonts/colors/spacing).
Template handles all visual decisions.