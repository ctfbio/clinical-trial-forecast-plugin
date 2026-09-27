---
name: forecast-readiness-review
description: Check whether a study's scenario configurations in Clinical Trial Forecast are ready to forecast, find the missing design, site and cost inputs, and order the work, using reads only.
---

Use this workflow before a forecasting session or when a forecast looks incomplete. It reads and plans; it changes nothing.

1. Resolve the study with `ctf_list_studies`, or start from `ctf_get_tenant_data_state` to see which studies and configurations the organization holds.
2. `ctf_get_forecast_readiness` with the `study_id` returns one verdict per scenario configuration (`ready`, `ready_with_gaps` or `blocked`) with the unsatisfied blocking and advisory items. `detail_level: "standard"` adds the row count behind each item; `"full"` adds the whole checklist. The configuration ids are the items' `id`.
3. `ctf_get_forecast_gaps` reports incomplete forecast totals for the whole organization and has no study filter. With `group_by: "scenario_configuration"` each group is labelled with its protocol number and scenario name, so keep the groups of the study in question; `detail_level: "full"` adds their ids and links.
4. For a blocked or incomplete configuration, read what exists. `ctf_get_study_design` returns one `section` per call (`arms`, `epochs`, `visits` or `schedule_of_activities`). `ctf_get_scenario_design_layer` returns endpoints, interim analyses and intended uses. `ctf_list_sites` and `ctf_list_activity_costs` return the configuration's sites and unit costs. `ctf_list_activities`, `ctf_list_reference_data` and `ctf_list_reference_data` are the organization's catalogs that schedule entries and costs refer to.
5. Present an ordered task list: blocking items before advisory ones; arms, epochs and visits before schedule entries; schedule entries and sites before costs; complete inputs before a recalculation. Name the configuration each task belongs to and give its `app_url`.

Adding the missing records and recalculating the forecast happen in the web application, or with the write workflows on a connection granted write access.

## Tools and connection scope

This workflow reads from the four areas a connection starts with: `ctf.planning`, `ctf.studies`, `ctf.design` and `ctf.scenarios`. Read access is enough for every tool.

State today: every tool below is implemented and served, read only, by the staging connector.

| Tool                            | Area scope      | Access |
| ------------------------------- | --------------- | ------ |
| `ctf_get_tenant_data_state`     | `ctf.planning`  | read   |
| `ctf_get_forecast_readiness`    | `ctf.planning`  | read   |
| `ctf_get_forecast_gaps`         | `ctf.planning`  | read   |
| `ctf_list_studies`              | `ctf.studies`   | read   |
| `ctf_list_sites`                | `ctf.studies`   | read   |
| `ctf_list_activity_costs`       | `ctf.studies`   | read   |
| `ctf_list_reference_data`       | `ctf.studies`   | read   |
| `ctf_list_reference_data`       | `ctf.studies`   | read   |
| `ctf_get_study_design`          | `ctf.design`    | read   |
| `ctf_list_activities`           | `ctf.design`    | read   |
| `ctf_get_scenario_design_layer` | `ctf.scenarios` | read   |

## Links to the web application

Results carry `app_url` values. Each one opens a page of Clinical Trial Forecast for a person signed in to the organization this connection belongs to: a record's `app_url` shows that record, and a result's top-level `app_url` shows the whole result, or the nearest page when the web application has no view of it. `null` means there is no page.

- After answering, give links from those values so the person can open the same data in Clinical Trial Forecast. Label each link with what its page shows, from the table below, for example: Open the cumulative spend chart for ASCEND-1 (Budget and Latest Estimate).
- When the page is only the nearest one (readiness, enrollment), say so, for example: Open ASCEND-1's budget overview.
- Give at most five record links. For more records, give the result's top-level `app_url` instead. For a comparison, a curve, a schedule or a readiness answer, give the top-level `app_url`.
- A comparison link overlays the first 3 compared scenarios; say so when more were compared.
- In a terminal (Claude Code), write the full https:// address as plain text, never in backticks or a code block. Elsewhere a markdown link is fine.
- Use only `app_url` values from results. Never build or edit a link.
- If the page says the link belongs to a workspace the account is not a member of, the person is signed in to the web application with a different account than the one connected here.
- When the person asks for a chart, you may draw one from the result's series, and also give the `app_url` of the maintained view.

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
| `/o/{organization}/study/{key}?view=compare-costs` | Scenario cost comparison with drivers and currency basis |
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
| `/o/{organization}/changeset/{key}` | The changeset diff, affected plans and web approval |
| `/o/{organization}/forecast-run/{key}` | The recalculation run and its stored totals |
| `/o/{organization}/study-vendor/{key}` | The plan vendor scope |
| `/o/{organization}/study-system/{key}` | The plan system and fee lines |
| `/o/{organization}/scenario-decision/{key}` | The recorded scenario decision |
