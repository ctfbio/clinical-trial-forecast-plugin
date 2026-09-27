# Clinical Trial Forecast plugin for Claude

Twelve readable workflows for Clinical Trial Forecast — organization records, study forecasting,
protocol extraction, scenario comparison, biosamples, workforce planning, public-trial research and
forecasting-session preparation, plus four that only read (a study overview, a scenario review, a
forecast-readiness review and a sample-testing review) — and a setup skill for connecting the bundled
remote MCP server. The workflows call the Clinical Trial Forecast connector; calculation methods stay
on the server.

- **Plugin identifier:** `clinical-trial-forecast` — the manifest name and the listing slug. It is
  stable: changing it would break every existing installation. The repository is named
  `clinical-trial-forecast-plugin`; that name is not the identifier.
- **Publisher:** CTFBio, LLC
- **License:** Apache-2.0 — see [`LICENSE`](LICENSE)
- **Support:** <https://app.clinicaltrialforecast.com/support> (moving to
  `help.clinicaltrialforecast.com` once that host is live)
- **Privacy policy:** <https://app.clinicaltrialforecast.com/privacy>

**The connector's source code is not part of this repository.** This repository carries the plugin
only: the manifest, the MCP entry, the workflow skills, the setup skill and the license. The remote
MCP server, its authorization server and its tool contracts are developed and maintained privately,
and no license is granted to them.

## Status

**This plugin is in beta.** It connects to the production Clinical Trial Forecast connector, which
serves read tools and previewed changes: every change is previewed first, and applying it asks for
your confirmation. A release carries only the workflows whose every tool the production connector
serves; the list at the end of this file names them, and the workflows held back until their tools
are served. Each skill's own table records the state of every tool it names.

Tools, responses and permissions may change between versions. Check results before relying on them
for a planning decision.

## Prerequisites and connection

The plugin requires Claude Code or Cowork, and an eligible Clinical Trial Forecast organization.
Installing it registers the bundled server; run `/mcp` to authenticate it. In claude.ai, the same
server is added as a custom connector at the address below, with no trailing slash, so a Claude Code
session on the same account sees one set of tools. Sign in to the application, select the intended
organization, and review the requested read and write permissions. Each connection is bound to that
organization; reconnect to select a different one. Step-by-step:
[`skills/setup/SKILL.md`](skills/setup/SKILL.md).

Consent also chooses which parts of the application a connection reaches. The catalog is divided
into nine areas — studies, study design, scenarios, forecasts, biosamples, protocols, public trial
data, workforce, and planning checks — and a connection is served only the areas that were checked.
A connection starts with studies, study design, scenarios and planning checks; forecasts and every
other area is requested by name and checked at consent. Each workflow names the areas it needs, so a
workflow whose area was not granted reports its operations as unavailable.

The [`.mcp.json`](.mcp.json) file names one HTTPS Streamable HTTP service at
`https://mcp.clinicaltrialforecast.com/mcp`. It contains no credentials, background shell commands
or additional service connections. Skills first check which tools are available; unavailable
operations are reported without claiming completion.

## Privacy and data scope

Workflows use study-level and aggregate operational information. They exclude participant
identifiers and patient data — **do not put protected health information into a tool argument.**
Claude reads user-provided documents; the connector receives only the structured values needed for
requested actions. Tool inputs and outputs are not sent to analytics or tracing. Public-trial reads
use stored source data and report available source dates. No language model runs inside the
connector: the reasoning stays in your Claude, on your plan.

For a failed write or an interrupted connection, read back the affected record before retrying;
preserve retry keys and reconcile conflicts.

## How this repository is produced

The tree here is built from a private source repository by a package builder, so a skill and the
connector's tool registry cannot drift apart unnoticed: the builder refuses to construct a tree when
a skill names a tool the registry does not declare, when the workflow inventory is incomplete, when
the MCP entry is not the canonical service, or when the license file is missing or incomplete.
Editing a file here rather than at its source would be overwritten by the next build.

The same source also builds a separate pre-release channel under a different identifier and
address, for testing. The two are never merged: the builder refuses an output that carries the
other channel's identifier or address, that pairs a name with the wrong address, or that uses
placeholder expansion in its configuration.

A skill-content change requires a version increment in the manifest. Before any release, the
identifier, the license, every workflow dependency, the source revision, the current plugin
validator result and actual install, update and remove behavior are verified on the selected
distribution route. Directory submission follows connector approval and the publication decision.

## Workflows in this build

This build carries the setup skill and 9 workflows whose every tool the production connector serves: `forecast-readiness-review`, `forecast-session-planning`, `protocol-schedule-ingestion`, `sample-testing-review`, `scenario-design-patterns`, `scenario-review`, `study-build`, `study-overview`, `ta-cost-library`.

Held back until the production connector serves every tool they name:

- `biosample-planning`: `ctf_get_biosample_plan`, `ctf_get_sample_quantity_map`, `ctf_upsert_biosample_plan`
- `protocol-to-study`: `ctf_accept_protocol_version`, `ctf_apply_protocol_to_scenario`, `ctf_list_protocol_documents`, `ctf_list_protocol_versions`, `ctf_stage_protocol_version`
- `public-trial-research`: `ctf_get_public_trial`, `ctf_resolve_public_entity`, `ctf_search_public_trials`
- `workforce-planning`: `ctf_archive_workforce_plan_item`, `ctf_compare_workforce_scenarios`, `ctf_get_workforce_plan`, `ctf_upsert_capex_line`, `ctf_upsert_workforce_allocation`, `ctf_upsert_workforce_cost_term`, `ctf_upsert_workforce_engagement`, `ctf_upsert_workforce_resource`
