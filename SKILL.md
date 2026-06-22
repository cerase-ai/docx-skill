---
slug: docx
description: "Crea un documento Word (.docx) editabile, o variante ODT / Google Doc, partendo da contenuto strutturato. Delegata da `source-to-artifact` o `deck` quando il formato richiesto è documento."
is_core: true
---
# DOCX — Word document creation

Create a `.docx` Word document (or .odt / Google Doc variant) from structured content. This skill is invoked by `source-to-artifact` or `deck` (Path C) when the user wants a document, not slides.

## Inputs

You get from the caller skill:
- **source content**: workspace path of a markdown file or pasted prose
- **target format**: one of `docx` (Word native), `odt` (LibreOffice), `gdoc` (Google Doc)
- **filename**: e.g. `report-q3.docx`

## Backend per format

### `docx` (native, fast)

Use python-docx in the workspace. Pattern (in your code interpretation):

```python
from docx import Document
doc = Document()
doc.add_heading('<title>', level=1)
doc.add_paragraph('<intro>')
doc.add_heading('<section>', level=2)
doc.add_paragraph('<body>')
# Bullet list:
for item in items:
    doc.add_paragraph(item, style='List Bullet')
# Table:
table = doc.add_table(rows=1, cols=len(headers))
table.style = 'Light List Accent 1'
for row in rows: ...
doc.save('<filename>.docx')
```

Write the file to the workspace. Attach to reply.

### `odt` (cross-format conversion via cerase-office-converter)

First create the `.docx` natively (see above), then convert:

```
call_recipe("cerase-office-converter.convert_docx_to_odt", {input_b64: <base64 of .docx>})
```

Returns `{filename, size_bytes, contents_base64}`. Decode + write `.odt` to workspace.

### `gdoc` (Google Doc via google-workspace MCP)

Requires google-workspace MCP installed + tenant credentials (CONNECT-1 connector). Pattern:

```
call_recipe("google-workspace.docs_create", {
  title: "<filename without ext>",
  markdown_content: <full markdown source>,
})
```

Returns `{doc_id, doc_url}`. Surface the URL to the user.

If google-workspace is not installed/credenzialed, tell the user politely: "per esportare in Google Doc serve il connector Google Workspace — l'admin lo deve abilitare". Fall back to `.docx`.

## Style rules

- Headings hierarchy: H1 = title, H2 = section, H3 = sub-section. Avoid going deeper.
- Bullet lists: ≤ 7 items per group (cognitive limit). Split into sub-headings if longer.
- Tables: header row styled differently; first column left-aligned, numeric columns right-aligned.
- No remote images (browsers/Word/LibreOffice handle them inconsistently).
- Language: same as the source content / user's chat language.

## Don't

- Don't bash subprocess to call libreoffice yourself. Use `cerase-office-converter` recipes — they handle the headless flag, profile dir, output encoding correctly.
- Don't return raw bytes in the chat — always write to workspace + attach as file.
- Don't silently drop content: if the source is too large for a single document, propose splitting ("vuoi che lo divido in 3 documenti per capitoli?").
