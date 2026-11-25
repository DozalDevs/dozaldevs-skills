<!-- Generated from the tools themselves; do not edit by hand. -->

# OlaClick

The actions your AI can run on OlaClick through DozalDevs. They are the same tools a Dozal uses, run the same way. Run one with `run_app_action` (MCP) or the run-app-action command, app `olaclick` and the action's slug below; `get_app_action` shows its inputs. A Viewer key runs only the actions that read.

| Action                          | Reads or changes      | What it does                                                                                                                                                                                  |
| ------------------------------- | --------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `ADD_ORDER_ITEM`                | changes               | Add an item to an API-created order (unit prices IN, line totals computed server-side; free-form lines allowed)                                                                               |
| `APPLY_ORDER_DISCOUNT`          | changes               | Apply a discount to an OlaClick order (AMOUNT or PERCENTAGE; multiple discounts stack)                                                                                                        |
| `ASSIGN_RIDER`                  | changes               | Assign a rider to an order (external or OlaClick company user)                                                                                                                                |
| `BULK_UPSERT_PRODUCTS`          | changes               | Bulk upsert up to 500 products in ONE idempotent API call (PUT, not a loop; matches by id or creates)                                                                                         |
| `CANCEL_ORDER`                  | changes (cancel)      | Cancel an API-created order (fails if active payments exist)                                                                                                                                  |
| `CANCEL_ORDER_ITEM`             | changes (cancel)      | Cancel (soft-delete) an item from an API-created order                                                                                                                                        |
| `CANCEL_ORDER_PAYMENT`          | changes (cancel)      | Cancel a payment for an OlaClick order (required before order cancellation if payments exist)                                                                                                 |
| `CREATE_MENU_PRODUCT`           | changes               | Create a new menu product with 1-30 variants and optional image (prices are decimals, not minor units)                                                                                        |
| `CREATE_PRODUCT_VARIANT`        | changes               | ⚠️ Create a new product variant (NOT idempotent — retries can duplicate; original_price required)                                                                                             |
| `CREATE_WEBHOOK`                | changes               | Create a new webhook subscription for the OlaClick account                                                                                                                                    |
| `DELETE_WEBHOOK`                | changes (delete data) | Delete a webhook subscription (deliveries stop immediately)                                                                                                                                   |
| `GET_CLIENT`                    | reads                 | Get a single OlaClick client by ID                                                                                                                                                            |
| `GET_COMPANY`                   | reads                 | Get OlaClick company profile information (business hours, contact details, etc.)                                                                                                              |
| `GET_MENU`                      | reads                 | Fetch the complete OlaClick menu catalog (unpaginated, potentially large)                                                                                                                     |
| `GET_MENU_CATEGORY`             | reads                 | Fetch details for a single menu category by ID                                                                                                                                                |
| `GET_MENU_PRODUCT`              | reads                 | Fetch details for a single menu product by ID                                                                                                                                                 |
| `GET_ORDER`                     | reads                 | Fetch details for a single order by UUID or public ID (e.g., PE-1234567890)                                                                                                                   |
| `GET_ORDERS_BY_PAYMENT_METHODS` | reads                 | Get a report of orders grouped by payment method                                                                                                                                              |
| `GET_ORDERS_BY_TIPS`            | reads                 | Get a report of orders grouped by tip amount                                                                                                                                                  |
| `GET_PRODUCTS_SOLD`             | reads                 | Get a report of products sold across all orders                                                                                                                                               |
| `GET_STORE_STATUS`              | reads                 | Get store status across delivery marketplaces (Rappi, iFood, Didi, PedidosYa, Uber Eats, 99Food) — an unconfigured or unreachable provider reports UNKNOWN rather than failing the whole call |
| `GET_WEBHOOK`                   | reads                 | Fetch details for a single webhook subscription by ID                                                                                                                                         |
| `LIST_CLIENTS`                  | reads                 | List OlaClick clients with optional filtering by buying-frequency and recency segments                                                                                                        |
| `LIST_KITCHENS`                 | reads                 | List all kitchens configured for the restaurant                                                                                                                                               |
| `LIST_MENU_CATEGORIES`          | reads                 | List all menu categories                                                                                                                                                                      |
| `LIST_MENU_PRODUCTS`            | reads                 | List menu products with optional filters and pagination                                                                                                                                       |
| `LIST_ORDERS`                   | reads                 | List orders with filters for status, source, service type, payment status, dates, rider, and client                                                                                           |
| `LIST_PRODUCT_VARIANTS`         | reads                 | List all variants for a specific menu product                                                                                                                                                 |
| `LIST_PROMOTIONS`               | reads                 | List OlaClick promotions                                                                                                                                                                      |
| `LIST_USERS`                    | reads                 | List OlaClick users with optional filtering by role and active status                                                                                                                         |
| `LIST_WEBHOOKS`                 | reads                 | List all webhook subscriptions for the OlaClick account                                                                                                                                       |
| `REGISTER_ORDER_PAYMENT`        | changes (money)       | Register a payment for an OlaClick order (multiple payments per order allowed)                                                                                                                |
| `REMOVE_ORDER_DISCOUNT`         | changes               | Remove a discount from an OlaClick order (hard delete, not soft-cancel)                                                                                                                       |
| `UNASSIGN_RIDER`                | changes               | Unassign the current rider from an order                                                                                                                                                      |
| `UPDATE_DELIVERY_STATUS`        | changes               | Update delivery status for an order (auto-sets delivered_by: API)                                                                                                                             |
| `UPDATE_MENU_PRODUCT`           | changes               | Update an existing menu product (at least one field required; response omits variants)                                                                                                        |
| `UPDATE_ORDER_STATUS`           | changes               | Update an order status via forward-only transition (PENDING→PREPARING→READY→DELIVERED→FINALIZED)                                                                                              |
| `UPDATE_PRODUCT_VARIANT`        | changes               | Update an existing product variant (at least one field required)                                                                                                                              |
| `UPDATE_RIDER_POSITION`         | changes               | Update rider position for an order (intended ~every 30s during delivery; do not spam)                                                                                                         |
| `UPDATE_WEBHOOK`                | changes               | Update an existing webhook subscription                                                                                                                                                       |
| `VALIDATE_ENV`                  | reads                 | Validate that required OlaClick environment variables are set                                                                                                                                 |

