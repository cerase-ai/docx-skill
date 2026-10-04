# docx-skill

A Cerase skill that has the assistant produce an editable Word document
(`.docx`), or an OpenDocument (`.odt`) or Google Doc version of it, from
content the conversation provides. It is meant to be called by another skill:
`deck` when the person wants a document rather than slides, or
`source-to-artifact`. The caller passes the source (a markdown file in the
workspace or pasted text), the target format and the file name.

## What the assistant does

- **`docx`:** writes the content as a markdown file in the workspace (a YAML
  block with title, subtitle, author and date; headings, bullet and numbered
  lists, pipe tables, bold and italic) and converts it with
  `cerase-office-converter.convert_md_to_docx`. When the person provides a Word
  template in the workspace, it is passed as `reference_doc_path` and the
  document takes its fonts, colours and heading styles. The converter writes
  the file to `outputs/` in the workspace and returns its path, and the
  assistant attaches it with `[[attach: <path>]]`.
- **`odt`:** builds the `.docx` first, then converts it with
  `cerase-office-converter.convert_docx_to_odt`.
- **`gdoc`:** builds the `.docx`, uploads it to the person's Drive with
  `google-workspace.uploadFile` and `convertToGoogleFormat: true`, and gives
  the person the link of the new Google Doc. When the Google Workspace
  connector is not among the assistant's connectors, it says the
  organisation's administrator has to assign it and sends the `.docx`
  instead.

Style rules: the title in the YAML block, `H1` for sections, `H2` for
sub-sections, `H3` at most; at most seven bullets per group; a header row on
every table, with numeric columns right-aligned; no images, because the
converter receives only the markdown file; the language of the source or of
the person's chat. The assistant's container has no Python, LibreOffice or
pandoc, so the assistant never builds the file itself. It never pastes file
content or base64 in the chat, and it asks whether to split a source too large
for one document instead of dropping content.

## Requirements

- The `cerase-office-converter` connector for every format:
  `convert_md_to_docx` builds the `.docx`, and `convert_docx_to_odt` converts
  it for `odt`.
- A Google Workspace connector exposing `uploadFile` for `gdoc`.

## Files

| File | Purpose |
|---|---|
| `SKILL.md` | The instructions the assistant loads: `name` and `description` frontmatter, then the method per format. |
| `cerase.json` | Marketplace manifest: namespace `studio.guidance`, name `docx`, display name, description, licence. |
| `i18n.yaml` | Italian display name and description for the Marketplace; not sent to the assistant. |
| `LICENSE` | MIT licence text. |

## Installation

Published in the Cerase Marketplace as `studio.guidance/docx`
([marketplace page](https://marketplace.cerase.ai/en/p/studio.guidance/docx)).
A Cerase appliance does not attach it by default: an administrator installs it
from the Marketplace.

## License

MIT. See [LICENSE](LICENSE).
