<!-- Generated from the tools themselves; do not edit by hand. -->

# insideairbnb

The actions your AI can run on insideairbnb through DozalDevs. They are the same tools a Dozal uses, run the same way. Run one with `run_app_action` (MCP) or the run-app-action command, app `insideairbnb` and the action's slug below; `get_app_action` shows its inputs. A Viewer key runs only the actions that read.

| Action         | Reads or changes | What it does                                                                                           |
| -------------- | ---------------- | ------------------------------------------------------------------------------------------------------ |
| `LIST_CITIES`  | reads            | List the United States cities Inside Airbnb publishes listing snapshots for                            |
| `LIST_HOSTS`   | reads            | List the hosts with many listings in one city from its Inside Airbnb snapshot, biggest first           |
| `NEARBY_COUNT` | reads            | Count single-listing, under-booked entire homes near a property manager's own Inside Airbnb listings   |
| `NEARBY_HOMES` | reads            | Build a manager's list of nearby under-booked single-owner Airbnb homes from public Inside Airbnb data |
