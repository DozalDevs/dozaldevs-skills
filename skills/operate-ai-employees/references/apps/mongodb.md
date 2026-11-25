<!-- Generated from the tools themselves; do not edit by hand. -->

# MongoDB

The actions your AI can run on MongoDB through DozalDevs. They are the same tools a Dozal uses, run the same way. Run one with `run_app_action` (MCP) or the run-app-action command, app `mongodb` and the action's slug below; `get_app_action` shows its inputs. A Viewer key runs only the actions that read.

| Action                       | Reads or changes      | What it does                                                                                |
| ---------------------------- | --------------------- | ------------------------------------------------------------------------------------------- |
| `AGGREGATE_DOCUMENTS`        | changes               | Run an aggregation pipeline over a collection and return the resulting documents            |
| `DELETE_DOCUMENTS`           | changes (delete data) | Delete documents matching a filter from a MongoDB collection                                |
| `FIND_AND_REPLACE_DOCUMENTS` | changes               | Replace the first document matching a filter and return the before/after version            |
| `FIND_AND_UPDATE_DOCUMENTS`  | changes               | Update the first document matching a filter and return the before/after version             |
| `FIND_DOCUMENTS`             | reads                 | Find documents in a MongoDB collection, optionally filtered, projected, sorted, and limited |
| `INSERT_DOCUMENTS`           | changes               | Insert one or more documents into a MongoDB collection                                      |
| `UPDATE_DOCUMENTS`           | changes               | Update multiple documents in a collection matching a filter (updateMany)                    |

## Notes

Written for Dozals, so an action is named by its slug in lowercase with dashes (`aggregate-documents` is `AGGREGATE_DOCUMENTS`), and `--name` is the input `name`.

### Reading

- `find-documents` takes `--collection`, and optionally `--query` (a JSON filter, default `{}`), `--projection`, `--sort` and `--limit`. It returns `{documents}`. Always set `--limit` when you only need a sample.
- The database is the one in the saved connection unless you pass `--database`. Collection and database names may hold only letters, digits, `_`, `.` and `-`.
- `aggregate-documents` takes a pipeline as a JSON array of stages. Don't use `$out` or `$merge` unless the job says to write a collection; both write.

### Writing

- `insert-documents` adds documents. `update-documents` changes every document that matches the filter (`updateMany`). `find-and-update-documents` and `find-and-replace-documents` change only the first match and return the document before and after; `--return-updated` chooses which version you get back.
- `--upsert` creates a document when nothing matches. Leave it off unless you want that.
- `delete-documents` removes every document that matches the filter, can't be undone, and may wait for the owner's approval. Test the same filter with `find-documents` first.
- If the job names a connection, pass it with `--connection <name>`. Without it you get the default one.
