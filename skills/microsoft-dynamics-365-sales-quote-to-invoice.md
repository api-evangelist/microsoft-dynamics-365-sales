---
name: Move a Dynamics 365 Sales deal from quote to invoice
description: >-
  Walk the documented Dynamics 365 Sales record chain — opportunity, quote, sales order,
  invoice — over the Dataverse Web API, binding each record to the previous one with OData
  navigation binds and handling the concurrency and rate-limit signals correctly.
api: openapi/microsoft-dynamics-365-sales-quotes-api-openapi.yml
operations:
  - opportunities_get
  - quotes_create
  - quotes_list
  - salesorders_create
  - salesorders_list
  - invoices_create
  - invoices_list
  - products_list
generated: '2026-08-13'
method: generated
source: >-
  operationIds from openapi/; the record chain from
  https://learn.microsoft.com/en-us/dynamics365/sales/nurture-sales-from-lead-order-sales
---

# Quote → order → invoice

Microsoft's published sales process is: **Qualify → Develop → Propose → Close → Fulfill**,
mapping to Lead → Opportunity → Quote → SalesOrder → Invoice. This skill covers the last
three transitions.

## Headers on every call

```
Authorization: Bearer <Microsoft Entra ID token>
Accept: application/json
OData-MaxVersion: 4.0
OData-Version: 4.0
If-None-Match: null
Content-Type: application/json      # bodies only
```

## 1. Confirm the deal — `opportunities_get`

`GET /opportunities(<opportunityid>)?$select=name,estimatedvalue,statecode`

Keep the `@odata.etag`.

## 2. Read the catalogue — `products_list`

`GET /products?$select=...&$filter=...`

Use `Prefer: odata.maxpagesize=<n>` and follow `@odata.nextLink`. Standard tables cap at
5,000 rows per page.

## 3. Propose — `quotes_create`

`POST /quotes`

Bind the quote to the deal and the customer with `@odata.bind` navigation properties rather
than raw id fields:

```json
{
  "name": "Q-2026-0001",
  "opportunityid@odata.bind": "/opportunities(<opportunityid>)",
  "customerid_account@odata.bind": "/accounts(<accountid>)"
}
```

Add `Prefer: return=representation` if you need the created record back; otherwise expect
`204 No Content` with the new id in `OData-EntityId`.

Repeat-safety: generate the quote GUID yourself and `PATCH /quotes(<guid>)` with
`If-None-Match: *` so a replayed request returns `412` instead of creating a second quote.
There is no idempotency key on this API.

## 4. Close — `salesorders_create`

`POST /salesorders`, binding back to the quote:

```json
{ "quoteid@odata.bind": "/quotes(<quoteid>)" }
```

Learn: *"When the customer agrees to the quote, an order is generated. The quote and
opportunity associated with the order are closed."* If the environment runs the standard
process, prefer the platform's Win/Convert actions so the state transitions on the quote and
opportunity happen as designed; use a direct create only where those actions are not in use.

Confirm with `salesorders_list` and a `$filter` on the quote id before creating a second one.

## 5. Fulfil — `invoices_create`

`POST /invoices`, binding to the order:

```json
{ "salesorderid@odata.bind": "/salesorders(<salesorderid>)" }
```

Confirm with `invoices_list`.

## Doing it in one round trip — `batch_execute`

`POST /$batch` (`openapi/microsoft-dynamics-365-sales-batch-api-openapi.yml`) accepts up to
1,000 operations as `multipart/mixed`, and inside a batch the URL cap rises from 32 KB to
64 KB. Microsoft's own guidance is to keep batches small — start at 10 — and raise
concurrency instead, because a batch accrues against the 1,200-second execution-time limit
rather than the 6,000-request limit.

## Failure handling

- `412 Precondition Failed` — the record moved under you (`ConcurrencyVersionMismatch`) or a
  matching key already exists (`DuplicateRecord`). Re-read, re-decide, retry with a fresh
  etag. Not transient.
- `429 Too Many Requests` — sleep exactly `Retry-After` seconds. The duration lengthens if
  you keep pushing. Do not compute your own backoff from
  `x-ms-ratelimit-*`; those headers are documented as debug-only.
- `403 Forbidden` — a privilege is missing on the table or column. Stop and escalate.
- Errors are the OData envelope, not RFC 9457 problem+json. `error.code` is often empty.
