---
name: Use the Dynamics 365 Sales MCP server for deal insight
description: >-
  Connect an MCP client to Microsoft's hosted Dynamics 365 Sales MCP server and use its 20
  insight tools to research a lead, assess qualification, draft outreach, and read
  opportunity health — knowing which tools exist, what they cost, and what they cannot do.
api: mcp/microsoft-dynamics-365-sales-mcp.yml
operations:
  - get_lead_research
  - get_lead_qualification_assessment
  - draft_outreach_email
  - get_account_research
  - get_competitor_research
  - get_engage_summary
  - get_opportunity_health
  - get_opportunity_top_risks
  - get_opportunity_deal_overview
  - get_customer_updates
generated: '2026-08-13'
method: generated
source: >-
  Tool names, parameters and return shapes transcribed from Microsoft Learn —
  https://learn.microsoft.com/en-us/dynamics365/sales/model-context-protocol-sales-overview
  and the three tool tables it links.
---

# Sales insight over MCP

## Connect

Two servers, and you almost always want both:

```json
{
  "servers": {
    "Sales-mcp-server": {
      "url": "https://agent365.svc.cloud.microsoft/mcp/environments/<EnvironmentID>/servers/msdyn_SalesMCPServer",
      "type": "http"
    },
    "Dataverse-mcp-server": {
      "url": "https://<OrgURL>/api/mcp",
      "type": "http"
    }
  },
  "inputs": []
}
```

`<EnvironmentID>` is your tenant's Dataverse environment GUID. A wrong or placeholder id
returns `HTTP 400 {"code":"EndpointInvalid"}` — that is the endpoint working, not an outage.

Prerequisites: admin permissions in Dynamics 365 Sales and in Copilot Studio; Dataverse MCP
access allowed for the environment when the client is not Copilot Studio; enough Copilot
Studio credits. Claude Desktop is not supported at time of writing.

## Know the surface split before you plan

**The Sales MCP server has no CRUD tools.** All 20 of its tools are read/insight/generate
actions. If a task needs to create a lead or update an opportunity, route it to the
Dataverse MCP server or to the Web API — not to this one. See
`mcp/microsoft-dynamics-365-sales-tool-crosswalk.yml`: zero of the 20 tools map to a REST
operation.

## Qualifying a lead

1. `get_lead_research(msdyn_LeadId)` — professional background, decision-making authority,
   preferences.
2. `get_account_research(msdyn_EntityId)` — financials, company overview, strategic goals,
   recent news. Takes a **guid**, not a string id, unlike the lead tools.
3. `get_competitor_research(msdyn_EntityId)` — competitive threats, market positioning,
   alternative suppliers.
4. `get_engage_summary(msdyn_LeadId)` — engagement patterns and touchpoint effectiveness.
5. `get_lead_qualification_assessment(LeadId)` — evaluation against the target customer
   profile, qualification ranking, overall assessment.
6. `draft_outreach_email(LeadId)` — returns a subject and body. **Review before sending.**
   This tool composes; it does not send.

Note the parameter-name inconsistency in Microsoft's own tool set: `msdyn_LeadId` for
research and engagement, plain `LeadId` for assessment and email drafting. Pass what each
tool declares.

## Working an opportunity

- `get_opportunity_deal_overview(OpportunityId)` — the comprehensive read; start here.
- `get_key_opportunity_insights` / `get_key_opportunity_signals` /
  `get_key_opportunity_stakeholders` — objective and status, recent developments, decision
  makers.
- `get_opportunity_health(OpportunityId)` — scored against MEDDPICC (Metrics, Economic
  Buyer, Decision Process, Decision Criteria, Paper Process, Identify Pain, Champion,
  Competition).
- `get_opportunity_pain_points_and_needs`, `get_opportunity_top_risks` — the risk criteria
  are configured by the tenant admin, so the output is environment-specific.

These depend on the Sales Opportunity Agent being configured and turned on. If it is off,
expect empty results rather than an error.

## Daily catch-up

- `get_customer_updates` — no parameters. Leads, accounts and opportunities the user owns
  that changed since their last login.
- `get_sales_record_summary(RecordId)`, `get_sales_lead_catchup(LeadId)`,
  `get_sales_account_catchup(AccountId)`, `get_sales_opportunity_catchup(OpportunityId)` —
  summaries restricted to the fields an admin configured. If a field you expect is missing,
  it is a configuration choice, not a bug.

## Documents

- `get_sharepoint_search_results(Query)` — documents matching the query from the configured
  SharePoint sites.
- `get_answer_from_sharepoint_documents(Query)` — a generated answer over those documents.

Both are scoped to admin-configured SharePoint sites and reach outside Dataverse entirely.

## Cost

Every tool call consumes Copilot Studio credits. Eighteen tools bill as "Text and generative
AI tools (basic)"; `get_sales_record_summary` and `get_answer_from_sharepoint_documents`
bill as "Generative answer", the more expensive tier. Learn states plainly that Copilot
credits are not charged against Dynamics 365 Sales licences — they are a separate meter.
Budget accordingly before running a tool across a whole pipeline.
