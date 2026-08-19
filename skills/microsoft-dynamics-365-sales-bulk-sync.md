---
name: Bulk read and sync Dynamics 365 Sales data without tripping service protection
description: >-
  Page every account, contact and opportunity out of a Dynamics 365 Sales environment, keep
  it synchronised with change tracking, and stay inside Dataverse service protection limits
  by obeying Retry-After instead of guessing a rate.
api: openapi/microsoft-dynamics-365-sales-accounts-api-openapi.yml
operations:
  - accounts_list
  - contacts_list
  - opportunities_list
  - leads_list
  - batch_execute
generated: '2026-08-13'
method: generated
source: >-
  operationIds from openapi/; limits and guidance from
  https://learn.microsoft.com/en-us/power-apps/developer/data-platform/api-limits
---

# Bulk read and keep in sync

## The three limits you are working against

Enforced **per authenticated user, per web server**, and most environments have more than
one web server — a trial environment has exactly one:

| Measure | Limit | Window |
| --- | --- | --- |
| Requests | 6,000 | 300s sliding |
| Combined execution time | 1,200s (20 min) | 300s sliding |
| Concurrent requests | 52 or higher | instantaneous |

Concurrency errors return immediately; the two windowed limits let you overshoot before they
bite.

## Rule one: let the server set the pace

Do not calculate a request rate. Microsoft's own guidance is to start low, ramp gradually,
and then take `Retry-After` as the authority. `x-ms-ratelimit-burst-remaining-xrm-requests`
and `x-ms-ratelimit-time-remaining-xrm-requests` are documented as **debug-only** and reset
when you land on a different server, so never drive a scheduler from them.

On `429`:

```
sleep(int(response.headers["Retry-After"]))
retry the same request
```

If you keep pushing, the duration lengthens deliberately. Slowing down shortens it.

## Rule two: many small parallel requests beat big batches

`batch_execute` (`POST /$batch`) accepts up to 1,000 operations, but a batch accrues against
the **execution-time** limit rather than the request-count limit, and larger batches make
`0x80072321` more likely, not less. Learn recommends starting at a batch size of 10 and
raising concurrency instead. With the Web API's small JSON payloads, network latency is
rarely the bottleneck.

Keep parallelism below 52. In .NET, set `ParallelOptions.MaxDegreeOfParallelism` explicitly
rather than letting it default to the core count.

## Rule three: page properly

For each of `accounts_list`, `contacts_list`, `leads_list`, `opportunities_list`:

```
GET /accounts?$select=accountid,name,accountnumber,revenue,modifiedon
Prefer: odata.maxpagesize=500
```

Follow `@odata.nextLink` until it is absent. Do not synthesise `$skip` offsets. The cap is
5,000 records per page on standard tables, 500 on elastic tables.

Always send `$select` — omitting it returns every non-null column on every row, which is the
fastest way to burn the execution-time limit.

For counts, `$count` plus the `Microsoft.Dynamics.CRM.totalrecordcount` and
`totalrecordcountlimitexceeded` annotations tell you whether the true total exceeded the
page cap.

## Rule four: sync with change tracking, not full re-reads

Send `Prefer: odata.track-changes` on the initial full read, keep the delta token, and pull
only what changed afterwards. This is the pull-based alternative to registering a webhook,
and it is the right choice for any agent that cannot host a public HTTPS endpoint. See
`asyncapi/microsoft-dynamics-365-sales-webhooks.yml` for the push-based options.

Cache `Microsoft.Dynamics.CRM.globalmetadataversion` — when it changes, refresh any cached
table definitions.

## Watch the URL length

FetchXml and long `$filter` expressions hit the 32 KB URL cap (`0x80060888`, "Invalid URI:
The Uri string is too long"), and any single OData segment over 260 characters returns
`400 Bad Request - Invalid URL`. Move long queries into a `$batch` body, where the cap rises
to 64 KB, and use parameter aliases —
`/MyApi(MyParameter=@alias)?@alias='longvalue'` — to keep segments short.

## What does not count against you

Plug-ins and custom workflow activities run in the isolated sandbox service and do not use
the public endpoints, so their requests are exempt. Their **execution time** is still added
to the request that triggered them, so a write that fires heavy synchronous logic is
expensive even though it is one request.

Entitlement limits (the per-licence daily API allocation) are evaluated **separately** from
service protection, and batching does not avoid them.
