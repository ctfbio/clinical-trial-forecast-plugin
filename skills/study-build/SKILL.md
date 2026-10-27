---
name: study-build
description: Build or amend a study plan, its schedule, costs, vendors, systems and calculation rules through previewed changesets, then validate and recalculate it.
---

Start with `ctf_list_capabilities`. State the connected environment and organization. Stop and name any required area without a write grant; a granted area marked “changes only” admits changes even when its list tools are absent.

Start from the scenario: pick one with `ctf_list_budget_scenarios` or create it as a `budget_scenario` item. Establish the study and configuration using `ctf_list_studies`, `ctf_get_study` and `ctf_list_scenario_configurations`; a `scenario_configuration` item places a study in the chosen scenario (`scenario_id` or `scenario_key`, with `study_id` or `study_key`). Use returned identifiers exactly. A new record uses a unique client_key within the draft; never invent an existing identifier. Suggested scenario name: `<study short> <type> <letter>: <one-line change>`.

Build in dependency order: scenario; study and partners; configuration; arms, epochs, visits and schedule entries; activities and costs; vendors and systems; calculation rules. Read `ctf_get_study_design`, `ctf_list_activities`, `ctf_list_activity_costs`, `ctf_list_study_vendors`, `ctf_list_study_systems` and `ctf_list_calculation_rules` when listed. Use `ctf_list_reference_data` for closed vocabularies and sourced cost benchmarks. Codes come only from the protocol or the user, with provenance, never from memory.

For each batch, or one complete build:

1. Call `ctf_preview_changeset` with the explicit environment. Limits are 200 items per part, 10,000 CSV characters per part and 2,000 items per draft. Show the diff, affected plans, locks, findings and approval requirement.
2. After the user has seen the diff, ask for their change reason (10–500 characters) and agreement to apply it.
3. Call `ctf_apply_changeset` with the returned changeset_id, fingerprint and minted idempotency_key, unchanged, and the user's reason. A pending approval goes to the supplied web link. Any organization Admin may approve; self-approval is allowed unless the organization's four-eyes setting requires another Admin. There is no connector approval tool. The preview's `risk` names the apply tool: tier 2 goes to `ctf_apply_changeset`; tier 3 (a protected, locked or several plans, an archive, an organization catalog, a decision, a rollback or more than 50 items) goes only to `ctf_apply_high_impact_changeset` with the preview's `confirmation_code`, after the user agrees.
4. Read `ctf_get_change`, verify the resulting records, and give **Open in CTFBio** links from app_url and view_links (whiteboard or schedule-costs). On “outcome unknown”, reconcile by applying the same changeset with the same key, fingerprint and reason; never create a replacement draft just to retry.

Use `ctf_validate_configuration`. Block on error findings; present warnings and information with their scenario-type severity. Recalculate using another preview/apply cycle with a recalculate block, enrollment before cost, then poll `ctf_get_change`. Never report a current total while is_stale is true. A frozen total is valid: state its lock and computation time.

Reconcile each arm's activity × encounter marks to the protocol: every footnote is a condition, sourced probability, or explicit “not modelled” choice; offsets increase and fit windows; arm differences match the protocol; every active site has costs and nonzero capacity; vendor scope is contracted and costed; the independent total matches the checked classes. Disclose classes not independently checked.

What-If and Budget are editable types; Draft is a status. Forecast changes need a clear reason. Prefer a clone or amendment for Baseline and Approved Budget; protected production changes require web approval, including changes into or out of those types. Actual comes from integrations. Stop on locked or archived plans and name the holder. For a shared activity, offer a fork limited to this plan.

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
