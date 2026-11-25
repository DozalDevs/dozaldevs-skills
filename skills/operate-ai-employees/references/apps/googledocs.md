<!-- Generated from the tools themselves; do not edit by hand. -->

# Google Docs

The actions your AI can run on Google Docs through DozalDevs. They are the same tools a Dozal uses, run the same way. Run one with `run_app_action` (MCP) or the run-app-action command, app `googledocs` and the action's slug below; `get_app_action` shows its inputs. A Viewer key runs only the actions that read.

| Action                     | Reads or changes | What it does                                                                          |
| -------------------------- | ---------------- | ------------------------------------------------------------------------------------- |
| `CREATE_DOCUMENT`          | changes          | Create a new Google Docs document with a title and optional text                      |
| `CREATE_DOCUMENT_MARKDOWN` | changes          | Create a Google Docs document from Markdown text                                      |
| `CREATE_DRIVE_FILE`        | changes          | Create an item without content in Google Drive: an empty file, a folder or a shortcut |
| `GET_DOCUMENT_BY_ID`       | reads            | Read a Google Docs document's full structure by its id                                |
| `GET_DOCUMENT_END_INDEX`   | reads            | Find the position of the end of a Google Docs document                                |
| `GET_DOCUMENT_PLAINTEXT`   | reads            | Read a Google Docs document as plain text                                             |
| `INSERT_TEXT_ACTION`       | changes          | Insert text into a Google Docs document at a position or at the end                   |
| `REPLACE_ALL_TEXT`         | changes          | Replace every occurrence of some text in a Google Docs document                       |
| `SEARCH_DOCUMENTS`         | reads            | Search for Google Docs documents by name, date or owner                               |
| `UPDATE_DOCUMENT_MARKDOWN` | changes          | Replace the whole content of a Google Docs document with Markdown text                |

## Notes

Written for Dozals, so an action is named by its slug in lowercase with dashes (`create-document` is `CREATE_DOCUMENT`), and `--name` is the input `name`.

- Every Google Docs tool works on the Google Docs of the account your organization connected. Documents are identified by id: find it with `search-documents` (by name, owner or date) when you only have a title. Search only finds documents created through DozalDevs, so when a document you expect is not found, ask for its link instead of concluding it does not exist.
- To read: `get-document-plaintext` for the text, `get-document-by-id` for the full structure (headings, tables, styles), `get-document-end-index` for the position of the end.
- To write: `create-document` (title and optional text) or `create-document-markdown` (a formatted document from Markdown); `insert-text-action` adds text at a position or at the end; `replace-all-text` fills in or changes every occurrence of some text; `update-document-markdown` replaces the whole content. These are writes and may wait for a person's approval.
- A 403 means the connected person lacks access to that document; say so plainly instead of retrying.
