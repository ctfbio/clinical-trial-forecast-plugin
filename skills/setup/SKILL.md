---
name: setup
description: Connect the Clinical Trial Forecast MCP server this plugin bundles, in Claude Code or in claude.ai, choose the organization and areas, and check what is available before using a workflow.
---

# Connecting the Clinical Trial Forecast connector

This plugin bundles one remote MCP server, declared in `.mcp.json` under the key `ctf` at
`https://mcp.clinicaltrialforecast.com/mcp`; Claude Code names the server's tools after that key.
That address is the service this build of the plugin connects to; publication of this plugin does
not by itself establish that a service is answering there. The steps below are what a connection
requires.

## Before connecting

1. A Claude surface that can reach a remote MCP server: Claude Code or Cowork through this plugin,
   or claude.ai (web, desktop or mobile) through a custom connector.
2. A Clinical Trial Forecast account, and membership of an organization eligible to use the
   connector. Eligibility is the same as the web application's; where a connection is refused for
   it, the refusal says so. Create the account, the organization and its subscription at
   https://app.clinicaltrialforecast.com/claude, which also shows when everything is ready.
3. Accept the current legal notice yourself in the web application. An earlier acceptance does not
   cover a newer notice. The connector checks the current requirement during connection setup and
   before every authenticated request, including an existing connection.
4. Nothing else. The plugin carries no credentials, no API key field, no background command and no
   second service connection. There is nothing to paste into a configuration file.

## Connect in Claude Code

1. Installing the plugin registers the `ctf` server; there is no separate server to add.
2. Run `/mcp`, select the `ctf` server and authenticate. The client opens a browser, and the
   organization and area choices below follow.

## Connect in claude.ai

1. Add a custom connector (Settings, Connectors, Add custom connector) with the address
   `https://mcp.clinicaltrialforecast.com/mcp`, exactly as written, with no trailing slash. It is the
   address this plugin registers, so a Claude Code session signed in to the same account sees one
   set of these tools rather than two.
2. On Team and Enterprise plans an Owner adds the connector for the organization first; members then
   connect to it. On individual plans the member adds it.
3. Connect, then enable the connector in the conversation that uses it.
4. A custom connector brings the tools and their descriptions. The workflow skills in this plugin
   reach a claude.ai conversation only when the plugin itself is added to the account; each tool
   result is complete JSON either way.

## Sign in and consent

1. Sign in to Clinical Trial Forecast. Sign-in happens on the product's own pages; the plugin and
   the client never see the password.
2. **Choose the organization.** A connection is bound to exactly one organization and stays bound to
   it. To work in a different one, disconnect and connect again, then choose that organization.
3. **Review the areas.** Consent lists the parts of the application the connection may reach as
   separate checkboxes — studies, study design, scenarios, forecasts, vendors, biosamples, protocols, public
   trial data, workforce, and planning checks — and any of them can be cleared before approving. A
   connection starts with studies, study design, scenarios, planning checks, forecasts and vendors; every other area is
   added only when it is requested and checked. With write access, additional granted areas can be marked **changes only** when their read
   tools would exceed the surface budget. Their changeset kinds remain available. The six default
   areas fit ceiling set B. An oversized read-only selection is refused with the measured numbers.
4. **Review read versus write.** Read and write are separate grants. A connection granted read only
   can open records and cannot change them.
5. Approve. The client stores the resulting authorization; the plugin stores nothing.

## After connecting

- Ask for the tools the connection can see before assuming an operation exists. A workflow whose
  area was not granted reports its operations as unavailable rather than producing a result that did
  not happen.
- A good first question is what the organization holds: its studies, their scenario configurations
  and its budget scenarios. The `study-overview` workflow answers it with reads only.
- Each workflow skill lists the area scopes it needs and the state of every tool it names. Read that
  list when an operation is missing: the cause is either an ungranted area or a tool that is
  declared and not yet enabled.
- A refusal always says what happened and what to do. "Not found" is also the answer for an object
  that belongs to a different organization, so a missing record can mean the connection is bound to
  the wrong organization.

## When the legal notice changes

If the connector asks for current legal acceptance, open the web link in its refusal, review the
notice, select the required checkboxes and choose **Agree and Continue** yourself. The plugin cannot
accept for you. After acceptance, retry your request. An existing connection can continue without
reconnecting; reconnect only if your sign-in expired or the client requires it. If initial connection
setup was interrupted, follow the authorization page's return instructions and connect again.

An unavailable legal-status check does not grant access. Wait and retry if the service reports that
verification is temporarily unavailable.

## Data this connection carries

- Study-level and aggregate operational records: studies, sites, budget scenarios, forecasts,
  biosample plans, workforce plans, and stored public clinical-trial reference data.
- **No protected health information and no participant-level data.** Do not put participant
  identifiers, medical records or any individual's health information into a tool argument. The
  connector is not a place for them and will not store them.
- Claude reads documents you provide; only the structured values needed for a requested action are
  sent to the connector. Tool arguments and results are not sent to analytics or tracing systems.
