<!--
Sync Impact Report
- Version: 1.4.0 → 1.5.0
- Modified sections:
  - Additional Constraints & Safety Requirements:
    - Added mandatory CivicTheme version verification before and after each
      per-version upgrade, with explicit STOP conditions when installed
      versions do not match the documented from/to versions for that step.
- Added principles:
  - (None – principles I–V unchanged from v1.4.0)
- Added sections:
  - (None)
- Removed sections:
  - None
- Templates reviewed (✅ aligned / ⚠ pending):
  - ✅ .specify/templates/plan-template.md (no structural changes required; plans may reference version verification checks where relevant)
  - ✅ .specify/templates/spec-template.md
  - ✅ .specify/templates/tasks-template.md
  - ✅ .specify/templates/checklist-template.md
  - ✅ .specify/templates/agent-file-template.md
  - ✅ Global CivicTheme upgrade documentation framework (source document and
    extracts in feature specifications)
  - ⚠ .specify/templates/commands/*.md (no command templates present in repository)
- Deferred TODOs:
  - None
-->

# CivicTheme Upgrade Assistant Constitution

## Core Principles

### I. Planning-First, Non-Destructive Documentation

- All CivicTheme upgrade work MUST start from a per-version specification derived
  from the shared CivicTheme upgrade documentation framework defined in this
  repository before any code or configuration changes are proposed or applied.
- The CivicTheme upgrade documentation framework and this constitution define
  planning and governance only and MUST NOT contain copy-pasteable destructive
  commands intended to run directly against production systems.
- For each CivicTheme upgrade, there MUST be a clear separation between:
  - the version-specific spec (analysis and plan),
  - and any execution or runbook documentation (step-by-step commands and checks).

Rationale: Keeping planning separate from execution makes upgrades reviewable,
repeatable and safe, and matches the shared CivicTheme upgrade documentation
framework.

### II. Safe Environments & Reproducibility

- All upgrade activities described in specs and tasks MUST assume execution in a
  non-production or otherwise safely isolated environment.
- Each per-version specification MUST explicitly document environment
  assumptions, following the environment pattern defined in the CivicTheme
  upgrade documentation framework.
- Execution/runbook documents MUST be reproducible from version-controlled
  specifications; no undocumented, ad-hoc manual steps are permitted as the sole
  source of truth.

Rationale: Explicit environment assumptions and reproducible runbooks minimise
upgrade risk and make AI- and human-led work auditable.

### III. Independently Testable User Journeys

- Feature specifications generated from `.specify/templates/spec-template.md`
  MUST express behaviour as independently testable user stories with explicit
  priorities (P1, P2, P3, …).
- Implementation plans and task lists generated from
  `.specify/templates/plan-template.md` and
  `.specify/templates/tasks-template.md` MUST map tasks and (when requested)
  tests back to specific user stories so that each story is independently
  implementable and verifiable.
- When tests are requested in a feature or upgrade specification, test tasks
  MUST be added before implementation tasks for the same story, and tests MUST
  be written and executed in a failing state before implementation proceeds.

Rationale: Prioritised, independently testable stories allow incremental,
verifiable delivery and make it easier for AI tooling to generate safe, scoped
changes.

### IV. Customisation Inventory & Impact Mapping

- CivicTheme-specific work MUST maintain a customisation inventory as described
  in the CivicTheme upgrade documentation framework for each per-version
  specification.
- AI assistants and human developers MUST treat the customisation inventory as
  the primary index of upgrade risk, referencing its IDs (for example C1, C2)
  in research, plans and tasks.
- Version analysis for each upgrade MUST cross-reference upstream CivicTheme
  changes (release notes, changelog entries and relevant issues) with the
  customisation inventory to identify high-risk areas.

Rationale: A shared inventory of customisations ensures upgrades preserve
project-specific behaviour and makes the impact of upstream changes explicit.

### V. Template-Driven Consistency & Versioning

- Templates under `.specify/templates/` are the single source of truth for the
  structure of specs, plans, tasks, checklists and agent-facing guidance;
  generated documents MUST not diverge structurally from these templates unless
  the templates are updated in the same change.
- Changes to this constitution that affect required sections or fields in
  templates MUST be reflected by updating the relevant templates in
  `.specify/templates/` and, where applicable, regenerating derived documents.
- This constitution uses semantic versioning:
  - MAJOR: Breaking changes to principles or governance that alter expected
    behaviour of plans/specs/tasks.
  - MINOR: New principles or sections, or materially expanded guidance.
  - PATCH: Clarifications, wording changes and non-semantic refinements.

Rationale: Template-driven governance keeps AI outputs consistent and makes
constitution changes predictable for downstream tools and contributors.

## Additional Constraints & Safety Requirements

- Planning and documentation artefacts produced from this repository MUST focus
  on analysis, structure and decision-making, not on executing or orchestrating
  upgrades directly.
- Destination projects that use this framework MUST maintain a persistent
  "CivicTheme customisation register" file at the canonical path
  `docs/civic-theme-upgrades/customisations.md`. On every run in a given
  project, the upgrade assistant or maintainer MUST ensure that this register
  exists and is up to date by combining interactive input from developers with
  analysis of the project's version control history for CivicTheme and any
  CivicTheme-based sub-themes. If the register does not exist, it MUST be
  created at this path; if it exists, it MUST be updated to capture any new or
  changed customisations since the last run.
- CivicTheme upgrade paths MUST be planned and executed one upstream release at
  a time. Each per-version spec and its associated runbook MUST cover a single
  contiguous upgrade step (for example `v1.9.0` → `v1.10.0`). Avoid skipping
  more than one CivicTheme release in a single planned upgrade; when multiple
  releases must be traversed, separate specs and plans MUST be created for each
  step and follow the per-version spec directory structure defined in the
  CivicTheme upgrade documentation framework.
- Every per-version CivicTheme upgrade MUST include explicit version
  verification at both the start and the end of the upgrade:
  - Before any upgrade work proceeds, the relevant tasks and runbooks MUST
    confirm that the installed `drupal/civictheme` version matches the
    documented "from" version for that step (as recorded in the spec, tasks and
    per-version directory name) and define a STOP CONDITION if it does not. In
    that case the upgrade MUST NOT proceed until the environment and
    documentation are aligned, and a different per-version plan MUST be used if
    the installed version is outside the intended range.
  - Before an upgrade is treated as complete or used as the basis for a
    subsequent CivicTheme upgrade, validation sections in `tasks.md` and
    `playbook.md` MUST re-check that the installed `drupal/civictheme` version
    matches the documented "to" version for that step, and any mismatch MUST be
    treated as a failed upgrade that blocks further work until resolved.
- For each per-version CivicTheme upgrade directory, `spec.md` MUST focus on
  **what and why** (context, upstream changes, risks, customisations, success
  criteria), `tasks.md` MUST act as the tickable **checklist** derived from
  that spec, and `playbook.md` MUST define **how** the work is executed as an
  ordered runbook in non-production or otherwise safe environments.
- AI coding assistants operating on repositories that consume this framework
  MUST prefer editing existing source and documentation files over creating new
  ones, and MUST avoid editing minified or generated assets in downstream
  projects.
- Existing `.gitignore` files in destination projects MUST NOT be modified
  during CivicTheme upgrades. The assumption is that the theme is already
  operational and its ignored files and folders (such as `node_modules/`,
  `dist/`, vendor directories, and build artefacts) are correctly configured.
  Modifying `.gitignore` can disrupt the existing build pipeline, accidentally
  commit generated files, or break local development environments.
- When this framework is applied to a specific project, any additional
  organisation- or security-specific constraints MUST be recorded in that
  project's per-version specs or execution documents, with links back to those
  documents from plan and task files where relevant.

Rationale: These constraints align the framework with safe development
practices and ensure project-specific rules are documented where they are
enforced.

## Development Workflow & Quality Gates

- The default workflow for work that uses this framework is:
  1. Feature or upgrade spec (`spec-template.md`),
  2. Implementation plan (`plan-template.md`),
  3. Task list (`tasks-template.md`),
  4. Code and configuration changes guided by those documents.
- Each `plan.md` MUST include a "Constitution Check" section that:
  - summarises how the plan complies with principles I–V, and
  - lists any deliberate violations together with justification and links to the
    "Complexity Tracking" table at the end of the plan.
- Checklists generated from `.specify/templates/checklist-template.md` MUST be
  explicit about what is being verified (spec completeness, plan alignment,
  tasks coverage or execution verification) and SHOULD reference user stories or
  inventory IDs where applicable.

Rationale: A consistent workflow and explicit quality gates make it easier to
reason about upgrade safety and provide clear review points for humans.

## Governance

- This constitution governs how `.specify/templates/*` and related documentation
  in this repository are interpreted and used by AI tooling and human
  contributors.
- Amendments to the constitution MUST:
  - be made via pull request,
  - include an updated version number according to the semantic versioning
    rules in Principle V,
  - update the Sync Impact Report comment at the top of this file, and
  - call out any required template changes in `.specify/templates/`.
- Substantive changes to CivicTheme upgrade governance – including new
  validation patterns, additional principles or structural changes to the
  canonical documentation layout – SHOULD be proposed and reviewed via new
  feature specifications under `specs/` (for example
  `specs/001-rely-contents-docs/spec.md`) and, where they affect global
  rules, MUST be accompanied by amendments to this constitution rather than
  ad-hoc edits to downstream quickstarts or planning documents.
- Any feature, plan or task document generated under this framework MUST be
  reviewed for compliance with the current constitution before being treated as
  authoritative input for code changes.
- When a downstream project needs to diverge from this constitution, that
  divergence MUST be documented explicitly in the project's own documentation
  (for example, in a project-specific constitution or override section) and
  referenced from the relevant specs and plans.

Rationale: Governance rules make the evolution of the framework predictable,
ensure changes are reviewed and documented, and keep AI tooling aligned with
current expectations.

**Version**: 1.5.0 | **Ratified**: 2025-12-04 | **Last Amended**: 2025-12-04
