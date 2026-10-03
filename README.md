# docx-skill

A Cerase skill that has the assistant produce an editable Word document
(`.docx`), or an OpenDocument (`.odt`) or Google Doc variant, from structured
content. It is meant to be called by another skill: `deck` when the person
wants a document rather than slides, or `source-to-artifact`. The caller
passes the source (a markdown file in the workspace or pasted text), the target
format and the file name.

## What the assistant does

- **`docx`:** writes a script with `python-docx` (headings, paragraphs, bullet
  lists, tables), saves the file in the workspace and attaches it to the reply.
- **`odt`:** builds the `.docx` first, then converts it with
  `cerase-office-converter.convert_docx_to_odt`.
- **`gdoc`:** calls `google-workspace.docs_create` with the title and the
  markdown, and returns the link. When the Google Workspace connector is not
  installed or connected, it says an administrator has to enable it and
  produces a `.docx` instead.

Style rules: `H1` for the title, `H2` for sections, `H3` for sub-sections and
nothing deeper; at most seven bullets per group; a styled table header with
numeric columns right-aligned; no remote images; the language of the source.
The assistant never runs LibreOffice itself, never pastes file bytes in the
chat, and proposes splitting a source too large for one document instead of
dropping content.

## Requirements

- Python with `python-docx` wherever the assistant runs code, for the `docx`
  path and as the first step of `odt`.
- The `cerase-office-converter` connector for `odt`.
- A Google Workspace connector exposing `docs_create` for `gdoc`.

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
