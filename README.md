# markset

A Claude Code skill that turns markdown into polished PDFs — authored by AI, rendered with a fixed design grammar.

---

## Why This Exists

AI is genuinely good at writing structured content in markdown. The problem is that markdown doesn't carry any design — when you share it, you're sharing a plain text file. Converting it to a Word document or pasting it into a design tool defeats the purpose.

markset takes a different approach: define a strict design grammar once (typography, layout, colors, spacing, heading hierarchy), encode it in a LaTeX template, and let Claude write to that grammar. Every document Claude authors through this skill looks the same — consistent, intentional, shareable — without any manual formatting.

The workflow is intentionally simple: Claude writes markdown, Claude runs a script, you get a PDF.

---

## How It Works

```
Markdown  →  Pandoc  →  Tectonic (XeLaTeX)  →  PDF
               ↑               ↑
         Lua filter       LaTeX template
         (callouts,       (typography,
          divs)           layout, fonts)
```

- **Pandoc** converts markdown to LaTeX using a custom template
- **Tectonic** compiles the LaTeX to PDF (self-contained, no TeX Live install needed)
- **Lua filter** handles custom markdown syntax (callouts, image placeholders)
- **LaTeX template** enforces the design — fonts, spacing, colors, heading hierarchy, table styling — all fixed, no overrides possible from markdown

The skill ships its own Pandoc and Tectonic binaries. No system-level tools required beyond bash.

---

## Compatibility

| Platform | Architecture | Status |
|----------|-------------|--------|
| macOS (Apple Silicon) | arm64 | Supported |
| macOS (Intel) | x86_64 | Supported |
| Linux | x86_64 | Supported — add Linux binaries to `bin/` |
| Linux | arm64 | Not included — add binaries manually |
| Windows | any | Not supported (build script requires bash) |
: Platform compatibility

**Runtime requirements:** bash, `uname`, `mktemp`, `ln` — standard on all Unix systems.

**Binary versions tested:**
- Pandoc 3.x (3.9+)
- Tectonic 0.15+

---

## What's Included

```
markset/
├── SKILL.md                  Claude skill definition and capability index
├── README.md                 This file
├── bin/
│   ├── pandoc-arm64          Pandoc binary (macOS Apple Silicon)
│   ├── pandoc-x86_64         Pandoc binary (macOS Intel / Linux x86_64)
│   ├── tectonic-arm64        Tectonic binary (macOS Apple Silicon)
│   └── tectonic-x86_64       Tectonic binary (macOS Intel / Linux x86_64)
├── templates/
│   ├── template.tex          LaTeX template (design grammar)
│   ├── divs.lua              Lua filter for custom syntax
│   ├── styleguide_md.md      Markdown authoring rules for Claude
│   └── fonts/                Bundled TTF files (see below)
├── workflow/
│   ├── build-pdf.sh          Build script
│   ├── author-markdown.md    Claude instructions for authoring
│   └── build-pdf.md          Claude instructions for building
└── cache/                    Tectonic LaTeX package cache (gitignored)
```

**What's not included** (must be added before first use):
- On Linux: the binaries are macOS builds — replace with Linux builds

---

## Setup

### 1. Detect Your Architecture

```bash
uname -m
```

| Output | Machine | Binary suffix |
|--------|---------|--------------|
| `arm64` | Apple Silicon Mac | `arm64` |
| `x86_64` | Intel Mac or Linux x86_64 | `x86_64` |
: Architecture detection

---

### 2. Add Binaries

The `bin/` directory needs `pandoc-<arch>` and `tectonic-<arch>`. Choose one method:

#### Download fresh

**Pandoc** — [github.com/jgm/pandoc/releases/latest](https://github.com/jgm/pandoc/releases/latest)

Download the standalone binary for your platform (no installer), then:

```bash
# macOS arm64 example
unzip pandoc-*-arm64-macOS.zip
cp pandoc-*/bin/pandoc .claude/skills/markset/bin/pandoc-arm64
chmod +x .claude/skills/markset/bin/pandoc-arm64
```

**Tectonic** — [github.com/tectonic-typesetting/tectonic/releases/latest](https://github.com/tectonic-typesetting/tectonic/releases/latest)

Download the single-binary release for your platform, then:

```bash
# macOS arm64 example
tar -xzf tectonic-*-aarch64-apple-darwin.tar.gz
cp tectonic .claude/skills/markset/bin/tectonic-arm64
chmod +x .claude/skills/markset/bin/tectonic-arm64
```
---

### 3. Verify Setup

```bash
.claude/skills/markset/workflow/build-pdf.sh <input>.md <output>.pdf
```

Expected:

```
PDF generated → <output>.pdf
```

---

## Usage

The skill is controlled entirely through Claude. Once SKILL.md is loaded, Claude knows how to:

1. **Author markdown** — following the design grammar in `templates/styleguide_md.md`
2. **Build a PDF** — by running `workflow/build-pdf.sh` with the correct paths

To use manually:

```bash
# Build any markdown file to PDF
.claude/skills/markset/workflow/build-pdf.sh <input.md> <output.pdf>

# Default output (no second arg): build/<filename>.pdf relative to input
.claude/skills/markset/workflow/build-pdf.sh docs/report.md
```

**First build:** Tectonic downloads LaTeX packages to `cache/` (~300 MB). Subsequent builds use the local cache and are fast.

---

## Troubleshooting

**macOS security warning — binary blocked by Gatekeeper**

```bash
xattr -d com.apple.quarantine .claude/skills/markset/bin/pandoc-arm64
xattr -d com.apple.quarantine .claude/skills/markset/bin/tectonic-arm64
```

**Binary not found or not executable**

```
Error: pandoc binary not found or not executable at .../bin/pandoc-arm64
```

```bash
ls -la .claude/skills/markset/bin/
chmod +x .claude/skills/markset/bin/pandoc-arm64
```

**Font not found during compilation**

Confirm all 8 TTF files are present in `templates/fonts/`. The build script passes the font directory directly to the LaTeX template — no system font install is needed or used.

**Template not found**

```
Error: LaTeX template not found at .../templates/template.tex
```

The skill folder is incomplete. Ensure `templates/template.tex` exists.

**Slow first build**

Expected. Tectonic fetches LaTeX packages on the first run (~300 MB into `cache/`). Transfer `cache/` from another machine to skip this.

---

## License

The build scripts and LaTeX template are MIT licensed.
Fonts (Inter, JetBrains Mono) are licensed under the SIL Open Font License 1.1.
Pandoc and Tectonic are their own projects with their own licenses — see their respective repositories.