- No language model runs inside the connector. The reasoning stays in your Claude, on your plan.
- Clearing or erasing workspace content happens only in the web application, by an organization
  Admin; the connector has no tool for either. Clearing the planning data (optionally with the
  stored files and the AI chat history) and the separate erasure of all workspace content act on the
  web application's live content only. CTFBio cannot erase copies already held outside it, such as
  results in your own AI client's conversations or memory and files you downloaded. Retention by
  service providers and in backups is separate from both.

## Beta

This connector is in beta. Tools, responses and permissions may change between versions, and results
should be checked before they are relied on for a planning decision.

## Changing or removing the connection

- **Different organization:** disconnect the `ctf` server (or the claude.ai connector), connect
  again, choose the other organization.
- **Different areas:** the granted areas are fixed for the life of a connection. Reconnect and check
  the areas you need.
- **Remove:** disconnect the `ctf` server in the client, then uninstall the plugin; in claude.ai,
  remove the custom connector. Removing the plugin alone does not end an
  authorization the client still holds. Removing the connection ends its authorization; results it
  already returned stay in your client until you delete them there.
- **While the workspace is being cleared:** during an organization Admin's clearing or erasure of
  workspace content, the connector refuses every request with "The workspace is being cleared by an
  organization Admin." The connection is not revoked. Retry once the cleanup has finished; reconnect
  only if your client asks you to sign in again.

## Study configuration and environment

The server key is `ctf` on both channels. Read the environment line before preparing changes. The default selection is studies, scenarios, planning, design, forecasts and vendors. Write access is offered only after the deployment has admitted it. An area labelled **changes only** grants its changeset kinds while omitting its read tools from this connection's listing.

Every write uses preview, a visible diff, the user's change reason and the preview's minted key on apply. A protected change opens the web approval page; an org Admin may approve their own change unless the organization's four-eyes setting requires another Admin.

A custom connector supplies tools without these skills. In claude.ai, install the private plugin through Customize → Plugins; a private GitHub marketplace requires the Claude GitHub App. Connect from the plugin's Connectors tab. Team/Enterprise Owners can set the plugin to Installed by default. The generated claude.ai skills bundle offers the same served workflows when installed as skills. Prompts are optional client features; skills remain the primary guidance on claude.ai.

## Links to the web application

Results carry `app_url` values. Each one opens a page of Clinical Trial Forecast for a person signed in to the organization this connection belongs to: a record's `app_url` shows that record, and a result's top-level `app_url` shows the whole result, or the nearest page when the web application has no view of it. `null` means there is no page.

- After answering, give an **Open in CTFBio** link from the returned app_url so the person can open the same data. Label each link with what its page shows, from the table below, for example: Open the cumulative spend chart for ASCEND-1 (Budget and Latest Estimate).
- When the page is only the nearest one (readiness, enrollment), say so, for example: Open ASCEND-1's budget overview.
- Give at most five record links. For more records, give the result's top-level `app_url` instead. For a comparison, a curve, a schedule or a readiness answer, give the top-level `app_url`.
- A budget comparison link opens Budgets › Compare with up to eight returned scenarios preselected; the spend overlay shows up to 3.
- In a terminal (Claude Code), write the full https:// address as plain text, never in backticks or a code block. Elsewhere a markdown link is fine.
- Use returned `view_links` for related screens: whiteboard, schedule-costs (cost and quantity), vendors, systems and team. Label these Open in CTFBio followed by the view name. Keep the returned org context and query parameters intact.
- Use only `app_url` values from results. Never build or edit a link.
- If the page says the link belongs to a workspace the account is not a member of, the person is signed in to the web application with a different account than the one connected here.
- When the person asks for a chart, you may draw one from the result's series, and also give the `app_url` of the maintained view.

### Current legal notice

If access is refused because the current legal notice needs acceptance, stop the workflow and give the person the web link supplied in the refusal. They must review and explicitly accept the notice themselves. Do not accept for them or retry protected operations until they have done so. After acceptance, retry the request; an existing connection can continue, with reconnection needed only if sign-in expired or the client requires it. If verification is unavailable, report that access remains blocked and retry later.

### What each link opens

