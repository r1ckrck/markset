# Theme File Plan — `templates/theme.tex`

## Goal

Extract all design tokens (colors, spacing, typography, layout) from `template.tex` into a single `theme.tex` file. A customiser only ever touches `theme.tex`. `template.tex` becomes purely structural.

---

## Integration

**`workflow/build-pdf.sh`** — add one variable:

```bash
-V themefile="$SKILL_DIR/templates/theme.tex"
```

**`templates/template.tex`** — add near the top (after `\documentclass`, before packages):

```latex
\input{$themefile$}
```

All hardcoded values in `template.tex` are replaced with named tokens defined in `theme.tex`.

---

## File Structure

```
templates/
├── theme.tex       ← NEW — all design tokens live here
├── template.tex    ← modified — structural only, uses tokens
├── divs.lua
├── styleguide_md.md
└── fonts/
```

---

## Section 1 — Spacing Unit (derived system)

A single `\msunit` drives all block and element spacing. Change one value, all derived spacing scales proportionally.

```latex
\newlength{\msunit}
\setlength{\msunit}{4pt}   % ← change this to scale all derived spacing
```

Five named tokens are derived from it:

| Token | Multiple | Value at 4pt | Used for |
|---|---|---|---|
| `\msspacexs` | 0.5u | 2pt | inline code padding |
| `\msspacesm` | 1u | 4pt | callout top/bottom, heading after |
| `\msspacemed` | 2u | 8pt | code margins, heading before H2–H4 |
| `\msspacelg` | 3u | 12pt | code belowskip, heading before H1 |
| `\msspacexl` | 4u | 16pt | image placeholder padding |

### What derives from `\msunit`

| Usage in template | Token | Current value |
|---|---|---|
| H1 `titlespacing` before | `\msspacelg` | 12pt |
| H2/H3/H4 `titlespacing` before | `\msspacemed` | 8pt |
| Callout `left` / `right` padding | `2.5 × \msunit` | 10pt |
| Callout `top` / `bottom` | `\msspacesm` | 4pt |
| Callout `before skip` / `after skip` | `\msspacesm` | 4pt |
| Code block `xleftmargin` / `xrightmargin` | `\msspacemed` | 8pt |
| Code block `belowskip` | `\msspacelg` | 12pt |
| Inline code `\fboxsep` | `\msspacexs` | 2pt |
| Image placeholder `left`/`right`/`top`/`bottom` | `\msspacexl` | 12pt |
| Image placeholder `before skip` / `after skip` | `\msspacemed` | 8pt |

### What does NOT derive from `\msunit`

These are independent design decisions with their own rationale:

| Value | Reason for independence |
|---|---|
| Page margins (top/bottom/left/right) | Layout concern, mm-based, unrelated to body rhythm |
| Rule widths (0.5pt, 0.75pt, 3pt) | Visual weight decisions, not spacing rhythm |
| Table `\tabcolsep` / `\arraystretch` | Typographic density — separate control axis |
| TOC indents | Structural, em-based alignment |
| List `\leftmargin` / `\itemsep` | Typographic alignment, not block rhythm |
| `headheight`, `headsep`, `footskip` | Header geometry — layout concern |

---

## Section 2 — Colors

All 17 color definitions moved from `template.tex` to `theme.tex`, keeping existing names (no renaming — minimises template.tex changes). Grouped semantically with comments.

```latex
% --- Accent ---
\definecolor{secondary}{HTML}{ca9ee6}
\definecolor{accentmuted}{HTML}{a080c0}

% --- Backgrounds ---
\definecolor{pagebg}{HTML}{faf7f2}
\definecolor{codebg}{HTML}{f0ece4}
\definecolor{calloutbg}{HTML}{f4f0e8}
\definecolor{placeholderbg}{HTML}{ece8e0}

% --- Text ---
\definecolor{bodytext}{HTML}{181826}
\definecolor{secondarytext}{HTML}{3d3d4d}
\definecolor{tertiarytext}{HTML}{5d5d6d}

% --- Rules / borders ---
\definecolor{tablerule}{HTML}{d4d0c6}
\definecolor{placeholderborder}{HTML}{c4c0b6}

% --- Callout borders ---
\definecolor{noteborder}{HTML}{8d8d9d}
\definecolor{tipborder}{HTML}{ca9ee6}
\definecolor{warningborder}{HTML}{d4a574}
\definecolor{importantborder}{HTML}{a080c0}

% --- Syntax highlighting ---
\definecolor{synstring}{HTML}{a080c0}
\definecolor{syncomment}{HTML}{5d5d6d}
\definecolor{synnumber}{HTML}{3d3d4d}
```

---

## Section 3 — Typography (partially derived)

`\msfontbody` is the single base. All other sizes derive from it using `\dimexpr` integer arithmetic. Change `\msfontbody` from 10pt → 12pt and all sizes scale proportionally.

```latex
\newlength{\msfontbody}
\setlength{\msfontbody}{10pt}   % ← change this to scale all type
```

### Scale table

Scale factors are defined as integer ratios (× 10) to work with `\dimexpr`.

| Token | Scale | Value at 10pt | Used for |
|---|---|---|---|
| `\msfonth1` | 16/10 × body | 16pt | H1 heading |
| `\msfonth2` | 13/10 × body | 13pt | H2 heading |
| `\msfonth3` | 11/10 × body | 11pt | H3 heading |
| `\msfonth4` | 10/10 × body | 10pt | H4 heading |
| `\msfontsmall` | 9/10 × body | 9pt | code blocks, captions, inline code |
| `\msfontxsmall` | 8/10 × body | 8pt | header/footer, footnotes |
| `\msfontxxsmall` | 6/10 × body | 6pt | code line numbers |

