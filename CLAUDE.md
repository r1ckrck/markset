# Markset Skill — Developer Reference

## What This Skill Does

Markset is a self-contained Claude Code skill that converts Markdown to polished PDFs.
It bundles its own Pandoc and Tectonic binaries, fonts, LaTeX template, and Lua filter — no system installs required.

Two capabilities:
1. **Author Markdown** — write documents that conform to the design grammar
2. **Build PDF** — compile those documents to PDF via Pandoc + Tectonic (XeLaTeX)

---

## Directory Layout

```
markset/
├── CLAUDE.md                   This file
├── SKILL.md                    Skill definition (name, trigger description)
├── README.md                   End-user setup guide
├── bin/
│   ├── pandoc-arm64            Pandoc binary — Apple Silicon
│   ├── pandoc-x86_64           Pandoc binary — Intel Mac / Linux x86_64
│   ├── tectonic-arm64          Tectonic binary — Apple Silicon
│   └── tectonic-x86_64         Tectonic binary — Intel Mac / Linux x86_64
├── templates/
│   ├── template.tex            LaTeX template (structural only — consumes theme tokens)
│   ├── apply-theme.lua         Theme pipeline: validate, merge overrides, compute derived, emit \ms* tokens
│   ├── divs.lua                Lua filter — callouts, image placeholders
│   ├── styleguide_md.md        Markdown authoring rules (single source of truth)
│   └── fonts/                  TTF files (Inter + JetBrains Mono — not committed)
├── themes/
│   ├── README.md               How to pick, copy, override a theme
│   ├── SCHEMA.md               Canonical token reference (every key, type, default)
│   └── presets/
│       └── default.yaml        Bundled default theme (warm cream, Inter, compact rhythm)
├── workflow/
│   ├── build-pdf.sh            Build script (run this to produce a PDF)
│   ├── author-markdown.md      Step-by-step authoring instructions for Claude
│   └── build-pdf.md            Step-by-step build instructions for Claude
└── cache/                      Tectonic LaTeX package cache (gitignored, ~300 MB)
```

---

## Build Command

```bash
# Standard invocation (uses themes/presets/default.yaml)
<skill-dir>/workflow/build-pdf.sh <input.md> [output.pdf]

# With a specific theme
<skill-dir>/workflow/build-pdf.sh --theme path/to/theme.yaml <input.md>
```

**Theme resolution** (highest wins): `--theme` flag → `./theme.yaml` next to input → `themes/presets/default.yaml`.

The script auto-detects architecture via `uname -m` and selects the matching binary (falls back to system `pandoc`/`tectonic` on `PATH`).

---

## Customising the Skill

Four axes of change, in order of how often you'll use them:

### Axis 1 — Theme YAML (most common)
Copy `themes/presets/default.yaml` to a new file, edit, point `--theme` at it.
The YAML controls spacing, typography, colours, fonts, and layout variants.
Every key is documented in `themes/SCHEMA.md`.

### Axis 2 — Frontmatter overrides (per-document)
Add `theme_overrides:` to a document's frontmatter with dot-path keys:
```yaml
theme_overrides:
  palette.accent.primary: "#ff6600"
  rhythm.base: 5pt
  type.h1: 22pt
```
Overrides win over the theme file, for this document only.

### Axis 3 — Preset files (named starting points)
Drop a new file in `themes/presets/`. It's just a YAML — anyone can `--theme` it.

### Axis 4 — Template (structural changes only)
`templates/template.tex` is **structural** — it defines layout flow (cover, TOC, headers, body), package loads, and numbering rules. It consumes `\ms*` tokens emitted by `apply-theme.lua`. Edit the template only when a change can't be expressed as a token:
- Adding a new cover-page variant
- Changing what the running header references
- New structural element (e.g. a margin-note block)

### Lua filters
- **`apply-theme.lua`** — runs first. Validates theme, merges `theme_overrides`, resolves palette `{{refs}}`, computes derived spacing/type sizes, emits LaTeX tokens into `header-includes`, sets layout boolean flags on metadata. **Extend this** when adding new tokens.
- **`divs.lua`** — runs second. Handles `:::` fenced divs (callouts, image-placeholders).

### Build script (`workflow/build-pdf.sh`)
- Paths derive from `SKILL_DIR` — no hardcoded absolutes
- Binary resolution: system `PATH` first, then skill-local `bin/<tool>-<arch>`
- Tectonic is symlinked into a temp `PATH` entry so Pandoc finds it by name
- Do not add system-level dependencies — the skill must remain self-contained

### Styleguide (`templates/styleguide_md.md`)
- Read before authoring — do not rely on memory
- Defines all permitted Markdown constructs and their constraints
- Now also documents `theme_overrides` frontmatter

---

## Authoring Rules (summary — read styleguide for full detail)

**Frontmatter** — all five fields are required or the build will fail:
```yaml
---
title: "Document Title"
author: "Arnesh Mandal"
version: "1.0"
date: "2026-01-30"
include-before: |
  \begin{lstlisting}[style=coverasciiart]
  ASCII art here (max 60 chars wide, 16 lines tall)
  \end{lstlisting}
---
```

**Key constraints:**
- One sentence per line; consecutive sentences merge into paragraphs in the PDF
- Headings max H4; Title Case; never add numbers manually (auto-numbered)
- Tables require a caption line (`: Caption text`) immediately after the last row — no blank line between
- Callouts (`:::note`, `:::tip`, `:::warning`, `:::important`) never inside lists or tables
- Code blocks always specify a language identifier
- No raw LaTeX (`\newpage`), HTML (`<div>`), or manual styling

---

## Common Build Failures

| Symptom | Cause | Fix |
|---------|-------|-----|
| Binary not found | `bin/pandoc-<arch>` or `bin/tectonic-<arch>` missing | Add binary per README setup |
| macOS Gatekeeper block | Quarantine attribute on binary | `xattr -d com.apple.quarantine bin/<binary>` |
| Font not found | TTF files absent from `templates/fonts/` | Add Inter + JetBrains Mono TTFs |
| `[markset] theme validation failed` | Theme YAML has missing or malformed values | Read the listed errors — each points to a key path and the offending value |
| LaTeX error — missing field | Frontmatter field omitted | Add all five required fields |
| Table render error | Caption line missing or has blank line before it | Move `: Caption` immediately after table |
| Callout render error | Callout nested inside list or table | Move callout outside the list/table |
: Build failure quick reference

---

## Adding Binaries (when not present)

```bash
# Detect arch
uname -m   # arm64 or x86_64

# Pandoc: download standalone binary from github.com/jgm/pandoc/releases/latest
cp pandoc-*/bin/pandoc .claude/skills/markset/bin/pandoc-arm64
chmod +x .claude/skills/markset/bin/pandoc-arm64

# Tectonic: download from github.com/tectonic-typesetting/tectonic/releases/latest
cp tectonic .claude/skills/markset/bin/tectonic-arm64
chmod +x .claude/skills/markset/bin/tectonic-arm64
```

---

## Environment Notes

- No conda/micromamba env needed — all binaries are bundled in `bin/`
- `OSFONTDIR` is set by the build script to `templates/fonts/` — no system font install required
- `TECTONIC_CACHE_DIR` is set to `cache/` — first build downloads ~300 MB of LaTeX packages; subsequent builds are fast
- The `cache/` directory transfers cleanly across machines (architecture-independent)