| Link | What the page shows |
| --- | --- |
| `/o/{organization}` | Home: headline numbers, attention alerts, cumulative spend and per-study summaries |
| `/o/{organization}?view=capabilities` | Available tools and web capabilities |
| `/o/{organization}?view=trials` | Home, scrolled to the per-study rollups |
| `/o/{organization}?view=gaps` | Home, with the forecast-gaps alert open |
| `/o/{organization}?view=team` | Team: headcount, utilization and partner spend |
| `/o/{organization}/table/{grid}` | That organization-wide grid |
| `/o/{organization}/billing` | The organization's subscription state and what an administrator can do about it |
| `/o/{organization}/study/{key}` | The study's budget overview: Budget, Latest Estimate and Actual spend curves, milestones and breakdowns |
| `/o/{organization}/study/{key}?view=compare&scenarios={key},{key}` | The study's cumulative spend chart with the compared scenarios overlaid, and the difference table |
| `/o/{organization}/study/{key}?view=compare-costs` | Budgets › Compare with the compared scenarios selected, drivers and currency basis |
| `/o/{organization}/study/{key}?view=compare-vendors` | Scenario vendor cost comparison |
| `/o/{organization}/study/{key}?view=compare-systems` | Scenario system cost comparison |
| `/o/{organization}/study/{key}?view=decisions` | Scenario decisions and recorded approvals |
| `/o/{organization}/study/{key}?view=budget-cap` | Budget cap status for compared plans |
| `/o/{organization}/study/{key}?view=readiness` | The study's budget overview (the web application has no readiness page) |
| `/o/{organization}/study/{key}?view=sites` | The study's sites grid |
| `/o/{organization}/study/{key}?view=partners` | The study's partners grid |
| `/o/{organization}/study/{key}?view=milestones` | The study's milestones grid |
| `/o/{organization}/study/{key}?view=design` | The study's schedule-of-activities grid |
| `/o/{organization}/configuration/{key}` | The plan's budget overview and its milestones |
| `/o/{organization}/configuration/{key}?view=design` | The plan's study design page: endpoints, interim timeline, intended-use coverage and the sample testing schedule chart |
| `/o/{organization}/configuration/{key}?view=schedule-costs` | The plan schedule of activities in cost and quantity view |
| `/o/{organization}/configuration/{key}?view=systems` | The plan's system roles and integration gaps |
| `/o/{organization}/configuration/{key}?view=vendors` | The plan vendors and contract status |
| `/o/{organization}/configuration/{key}?view=rules` | The plan calculation rules and consumers |
| `/o/{organization}/configuration/{key}?view=validation` | Configuration validation findings |
| `/o/{organization}/configuration/{key}?view=decisions` | Scenario decisions and attested approvals |
| `/o/{organization}/configuration/{key}?view=canvas` | The plan's systems map |
| `/o/{organization}/configuration/{key}?view=costs` | The plan's activity costs grid |
| `/o/{organization}/configuration/{key}?view=testing` | The plan's testing policies grid |
| `/o/{organization}/configuration/{key}?view=sites` | The study's sites grid |
| `/o/{organization}/scenario/{key}` | A per-study plan's budget overview, or Home under a company-wide plan |
| `/o/{organization}/scenario/{key}?view=company` | The company plan by department, program and account |
| `/o/{organization}/scenario/{key}?view=planner` | The resource planner under that plan |
| `/o/{organization}/scenario/{key}?view=team` | Team under that plan |
| `/o/{organization}/scenario/{key}?view=whiteboard` | The scenario whiteboard with study plans, vendors and samples |
| `/o/{organization}/site/{key}` | The study's sites grid, at that site |
| `/o/{organization}/partner/{key}` | The study's partners grid, at that partner |
| `/o/{organization}/milestone/{key}` | The plan's milestones |
| `/o/{organization}/activity/{key}` | The activities grid, at that activity |
| `/o/{organization}/activity-cost/{key}` | The plan's activity costs grid, at that cost |
| `/o/{organization}/budget-category/{key}` | The budget categories grid, at that category |
| `/o/{organization}/reimbursement-type/{key}` | The reimbursement types grid, at that type |
| `/o/{organization}/organization/{key}` | The modeled organizations grid, at that organization |
| `/o/{organization}/arm/{key}` | The study arms grid, at that arm |
| `/o/{organization}/epoch/{key}` | The study epochs grid, at that epoch |
| `/o/{organization}/visit/{key}` | The study visits grid, at that visit |
| `/o/{organization}/schedule-activity/{key}` | The schedule-of-activities grid, at that entry |
| `/o/{organization}/workforce-resource/{key}` | The exact workforce resource record |
| `/o/{organization}/workforce-engagement/{key}` | The exact workforce engagement record |
| `/o/{organization}/workforce-cost-term/{key}` | The exact workforce cost term record |
| `/o/{organization}/workforce-allocation/{key}` | The exact workforce allocation record |
| `/o/{organization}/protocol-version/{key}` | The exact protocol version record |
| `/o/{organization}/biosample-plan/{key}` | The exact biosample plan record |
| `/o/{organization}/kit/{key}` | The exact kit record |
| `/o/{organization}/kit-configuration/{key}` | The exact kit configuration record |
| `/o/{organization}/consent-template/{key}` | The exact consent template record |
| `/o/{organization}/testing-policy/{key}` | The exact testing policy record |
| `/o/{organization}/partner-rate-card/{key}` | The exact partner rate card record |
| `/o/{organization}/partner-capacity/{key}` | The exact partner capacity record |
| `/o/{organization}/capex-line/{key}` | The exact capex line record |
| `/o/{organization}/changeset/{key}` | The changeset diff, affected plans and web approval |
| `/o/{organization}/forecast-run/{key}` | The recalculation run and its stored totals |
| `/o/{organization}/study-vendor/{key}` | The plan vendor scope |
| `/o/{organization}/study-system/{key}` | The plan system and fee lines |
| `/o/{organization}/scenario-decision/{key}` | The recorded scenario decision |
