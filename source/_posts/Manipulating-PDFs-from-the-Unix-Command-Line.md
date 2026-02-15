---
title: Manipulating PDFs from the Unix Command Line
date: 2025-08-27
categories:
- Data Science, Machine Learning, Multimedia Processing
tags:
- Reference
---

## Poppler Utilities

The [Poppler utilities](https://poppler.freedesktop.org/) provide many Unix command-line tools for manipulating PDFs. Install via:

- `sudo apt-get install poppler-utils` (Ubuntu)
- `sudo apk add poppler-utils` (Alpine)
- `brew install poppler` (macOS)

### Extract Text from a PDF

```bash
pdftotext input.pdf output.txt
```

### Merge PDFs Together

```bash
pdfunite file1.pdf file2.pdf merged.pdf
```

### Split (Extract) Pages from a PDF

This will split each page of `input.pdf` to `output_1.pdf`, `output_2.pdf`, etc.

```bash
pdfseparate input.pdf output_%d.pdf
```

## Convert SVG to PDF

### Using `inkscape`

Inkscape is a powerful SVG editor with command-line support.  

You can convert SVG to PDF like this:

```sh
inkscape input.svg --export-type=pdf --export-filename=output.pdf
```

### Using `rsvg-convert` (from the librsvg package)

```sh
rsvg-convert -f pdf -o output.pdf input.svg
```

