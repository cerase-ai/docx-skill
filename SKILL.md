---
name: docx
description: "Creates an editable Word document (.docx), or an OpenDocument (.odt) or Google Doc version of it, from content the conversation provides. Delegated to by `source-to-artifact` or `deck` when the format asked for is a document."
---
# DOCX — Word documents

You write the document as markdown in your workspace, and the office converter turns it into Word. Your container runs no Python and no LibreOffice, so you never build the file yourself: the converter does.

## Inputs

From the caller skill, or from the person:
- **content**: a markdown file in the workspace, or text from the conversation
- **format**: `docx` (Word), `odt` (LibreOffice) or `gdoc` (Google Doc)
- **file name**: such as `report-q3.docx`

## Step 1 — write the markdown

Write `<name>.md` in the workspace with your write tool. The converter reads this shape:

- A YAML block on the first lines becomes the title block:
  ```
  ---
  title: Quarterly report for Northwind Traders
  subtitle: Third quarter 2026
  author: Ada Rossi
  date: 2026-10-05
  ---
  ```
- `#` for a section, `##` for a sub-section, `###` at most.
- `- ` for bullets, `1. ` for numbered lists.
- Pipe tables with a header row; a numeric column right-aligned with `|---:|` under its header.
- `**bold**` and `*italic*`.
- No images: the converter receives this one file, so an image path inside it is not found.

## Step 2 — convert

```
call_recipe("cerase-office-converter.convert_md_to_docx", {"path": "<name>.md", "output_filename": "<name>.docx"})
```

It answers `{path, filename, size_bytes}`: the document is in your workspace at `path`, which is `outputs/<name>.docx`.

When the person gives you a Word template (a .docx in the workspace), add `"reference_doc_path": "<template>.docx"`: the document then takes the template's fonts, colours and heading styles.

## Other formats

- **odt**: convert the .docx you just made: `call_recipe("cerase-office-converter.convert_docx_to_odt", {"path": "outputs/<name>.docx", "output_filename": "<name>.odt"})`
- **gdoc** (Google Doc): make the .docx, then upload it to the person's Drive converted to a Google Doc, with the upload call below, and only when the person asked for a Google file: the upload puts the content in their Drive. The answer carries the new file's `Link:`; give the person that link. If the Google Workspace connector is not among your connectors, say in their language that a Google Doc needs that connector, which the organisation's admin assigns, and send the .docx instead.
- **PDF**: that is the `pdf-writer` skill.

The upload that makes the Google Doc:

```
call_recipe("google-workspace.uploadFile", {"localPath": "outputs/<name>.docx", "name": "<title>", "mimeType": "application/vnd.openxmlformats-officedocument.wordprocessingml.document", "convertToGoogleFormat": true})
```

These calls are the complete set. Do not invent others.

## Deliver

Attach the file: `[[attach: outputs/<name>.docx]]`. Never paste its content or any base64 in the chat.

## Style rules

- Headings hierarchy: H1 = section, H2 = sub-section, H3 at most. The title is in the YAML block.
- Bullet lists: at most 7 items per group. Split into sub-headings when longer.
- Tables: a header row always; numeric columns right-aligned.
- Language: the same as the source content or the person's chat.

## Don't

- Don't write Python, or call `libreoffice`, `soffice` or `pandoc` from bash: none of them is in your container.
- Don't silently drop content: if the source is too large for one document, ask in their language whether to split it, for example one document per chapter.
