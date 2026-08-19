---
name: Capture and advance a Dynamics 365 Sales lead
description: >-
  Create a lead in Dynamics 365 Sales, read it back, update it as it develops, and open the
  opportunity that follows it — using the Dataverse Web API with the correct OData headers,
  duplicate-safe write pattern, and 429 back-off.
api: openapi/microsoft-dynamics-365-sales-leads-api-openapi.yml
operations:
  - leads_create
  - leads_get
  - leads_list
  - leads_update
  - opportunities_create
  - opportunities_get
generated: '2026-08-13'
method: generated
source: >-
  Grounded in the operationIds in openapi/ and the request semantics documented at
  https://learn.microsoft.com/en-us/power-apps/developer/data-platform/webapi/compose-http-requests-handle-errors
---

# Capture and advance a lead

## Before you call anything

- Base URL is per environment: `https://{org}.api.crm.dynamics.com/api/data/v9.2`. Resolve
  `{org}` from Power Apps → Settings → Developer resources → Web API endpoint, or from the
  Power Platform discovery service. Never guess it.
- Every request carries:
  `Authorization: Bearer <Entra ID token>`, `Accept: application/json`,
  `OData-MaxVersion: 4.0`, `OData-Version: 4.0`, `If-None-Match: null`.
- Requests with a body add `Content-Type: application/json`.

## 1. Create the lead — `leads_create`

`POST /leads`

There is **no idempotency key**. If you POST and the response is lost, retrying creates a
second lead. Two safe options, in order of preference:

1. Generate the GUID yourself and `PATCH /leads(<your-guid>)` with header `If-None-Match: *`.
   That is an upsert that refuses to update an existing record — a repeat returns
   `412 Precondition Failed` instead of duplicating.
2. If you must POST, send `MSCRM.SuppressDuplicateDetection: false` to opt into Dataverse
   duplicate-detection rules, then reconcile with `leads_list` and a `$filter` on a natural
   key before retrying.

Add `Prefer: return=representation` if you need the created record back — otherwise the
response is `204 No Content` and you only get the id in the `OData-EntityId` header.

## 2. Read it back — `leads_get`

`GET /leads(<leadid>)?$select=...`

Always send `$select`. Without it Dataverse returns every non-null column. Keep the
`@odata.etag` from the response — it is the concurrency token for step 3.

## 3. Work the list — `leads_list`

`GET /leads?$select=...&$filter=...&$orderby=...&$top=...`

Set `Prefer: odata.maxpagesize=<n>` and follow `@odata.nextLink` until it is absent.
Standard tables cap at 5,000 records per page. Do not build your own offsets.

## 4. Update as the lead develops — `leads_update`

`PATCH /leads(<leadid>)` with `If-Match: <the etag you kept>`.

- `412 Precondition Failed` means someone else changed the record. Re-read with `leads_get`,
  decide whether the write is still correct, then retry with the fresh etag. It is **not** a
  transient error — do not blind-retry.
- `If-Match: *` prevents the PATCH from creating a record; it returns `404` when the lead is
  gone.

## 5. Open the opportunity — `opportunities_create`

`POST /opportunities`, then `opportunities_get` to confirm.

Microsoft's documented sales process qualifies a lead *into* an opportunity via the Qualify
message rather than by creating an opportunity directly. This skill covers the direct
create; if the environment runs the standard qualification flow, prefer the Qualify action
on the lead so the platform maintains the originating-lead link, and use this step only when
qualification is not in play.

## Errors you must handle

| Status | Meaning | Do |
| --- | --- | --- |
| 400 | Invalid argument, over-long URL or OData segment | Fix the request. Never retry unchanged. |
| 401 | Token bad or expired | Get a fresh Entra ID token, retry once. |
| 403 | Missing privilege | Stop. Escalate to an admin — retrying will not help. |
| 404 | Record absent (or upsert-create prevented by `If-Match: *`) | Stop or create. |
| 412 | Concurrency mismatch or duplicate key | Re-read, re-decide, retry with new etag. |
| 429 | Service protection limit | Sleep for the `Retry-After` seconds, then retry. |
| 503 | Service unavailable | Back off, check https://status.cloud.microsoft/ |

The error body is the OData envelope `{"error":{"code":"…","message":"…"}}`. `error.code` is
frequently empty and is not the HTTP status — branch on the status. Send
`Prefer: odata.include-annotations="*"` when you need
`@Microsoft.PowerApps.CDS.HelpLink` and the plug-in trace text.

## Throughput

6,000 requests and 1,200 seconds of execution time per user per web server in a 300-second
sliding window; 52 concurrent requests. Prefer many small parallel requests over large
batches — batching trades the request-count limit for the execution-time limit.