## Notes

Written for Dozals, so an action is named by its slug in lowercase with dashes (`add-order-item` is `ADD_ORDER_ITEM`), and `--name` is the input `name`.

### Orders

- `get-order` takes the order's UUID or its public id (such as `PE-1234567890`).
- Order status only moves forward: PENDING, PREPARING, READY, DELIVERED, FINALIZED. `update-order-status` can't go back.
- To cancel an order that has payments, cancel each payment first with `cancel-order-payment`. `cancel-order` fails while an active payment exists. Cancels and payments may wait for the owner's approval.
- `add-order-item`, `cancel-order` and `cancel-order-item` work on orders created through the API. Give unit prices; line totals are computed for you.
- `apply-order-discount` takes `AMOUNT` or `PERCENTAGE`, and several discounts stack. `remove-order-discount` deletes the discount for good.
- `register-order-payment` can be called more than once per order. It may wait for the owner's approval.
- `update-rider-position` is meant for about every 30 seconds during a delivery. Don't call it faster.
- `update-delivery-status` marks `delivered_by` as the API.

### Menu

- `get-menu` returns the whole catalog in one answer, which can be large. Prefer `list-menu-categories`, `list-menu-products` or `get-menu-product` when you need less.
- In `create-menu-product`, prices are decimals, not minor units. A product takes 1 to 30 variants.
- `create-product-variant` isn't idempotent: calling it again can create a duplicate, and it needs `original_price`. Don't retry it blindly; list the variants first.
- `bulk-upsert-products` takes up to 500 products in one call and matches by id or creates. Use it instead of looping.
- `update-menu-product` needs at least one field, and its answer leaves out the variants.

### Other

- `get-store-status` covers the delivery marketplaces. A marketplace that isn't set up or can't be reached reports `UNKNOWN`; that doesn't fail the call.
- `delete-webhook` stops deliveries at once.
- A 403 on a menu change means the saved key lacks the menu write scope. Report it; the owner has to fix the key.
