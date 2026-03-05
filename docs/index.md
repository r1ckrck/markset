---
title: "Documentation System Overview"
author: "Arnesh Mandal"
version: "1.0"
date: "2026-01-30"
---

# Introduction

This project provides a complete markdown-to-PDF documentation system using Pandoc and Tectonic.
The system converts markdown files to professionally styled PDFs with consistent typography, colors, and layout.

# System Components

## Directory Structure

| Directory | Purpose |
|-----------|---------|
| `docs/` | Markdown source files |
| `build/` | Generated PDF output files |
| `templates/` | LaTeX template, Lua filters, style guides |
| `workflow/` | Build scripts and automation |

## Key Files

| File | Purpose |
|------|---------|
| `Makefile` | Build automation interface |
| `CLAUDE.md` | AI assistant instructions |
| `templates/template.tex` | LaTeX template implementing design spec |
| `templates/divs.lua` | Pandoc Lua filter for custom elements |
| `templates/styleguide_md.md` | Markdown writing rules |
| `templates/design_specification.md` | Complete visual design specification |
| `workflow/build-pdf.sh` | PDF generation script |

# Design System

## Typography

The system uses two professionally designed fonts:

- **Inter** — Body text and headings
- **JetBrains Mono** — Code blocks and inline code

## Color Palette

| Color | Hex | Usage |
|-------|-----|-------|
| Primary | `#181826` | Body text, headings |
| Secondary | `#ca9ee6` | Links, accents |
| Page background | `#faf7f2` | Warm cream |
| Code background | `#f0ece4` | Warm tan |

## Supported Elements

The system supports:

- Headings (H1, H2, H3)
- Code blocks with syntax highlighting
- Inline code
- Tables with captions
- Callouts (note, tip, warning, important)
- Images and figures
- Image placeholders
- Lists (ordered and unordered, max 2 levels)
- Block quotes
- Footnotes
- Links

# Usage

## Building PDFs

### Default Build

Build the default document:

```bash
make pdf
```

### Custom Input File

Build a specific markdown file:

```bash
make pdf IN=docs/my-document.md
```

### Custom Output Location

Specify both input and output:

```bash
make pdf IN=docs/my-document.md OUT=build/custom-name.pdf
```

### Clean Build Directory

Remove all generated PDFs:

```bash
make clean
```

## Writing Markdown

### Required Frontmatter

Every markdown file must start with YAML frontmatter containing title, author, version, and date fields.

### Style Guidelines

Follow these rules when writing markdown:

- One sentence per line
- Maximum heading depth: H3
- Use Title Case for headings
- All tables require captions
- Always specify language for code blocks
- No raw HTML or LaTeX
- Keep content concise

See `templates/styleguide_md.md` for complete writing guidelines.

## Using Callouts

Create callouts using fenced divs with types: note, tip, warning, or important.

## Adding Images

### Real Images

Use standard markdown image syntax with caption and path.

### Placeholders

Use image-placeholder divs when images are not yet available.
Include Figure title, Description, and optional Dimensions fields.

# System Requirements

## Fonts

The following fonts must be installed in `~/Library/Fonts/`:

- Inter (Regular, SemiBold, Italic, SemiBold Italic)
- JetBrains Mono (all weights and styles)

Download from:

- Inter: https://github.com/rsms/inter/releases/latest
- JetBrains Mono: https://github.com/JetBrains/JetBrainsMono/releases/latest

## Software Dependencies

The build system requires:

- Micromamba environment named `docs`
- Pandoc (installed in micromamba environment)
- Tectonic (installed in micromamba environment)

These dependencies are already configured in the micromamba environment.

# Technical Details

## Build Process

The build process:

1. Reads markdown source file
2. Processes custom elements with Lua filter
3. Applies LaTeX template
4. Generates PDF with Tectonic engine
5. Outputs to build directory

## Pandoc Configuration

The build system uses Pandoc with Tectonic engine, custom LaTeX template, and Lua filters for processing markdown elements.
Table of contents generation is enabled with depth 2, and sections are automatically numbered.

# Troubleshooting

## Font Not Found Errors

If you see "font cannot be found" errors:

1. Verify fonts are installed in `~/Library/Fonts/`
2. Check font file names match template expectations
3. Restart terminal session after font installation

## Build Failures

If PDF generation fails:

1. Check markdown file has required frontmatter
2. Verify markdown syntax follows style guide
3. Ensure input file path is correct
4. Check build output for specific error messages

## Deprecated Warning Messages

Warning messages about deprecated Pandoc options are expected.
The build script uses current syntax that may trigger warnings in newer Pandoc versions.
These warnings do not affect PDF generation.

# Reference

For detailed information, see:

- Writing rules: `templates/styleguide_md.md`
- Visual design: `templates/design_specification.md`
- Build script: `workflow/build-pdf.sh`
- LaTeX template: `templates/template.tex`