### Line heights (leading)

Headings: `size + 3pt` (matches current values at all sizes).
Body: `14/10 × body` (1.4× ratio).

| Token | Formula | Value at 10pt |
|---|---|---|
| `\msfontbodylead` | 14/10 × body | 14pt |
| `\msfonth1lead` | H1 + 3pt | 19pt |
| `\msfonth2lead` | H2 + 3pt | 16pt |
| `\msfonth3lead` | H3 + 3pt | 14pt |
| `\msfonth4lead` | H4 + 3pt | 13pt |
| `\msfontsmallead` | small + 3pt | 12pt |
| `\msfontxsmalllead` | xsmall + 2pt | 10pt |
| `\msfontxxsmalllead` | xxsmall + 2pt | 8pt |

### Font names and mono scale

Font family names and the monospace scale factor are defined as macros (not lengths) so they can be used in `\setmainfont` / `\setmonofont`.

```latex
\newcommand{\msmainfont}{Inter}
\newcommand{\msmonofont}{JetBrainsMono}
\newcommand{\msmonoscale}{0.9}    % mono font size relative to body
```

### What stays independent from `\msfontbody`

| Value | Reason |
|---|---|
| `coverasciiart` font size (14pt) | Presentational special case, fixed visual size |
| `asciiart` font size (12pt) | Same — decorative, not a body-relative size |
| TOC entry sizes | These mirror body size but are set explicitly |
| Quote / callout body size | Mirrors body — set to `\msfontbody` directly |

---

## Section 4 — Page Layout and Independent Values

These are not derived from any unit. Grouped for easy access.

```latex
% --- Page margins ---
\newlength{\mspagemargtop}     \setlength{\mspagemargtop}{20mm}
\newlength{\mspagemagbottom}   \setlength{\mspagemagbottom}{15mm}
\newlength{\mspagemargleft}    \setlength{\mspagemargleft}{12.5mm}
\newlength{\mspagemagright}    \setlength{\mspagemagright}{12.5mm}
\newlength{\msheadheight}      \setlength{\msheadheight}{12pt}
\newlength{\msheadsep}         \setlength{\msheadsep}{6pt}
\newlength{\msfootskip}        \setlength{\msfootskip}{10pt}

% --- Rule weights ---
\newlength{\msrulethin}        \setlength{\msrulethin}{0.5pt}
\newlength{\msrulethick}       \setlength{\msrulethick}{0.75pt}
\newlength{\mscalloutborder}   \setlength{\mscalloutborder}{3pt}

% --- Tables ---
\newlength{\mstablecolsep}     \setlength{\mstablecolsep}{8pt}
\newcommand{\mstablearraystretch}{1.4}

% --- Lists ---
\newlength{\mslistindent}      \setlength{\mslistindent}{14pt}
\newlength{\mslistindentsub}   \setlength{\mslistindentsub}{16pt}
\newlength{\mslistitemsep}     \setlength{\mslistitemsep}{3pt}
```

---

## Summary of Changes

| File | Change |
|---|---|
| `templates/theme.tex` | **Created** — all tokens defined here |
| `templates/template.tex` | Add `\input{$themefile$}`; remove color block; replace all hardcoded values with tokens |
| `workflow/build-pdf.sh` | Add `-V themefile="$SKILL_DIR/templates/theme.tex"` |

### Touched areas in `template.tex`

| Area | Lines (approx) | What changes |
|---|---|---|
| Colors block | 44–72 | Removed (moved to theme.tex) |
| geometry | 13–21 | Values → `\mspagemargtop` etc. |
| fontspec | 25–39 | Names → `\msmainfont`, `\msmonofont`, `\msmonoscale` |
| fancyhdr font sizes | 90–91 | `8pt/10pt` → `\msfontxsmall`/`\msfontxsmalllead` |
| titlesection heading sizes | 104–135 | `16pt/19pt` etc. → `\msfonth1`/`\msfonth1lead` etc. |
| titlespacing before-skip | 111, 119, 127, 135 | `12pt`/`8pt` → `\msspacelg`/`\msspacemed` |
| tocloft font sizes | 166–197 | `10pt/12pt` etc. → tokens |
| table settings | 210–215 | `8pt`/`1.4` → `\mstablecolsep`/`\mstablearraystretch` |
| lstdefinestyle codeblock | 228–251 | margin/skip values → spacing tokens |
| inlinecode | 304–307 | `2pt`/`9pt/11pt` → `\msspacexs`/`\msfontsmall` etc. |
| calloutbase tcolorbox | 313–330 | padding/skip → spacing tokens |
| callout border width | 319 | `3pt` → `\mscalloutborder` |
| imageplaceholder | 357–376 | padding/skip → spacing tokens |
| captionsetup | 384–405 | font sizes → tokens |
| footnotesize | 469 | `8pt/10pt` → `\msfontxsmall`/`\msfontxsmalllead` |
| title block on cover | 509–511 | `16pt/19pt`, `9pt/11pt` → tokens |

---

## Customisation Guide (for the theme file header)

The theme file will open with a usage comment explaining the three axes of control:

```
AXIS 1 — Spacing:  change \msunit (e.g. 4pt → 5pt) → all block spacing scales
AXIS 2 — Type:     change \msfontbody (e.g. 10pt → 12pt) → all font sizes scale
AXIS 3 — Colors:   change any \definecolor hex value → instant palette swap
AXIS 4 — Layout:   change page margins, rule weights, list/table settings independently
```
