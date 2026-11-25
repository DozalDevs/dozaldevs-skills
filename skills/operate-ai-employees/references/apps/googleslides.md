<!-- Generated from the tools themselves; do not edit by hand. -->

# Google Slides

The actions your AI can run on Google Slides through DozalDevs. They are the same tools a Dozal uses, run the same way. Run one with `run_app_action` (MCP) or the run-app-action command, app `googleslides` and the action's slug below; `get_app_action` shows its inputs. A Viewer key runs only the actions that read.

| Action                   | Reads or changes | What it does                                           |
| ------------------------ | ---------------- | ------------------------------------------------------ |
| `CREATE_SLIDES_MARKDOWN` | changes          | Create a Google Slides presentation from Markdown text |
| `PRESENTATIONS_GET`      | reads            | Read a Google Slides presentation's slides and content |

## Notes

Written for Dozals, so an action is named by its slug in lowercase with dashes (`create-slides-markdown` is `CREATE_SLIDES_MARKDOWN`), and `--name` is the input `name`.

- Every Google Slides tool works on the Google Slides of the account your organization connected.
- `presentations-get` reads a presentation (its slides and text) by its id.
- `create-slides-markdown` builds a new presentation from a title and Markdown text; it is a write and may wait for a person's approval. It uses plain title and body layouts.
- A 403 means the connected person lacks access to that presentation; say so plainly instead of retrying.
