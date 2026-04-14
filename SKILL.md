---
name: markbook
description: Create MarkBook (.mkb) archives — portable Markdown eBooks. Use this skill when the user wants to create a .mkb file, package markdown into a MarkBook, convert documents to MarkBook format, or asks about the MarkBook/markbooks.org standard. Also use when seeing .mkb files or references to markbooks.org.
---

# MarkBook (.mkb) Format

A MarkBook is a **zip archive** with a `.mkb` extension containing Markdown files, images, and assets. Think of it as an eBook made of Markdown — portable, self-contained, and readable by any Markdown viewer that supports the format.

**Standard:** [markbooks.org](https://markbooks.org)

## Archive Structure

```
book.mkb (zip)
├── index.md              # Required. The entry point / frontpage.
├── chapters/             # Optional. Additional pages.
│   ├── 01-introduction.md
│   ├── 02-theory.md
│   └── 03-results.md
├── assets/               # Optional. Images, figures, diagrams.
│   ├── cover.png
│   └── figure-1.svg
└── styles/               # Reserved for future use.
```

Only `index.md` is required. Everything else is optional.

## index.md

The entry point. Typically contains the title, introduction, and table of contents. Can specify chapter order via frontmatter:

```markdown
---
title: My Book
author: Jane Smith
chapters: chapters/01-introduction.md, chapters/02-theory.md, chapters/03-results.md
---

# My Book

Welcome to this book. Use the sidebar to navigate chapters.
```

If no `chapters:` frontmatter is present, viewers auto-discover all `.md` files (excluding index.md) and sort them alphabetically by title.

## Chapters

Each chapter is a standalone Markdown file with its own headings. The chapter title is extracted from:
1. The first H1 heading in the file
2. Frontmatter `title:` field
3. Filename (with numbering stripped: `01-introduction.md` becomes "Introduction")

Prefix filenames with numbers for filesystem ordering: `01-`, `02-`, etc. The viewer sorts by title, but numbered prefixes keep files tidy in editors.

## Image Paths

Images resolve in this order:
1. **Relative to the current file** — `![](diagram.png)` in `chapters/02-theory.md` looks for `chapters/diagram.png`
2. **Relative to the archive root** — falls back to `diagram.png` at the root

The convention is to put images in `assets/` and reference them as:
```markdown
![Figure 1](assets/figure-1.png)
```
This works from both `index.md` (direct path) and chapter files (root fallback).

## Links Between Pages

Link to other pages in the archive using relative paths:
```markdown
See the [introduction](chapters/01-introduction.md) for background.
[Back to contents](../index.md)
```

Wikilink syntax also works: `[[chapters/01-introduction]]`

## Creating a MarkBook

### From existing files

If you have markdown files and images in a directory:

```bash
# From inside the content directory:
cd my-book/
zip -r "../My Book.mkb" index.md chapters/ assets/

# Or from outside:
zip -r "My Book.mkb" my-book/ -x "my-book/.DS_Store" "my-book/**/.DS_Store"
```

Make sure `index.md` is at the root of the archive, not nested in a subdirectory.

### From scratch

1. Create a working directory
2. Write `index.md` with title and overview
3. Create `chapters/` and write each chapter as a separate `.md` file
4. Put images in `assets/`
5. Zip into `.mkb`

```bash
mkdir -p my-book/{chapters,assets}

# Write content...

cd my-book
zip -r "../My Book.mkb" .
```

### Verify the structure

A valid MarkBook has `index.md` at the zip root:
```bash
unzip -l "My Book.mkb" | head -20
# Should show: index.md (not my-book/index.md)
```

## Supported Markdown

MarkBooks support GitHub Flavored Markdown plus:
- **Math:** `$inline$` and `$$display$$` LaTeX, also `\(...\)` and `\[...\]`
- **Tables:** GFM pipe tables with alignment
- **Code blocks:** fenced with language hints for syntax highlighting
- **HTML:** inline formatting tags (`<sub>`, `<sup>`, `<u>`, `<mark>`)
- **Wikilinks:** `[[page]]` and `![[image.png]]` (Obsidian-compatible)
- **Frontmatter:** YAML between `---` fences

## Tips

- Keep image files reasonable in size — MarkBooks are meant to be portable
- Use relative paths everywhere — absolute paths break when the archive moves
- Each chapter should work as a standalone document if extracted
- The `chapters:` frontmatter field gives you explicit ordering control; without it, titles are sorted alphabetically
- Test your MarkBook by opening it in Markant before distributing
