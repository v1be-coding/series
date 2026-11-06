# V1be Coding

A build series where each app goes from spec to working, deployed app in one agent run. These are the terms shared by every episode.

## Series

**Episode**:
One app in the series, with its own repo, spec, live app and portfolio page.
_Avoid_: Project, build

**Foundation**:
The series-wide decisions and shared infrastructure, built once before episode 0.
_Avoid_: Episode -1, groundwork, platform

**Default stack**:
The infrastructure every episode uses unless that episode's ADR records a reason to deviate.
_Avoid_: Paved road, boilerplate

**Template repo**:
The repo every episode repo is created from. It holds the shared CI, pre-commit hooks, guardrails and smoke-test harness.
_Avoid_: Starter, scaffold

**Stack**:
A folder in `v1be-infra` that Terraform plans and applies on its own, with its own state file.
_Avoid_: Layer, workspace

## Workflow

**Setup**:
The supervised phase before a run that prepares what the run can't touch: DNS, certificates and secrets. Claude drafts the change in a PR, the human merges it, runs the `org` stack in Windows Sandbox and approves the CI apply, and the human creates the third-party keys.
_Avoid_: Bootstrap, provisioning step

**Run**:
One execution of `/implement-spec` on a fresh branch. It writes code, including app resources in the repo's infrastructure code, and pushes. It never touches accounts, DNS or secrets.
_Avoid_: Build, session

**Branch environment**:
A temporary, complete deployment of one attempt's branch, with its own URL and seed data. It's destroyed when the branch closes.
_Avoid_: Preview, staging, stage

**Attempt**:
A run plus whatever spec fixes came before it. An episode gets at most three.
_Avoid_: Try, iteration

## Data

**Real data**:
Data from actual café operations. It exists only in café production and never in a public repo or demo.
_Avoid_: Production data, live data

**Seed data**:
Realistic fake data committed to a repo so that demos and branch environments look real without exposing real data.
_Avoid_: Mock data, test data, dummy data

## Café

**Café backend**:
The single production system of record for Collective Cafe's sales, menu and supplies. Collective POS owns it, and the other café episodes go through it.
_Avoid_: Café database, POS database
