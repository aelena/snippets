# About

A simple code snippet that extracts all highlighted text from a pdf and any comment attached to any section of highlighted text. This could be then be pased to an LLM, or consolidated into some sort of personal knowledge database for researching topics. Might take into that direction using the pipe philosophy in Unix. For now this is just the snippet.

This produces a txt file with all the quotations as a hyphenated list.

Install this first 

```python
pip install pymupdf
```

Supports two parameters, the source pdf, of course and an optional parameter for the name of the output txt file, if you want to control that.

# the Src


```python

#!/usr/bin/env python3
# extract_highlights.py
# Usage: python extract_highlights.py input.pdf [output.txt]

import sys
import pathlib
import fitz  # PyMuPDF

def highlight_text_from_annot(page, annot):
    """Return the text covered by a highlight annotation on this page."""
    verts = getattr(annot, "vertices", None)
    if not verts:
        return ""
    words = page.get_text("words")  # list of (x0, y0, x1, y1, "word", block, line, word_no)
    if not words:
        return ""

    # Build rectangles for each quad in the highlight
    rects = []
    for i in range(0, len(verts), 4):
        quad = fitz.Quad(verts[i:i+4])
        rects.append(quad.rect)

    # For each rect, collect words that intersect it, keeping reading order
    snippet_parts = []
    for r in rects:
        selected = [w for w in words if fitz.Rect(w[:4]).intersects(r)]
        # sort by top-left (y, then x)
        selected.sort(key=lambda w: (w[1], w[0]))
        snippet_parts.append(" ".join(w[4] for w in selected))

    # Join across multiple quads (multi-line highlights)
    snippet = " ".join(p.strip() for p in snippet_parts if p.strip())
    # Normalize spacing
    snippet = " ".join(snippet.split())
    return snippet

def get_annot_comment(annot):
    """Return the attached comment (if any) for the annotation."""
    info = getattr(annot, "info", {}) or {}
    # PyMuPDF typically exposes 'content'; be tolerant of 'contents'
    comment = info.get("content") or info.get("contents") or ""
    comment = (comment or "").strip()
    # Normalize whitespace
    comment = " ".join(comment.split())
    return comment

def is_highlight(annot):
    """Detect highlight annotation robustly across PyMuPDF versions."""
    try:
        # annot.type is a tuple like (8, 'Highlight') in modern versions
        t = getattr(annot, "type", None)
        if isinstance(t, tuple) and len(t) >= 2:
            return t[1].lower() == "highlight"
    except Exception:
        pass
    # Fallback: subtype string
    try:
        return annot.info.get("type", "").lower() == "highlight"
    except Exception:
        return False

def extract_highlights(pdf_path: str, out_path: str):
    doc = fitz.open(pdf_path)
    meta_title = (doc.metadata or {}).get("title") or pathlib.Path(pdf_path).stem

    snippets = []
    for page in doc:
        annot = page.first_annot
        while annot:
            if is_highlight(annot):
                text = highlight_text_from_annot(page, annot)
                if text:
                    comment = get_annot_comment(annot)
                    if comment:
                        snippets.append(f"- {text} [{comment}]")
                    else:
                        snippets.append(f"- {text}")
            annot = annot.next

    # Write output
    with open(out_path, "w", encoding="utf-8") as f:
        f.write(f"{meta_title}\n\n")
        for s in snippets:
            f.write(s)
            f.write("\n\n")
    doc.close()

def main():
    if len(sys.argv) < 2:
        print("Usage: python extract_highlights.py input.pdf [output.txt]")
        sys.exit(1)
    pdf_path = sys.argv[1]
    out_path = sys.argv[2] if len(sys.argv) >= 3 else str(pathlib.Path(pdf_path).with_suffix(".highlights.txt"))
    extract_highlights(pdf_path, out_path)
    print(f"Done. Wrote: {out_path}")

if __name__ == "__main__":
    main()


```
