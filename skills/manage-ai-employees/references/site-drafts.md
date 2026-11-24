# Website drafts

> Reference for the `manage-ai-employees` skill. Load `operate-ai-employees` first.

## Live drafts of a website

A live draft is a private copy of one of the organization's websites that shows each change as it is made. The website itself never changes while a draft is open. Needs **Admin**. Drafts are switched off for the whole platform until the edit mode ships, so today every call answers `409 SITE_DRAFTS_OFF`.

```sh
dozaldevs open-site-draft "$SITE"   # 202 { draft_id, status: "starting" }
dozaldevs get-site-draft "$SITE"                # { draft_id, status, url } once running
dozaldevs close-site-draft "$SITE"  # stops it; the website is not changed
```

`open-site-draft` answers at once while the draft starts, and reopens a draft that is still running instead of starting a second one. Read `get-site-draft` until `status` is `running`: only then does `url` hold the private address, and the page answers there right away. `get-site-draft` answers `404` when the site has no open draft. A site that was taken down answers `409 SITE_DRAFT_SITE_OFFLINE`.

Each change to a draft is kept in its history, so it can be taken back and brought back:

```sh
dozaldevs list-site-draft-changes "$SITE"   # { changes: [{ seq, kind, label, undone, at }] }
dozaldevs undo-site-draft-change "$SITE"    # takes back the newest change still in place
dozaldevs redo-site-draft-change "$SITE"    # brings back the newest taken-back change
```

Undo answers `409 NOTHING_TO_UNDO` and redo `409 NOTHING_TO_REDO` when there is nothing to move, and either one answers `409 DRAFT_BUSY` while another change to the same draft is still being made. Neither changes the website itself.

A draft closes itself after a while with nobody looking at it, and always within a few hours. Close it yourself when you're done.
