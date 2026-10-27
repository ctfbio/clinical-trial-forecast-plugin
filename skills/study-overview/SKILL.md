---
name: study-overview
description: Answer what the connected Clinical Trial Forecast organization holds (its studies, their scenario configurations, sites, partners and milestones) using reads only.
---

Use this workflow for questions such as "what studies are in my organization", "what does this study look like" or "which sites and partners does it have". It only reads, so it works on a read-only connection.

1. Start with `ctf_get_tenant_data_state`. It returns the organization's name, organization-wide counts and a first page of its studies, budget scenarios and scenario configurations with their ids. The counts cover the whole organization; the page does not, so follow `next_cursor` while `has_more` is true before treating the page as complete.
2. Use `ctf_list_studies` for the studies with protocol number, title and phase. Narrow with `phase`, `protocol_number_contains` or `title_contains` rather than paging through everything, and page before concluding that a study does not exist.
3. For one study, call `ctf_get_study` with `detail_level: "standard"`. Summary detail returns the study without its configurations; standard adds its scenario configurations with dates and enrollment and site targets. `ctf_list_scenario_configurations` with the `study_id` returns the same configurations as a list.
4. Sites, partners and milestones take either `study_id` or `scenario_configuration_id`, never both: `ctf_list_sites`, `ctf_list_study_partners` and `ctf_list_milestones`. Milestone type codes and categories come from `ctf_list_reference_data`.
5. `ctf_list_organizations` lists the partner, CRO, site and vendor organizations modeled under the connected one. `ctf_get_organization_profile` returns its stated goal, role and programs, and answers not found when no profile has been saved yet.
6. Before someone archives a study, configuration or site in the web application, `ctf_preview_archive_impact` lists what depends on it and which locked records sit above it. The preview reserves nothing and archives nothing.
7. Answer with the names, protocol numbers, phases, counts and dates the results carry, and give each record's `app_url`, which opens the related page of the web application. Say which lists were partial and why.

Use the ids exactly as the results return them. Creating or changing studies, sites, partners or milestones happens in the web application, or with the write workflows on a connection granted write access.

## Tools and connection scope

This workflow reads from three areas a connection starts with: `ctf.planning`, `ctf.studies` and `ctf.scenarios`. Read access is enough for every tool.

State today: every tool below is implemented and read only. This workflow is offered only where the connector serves every one of them.

| Tool                               | Area scope      | Access |
| ---------------------------------- | --------------- | ------ |
| `ctf_get_tenant_data_state`        | `ctf.planning`  | read   |
| `ctf_list_studies`                 | `ctf.studies`   | read   |
| `ctf_get_study`                    | `ctf.studies`   | read   |
| `ctf_list_sites`                   | `ctf.studies`   | read   |
| `ctf_list_study_partners`          | `ctf.studies`   | read   |
| `ctf_list_organizations`           | `ctf.studies`   | read   |
| `ctf_get_organization_profile`     | `ctf.studies`   | read   |
| `ctf_preview_archive_impact`       | `ctf.studies`   | read   |
| `ctf_list_scenario_configurations` | `ctf.scenarios` | read   |
| `ctf_list_milestones`              | `ctf.scenarios` | read   |
| `ctf_list_reference_data`          | `ctf.scenarios` | read   |

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
