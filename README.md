# Microsoft Dynamics 365 Sales (microsoft-dynamics-365-sales)

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **Not from the company, and here with a question?** You are welcome here — we would rather be the
> front line and point you the right way than have a good report go nowhere. What this repository
> can answer is narrow, though, so it is worth knowing who you are actually looking for:
>
> - **A question about how the API works, an account, billing, or a bug in the service** — that is
>   the company's own support, not us. We profile this API; we do not operate it and cannot see
>   your account.
> - **A bug in an open-source project we only catalog** — file it on that project's own repository.
>   This has happened with a real and correct bug report that reached us instead of the people who
>   could fix it, which helped nobody.
> - **Anything about this listing itself** — the description, the tags, the rating, a missing or
>   wrong artifact — is ours. Open an issue here.
> - **Not sure, or something general about API Evangelist or APIs.io** — open an issue on the
>   [APIs.io Inbox](https://github.com/api-search/inbox) and we will route it.
>
> **This repository contains no software, and we will never ask you to download anything.** There is
> no build, release, installer, or binary here — only text and machine-readable API descriptions, so
> there is nothing here that can be "corrupt" or need "repairing". Any issue, comment, or email
> claiming otherwise and offering a download link is not from us and is hostile. Do not follow the
> link; it is a lure. Report it to GitHub and, if you like, tell us at
> [info@apievangelist.com](mailto:info@apievangelist.com) so we can take it down.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

Microsoft Dynamics 365 Sales is Microsoft's enterprise CRM application for managing leads, opportunities, accounts, contacts, sales pipelines, forecasts, and customer relationships, built on the Microsoft Dataverse platform and integrated with Microsoft 365, Teams, Copilot, and Power Platform. Developers programmatically interact with Dynamics 365 Sales data through the Dataverse Web API, an OData v4.0 RESTful interface that exposes every table, action, and function in a Sales environment. The Dataverse Web API uses OAuth 2.0 (Microsoft Entra ID) Bearer token authentication and is reached at a per-environment endpoint such as https://{org}.api.crm.dynamics.com/api/data/v9.2/.

**APIs.json:** [https://raw.githubusercontent.com/api-evangelist/microsoft-dynamics-365-sales/refs/heads/main/apis.yml](https://raw.githubusercontent.com/api-evangelist/microsoft-dynamics-365-sales/refs/heads/main/apis.yml)

## Scope

- **Type:** Index

## Tags

- CRM
- Sales
- Customer Relationship Management
- Dynamics 365
- Microsoft
- Dataverse
- OData
- Sales Automation

## Timestamps

- **Created:** 2026-05-11
- **Modified:** 2026-05-11

## APIs

### Microsoft Dataverse Web API (Dynamics 365 Sales)

OData v4.0 RESTful Web API for Microsoft Dataverse used to create, read, update, and delete Dynamics 365 Sales records (leads, opportunities, accounts, contacts, quotes, orders, invoices, products) and to invoke bound and unbound actions and functions. Authentication uses OAuth 2.0 Bearer tokens issued by Microsoft Entra ID; the endpoint is per-environment, e.g. https://{org}.api.crm.dynamics.com/api/data/v9.2/.

- **Human URL:** [https://learn.microsoft.com/en-us/power-apps/developer/data-platform/webapi/overview](https://learn.microsoft.com/en-us/power-apps/developer/data-platform/webapi/overview)
- **Base URL:** `https://{org}.api.crm.dynamics.com/api/data/v9.2`

#### Tags

- CRM
- Sales
- Dataverse
- OData
- Web API
- Dynamics 365

#### Properties

- [Documentation](https://learn.microsoft.com/en-us/power-apps/developer/data-platform/webapi/reference/about)
- [Overview](https://learn.microsoft.com/en-us/power-apps/developer/data-platform/webapi/overview)
- [Service  Documents](https://learn.microsoft.com/en-us/power-apps/developer/data-platform/webapi/web-api-service-documents)
- [Getting Started](https://learn.microsoft.com/en-us/power-apps/developer/data-platform/webapi/perform-operations-web-api)
- [Authentication](https://learn.microsoft.com/en-us/power-apps/developer/data-platform/authentication)
- [Postman Collection](collections/microsoft-dynamics-365-sales.postman_collection.json) — [Postman Collection 2.1](https://schema.getpostman.com/json/collection/v2.1.0/collection.json)
- [Open Collection](collections/microsoft-dynamics-365-sales.opencollection.json) — [Open Collection 1.0](https://schema.opencollection.com/opencollection/v1.0.0.json)

## Common Properties

- [GitHub Organization](https://github.com/MicrosoftDocs)
- [LinkedIn](https://www.linkedin.com/showcase/microsoft-dynamics-365-sales)
- [Website](https://www.microsoft.com/en-us/dynamics-365/products/sales)
- [Documentation](https://learn.microsoft.com/en-us/dynamics365/sales/)
- [Developer  Documentation](https://learn.microsoft.com/en-us/power-apps/developer/data-platform/webapi/overview)
- [Pricing](https://www.microsoft.com/en-us/dynamics-365/products/sales/pricing)
- [Sign Up](https://dynamics.microsoft.com/en-us/sales/overview/)

## Maintainers

**FN:** Kin Lane
**Email:** kin@apievangelist.com
