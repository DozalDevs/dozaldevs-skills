<!-- Generated from the tools themselves; do not edit by hand. -->

# Google Drive

The actions your AI can run on Google Drive through DozalDevs. They are the same tools a Dozal uses, run the same way. Run one with `run_app_action` (MCP) or the run-app-action command, app `googledrive` and the action's slug below; `get_app_action` shows its inputs. A Viewer key runs only the actions that read.

| Action                         | Reads or changes | What it does                                                                          |
| ------------------------------ | ---------------- | ------------------------------------------------------------------------------------- |
| `ADD_PARENT`                   | changes          | Add a parent folder to a Google Drive file                                            |
| `CREATE_FOLDER`                | changes          | Create a folder in Google Drive                                                       |
| `DOWNLOAD_FILE`                | reads            | Download the content of a Google Drive file by its id                                 |
| `DOWNLOAD_FILE_OPERATION`      | reads            | Download the content of a Google Drive file, optionally in a given format or revision |
| `EXPORT_GOOGLE_WORKSPACE_FILE` | reads            | Export a Google Doc, Sheet or Slides file as PDF, text or another format              |
| `FIND_FILE`                    | reads            | Search Google Drive for files by a query, folder or name                              |
| `FIND_FOLDER`                  | reads            | Search Google Drive for folders by name, parent folder or date                        |
| `GET_DOCUMENT`                 | reads            | Read a Google Docs document's structured content by its id, found through Drive       |
| `GET_FILE_METADATA`            | reads            | Read the details of one Drive file or folder by its id                                |
| `GET_SPREADSHEET_VALUES`       | reads            | Read the cell values of one range of a Google Sheet, through Drive                    |
| `INSERT_CHILD`                 | changes          | Put an existing Google Drive file into a folder                                       |
| `LIST_CHILDREN_V2`             | reads            | List what is inside one Google Drive folder                                           |
| `LIST_FILES`                   | reads            | List Google Drive files, optionally narrowed by a query, folder or shared drive       |
| `MOVE_FILE`                    | changes          | Move a Google Drive file to another folder                                            |
| `UPDATE_FILE_PUT`              | changes          | Change a Google Drive file's name, description or other details, or move it           |

## Notes

Written for Dozals, so an action is named by its slug in lowercase with dashes (`add-parent` is `ADD_PARENT`), and `--name` is the input `name`.

- Every Google Drive tool works on the Drive of the Google account your organization connected. Files are identified by id, never by name: find the id first with `find-file`, `find-folder` or `list-files`, then use it. `q` takes Drive's own search syntax (`name contains 'invoice' and trashed = false`).
- `list-children-v2` lists one folder; `get-file-metadata` reads a file's details and link; `get-document` reads a Google Doc, `get-spreadsheet-values` a range of a Google Sheet.
- `download-file` and `export-google-workspace-file` return a file's content (export a Google Doc, Sheet or Slides file to PDF or text with a MIME type).
- `create-folder`, `move-file`, `update-file-put`, `insert-child` and `add-parent` change Drive; they are writes and may wait for a person's approval. To move a file pass both the folder to add and the folder to remove.
- Read the next page of a listing with the page token from the previous answer.
- A 403 means the connected person lacks access to that file; say so plainly instead of retrying.
