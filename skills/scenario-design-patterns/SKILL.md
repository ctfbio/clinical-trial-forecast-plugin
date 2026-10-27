---
name: scenario-design-patterns
description: Design priced What-If scenarios, compare fresh or frozen results, and record a decision with a reproducible comparison memo.
---

Start with `ctf_list_capabilities`, `ctf_list_studies`, `ctf_list_scenario_configurations` and `ctf_list_budget_scenarios`. Confirm environment, grants and the baseline. Use `ctf_check_scenario_lock`; a locked or archived plan can be a comparator but cannot be edited.

Choose a template with an observable change:

- Fewer sites: archive the dropped sites; restate remaining caps/curve or move the end date. Remaining capacity must reach the target.
- One country: remove all its sites and country-specific costs, including applicable regulatory/import and monitoring lines; restate the FX basis.
- Lean launch: change priced rates or activation/timeline. A renamed clone alone is invalid.
- Sample-size change: change targets and scale per-participant vendor/drug quantities consistently.
- Vendor swap: change at least one service's vendor, its rates and transition costs.

Clone with overrides through `ctf_preview_changeset`. Show the diff and affected configurations; ask for the reason after the preview; apply with `ctf_apply_changeset` and the returned minted key. Read `ctf_get_change`. Preview/apply a recalculate block, enrollment then cost, and poll the resulting runs. Use `ctf_validate_configuration` and block on errors. SCENARIO_NOT_DIFFERENT and WHATIF_NO_PRICED_DIFFERENCE require explanation or correction. The preview's `risk` names the apply tool: tier 2 goes to `ctf_apply_changeset`; tier 3 (a protected plan or several plans, an archive, an organization catalog, a decision, a rollback or more than 50 items) goes only to `ctf_apply_high_impact_changeset` with the preview's `confirmation_code`, after the user agrees. A change into a locked plan previews with `can_apply` false and a SCENARIO_LOCKED finding; neither apply tool applies it, and the plan is unlocked in the web application.

Call `ctf_compare_study_scenarios` with the study and chosen scenario ids, breakdown dimensions and driver options. The omitted ids mean the study's configured planning scenarios. Use server totals and deltas; keep native currencies distinct and show the conversion basis when present. Never write a decision memo while any non-frozen option is stale, or while structurally different plans have identical totals without the service's no_priced_difference explanation. A frozen Approved Budget is a valid comparator; state its lock and computed_at.

Memo: identify the study, environment and decision; give a short recommendation supported by the comparison. Include an options table with type, native totals, deltas, FPI/LPI/LPO, enrollment duration, sites/countries, cap status, computed_at and stale/frozen state. Explain top cost drivers, timeline/enrollment/risk differences and data quality: benchmark versus negotiated rates, unresolved quotes, open findings and unchecked cost classes. Include the comparison checksum and returned app_url links.

Record the outcome and rationale with a scenario_decision item using another preview/apply cycle. `decision_status` is `adopted` (the default), `under_consideration` or `not_adopted`; approver and attestation keys belong to an adopted decision only. An off-platform approver requires their name, role, approval date and the recording user's explicit attestation. A named in-app approver (`approver_user_bk`) confirms in the web application. Without either, the connected user is the approver. A stale compared scenario is refused unless `acknowledge_stale` is true; say so in the memo when it is used. Protected changes may be approved by any org Admin, including the requester unless four-eyes is enabled. There is no connector approval tool.

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
