<!-- Generated from the tools themselves; do not edit by hand. -->

# DoorLoop

The actions your AI can run on DoorLoop through DozalDevs. They are the same tools a Dozal uses, run the same way. Run one with `run_app_action` (MCP) or the run-app-action command, app `doorloop` and the action's slug below; `get_app_action` shows its inputs. A Viewer key runs only the actions that read.

| Action                 | Reads or changes | What it does                                                                                         |
| ---------------------- | ---------------- | ---------------------------------------------------------------------------------------------------- |
| `CREATE_COMMUNICATION` | changes          | Create a communication log entry in DoorLoop (email, phone, or text)                                 |
| `CREATE_LEASE`         | changes          | Create a complete DoorLoop lease in one transactional call (property, units, tenants, deposit, rent) |
| `CREATE_LEASE_PAYMENT` | changes (money)  | Create a new DoorLoop lease payment                                                                  |
| `CREATE_OWNER`         | changes          | Create a new DoorLoop owner                                                                          |
| `CREATE_TASK`          | changes          | Create a new DoorLoop task (internal task, tenant/owner request, or work order)                      |
| `CREATE_TENANT`        | changes          | Create a new DoorLoop tenant                                                                         |
| `CREATE_VENDOR`        | changes          | Create a new DoorLoop vendor                                                                         |
| `GET_COMMUNICATION`    | reads            | Fetch details for a single DoorLoop communication by ID (includes bodyHtml)                          |
| `GET_FILE`             | reads            | Fetch details for a single DoorLoop file by ID (read-only)                                           |
| `GET_LEASE`            | reads            | Retrieve a single DoorLoop lease by ID                                                               |
| `GET_LEASE_PAYMENT`    | reads            | Retrieve a single DoorLoop lease payment by ID                                                       |
| `GET_OWNER`            | reads            | Fetch details for a single DoorLoop owner by ID                                                      |
| `GET_PROPERTY`         | reads            | Fetch details for a single DoorLoop property by ID                                                   |
| `GET_TASK`             | reads            | Fetch details for a single DoorLoop task by ID                                                       |
| `GET_TENANT`           | reads            | Fetch details for a single DoorLoop tenant by ID                                                     |
| `GET_UNIT`             | reads            | Retrieve a single DoorLoop unit by ID                                                                |
| `GET_VENDOR`           | reads            | Fetch details for a single DoorLoop vendor by ID                                                     |
| `LIST_ACCOUNTS`        | reads            | Retrieve the chart of accounts for the organization                                                  |
| `LIST_COMMUNICATIONS`  | reads            | List DoorLoop communications with optional filtering                                                 |
| `LIST_FILES`           | reads            | List DoorLoop files with optional filters (read-only)                                                |
| `LIST_LEASE_PAYMENTS`  | reads            | List DoorLoop lease payments with optional filters                                                   |
| `LIST_LEASES`          | reads            | List DoorLoop leases with optional filtering                                                         |
| `LIST_OWNERS`          | reads            | List DoorLoop owners with optional filtering                                                         |
| `LIST_PROPERTIES`      | reads            | List DoorLoop properties with optional filters                                                       |
| `LIST_TASKS`           | reads            | List DoorLoop tasks (maintenance requests) with optional filtering                                   |
| `LIST_TENANTS`         | reads            | List DoorLoop tenants with optional filtering                                                        |
| `LIST_UNITS`           | reads            | List DoorLoop units with optional filters                                                            |
| `LIST_VENDORS`         | reads            | List DoorLoop vendors with optional filters                                                          |
| `UPDATE_OWNER`         | changes          | Update an existing DoorLoop owner                                                                    |
| `UPDATE_TASK`          | changes          | Update an existing DoorLoop task                                                                     |
| `UPDATE_TENANT`        | changes          | Update an existing DoorLoop tenant                                                                   |
| `UPDATE_UNIT`          | changes          | Update an existing DoorLoop unit via a safe GET-then-PUT merge                                       |
| `UPDATE_VENDOR`        | changes          | Update an existing DoorLoop vendor                                                                   |
| `VALIDATE_ENV`         | reads            | Validate that required DoorLoop environment variables are set                                        |

## Notes

Written for Dozals, so an action is named by its slug in lowercase with dashes (`create-communication` is `CREATE_COMMUNICATION`), and `--name` is the input `name`.

### What you can do

- Most tools are lists and lookups. The list tools follow every page for you, so filter (by property, unit, status, dates or text) instead of pulling everything.
- You can't create or change a property, and you can't create a unit. You can update a unit's details with `update-unit`; it keeps what you don't set.
- `create-tenant` always makes a prospect tenant. Making someone a lease tenant happens through `create-lease` (`--tenant-ids` for existing tenants, or the `--new-tenant-*` flags for one new one).
- `create-lease` does the whole lease in one call: property, units, term (`Fixed` or `AtWill`; `Fixed` needs `--end`), tenants, deposit and rent. Pass JSON flags (`--units`, `--rent`, charges and credits) as one quoted JSON value.
- `create-lease-payment` records money received, and it may wait for the owner's approval. It needs `--amount-received`, `--payment-method`, `--lease`, `--deposit-to-account` and `--auto-apply-payment-on-charges`.
- Find account ids for deposits and charges with `list-accounts`.
- `list-tasks` is the maintenance-request list. `NOT_COMPLETED` is a filter value only; you can't set it on a task.
- Ids are 24-character hex strings. Copy them exactly from a previous result.
- A 404 means that id doesn't exist. Check the id before you try another. Any other error gives only the status, so report it with what you were doing.
