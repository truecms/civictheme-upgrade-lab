# CivicTheme upgrade documentation framework

## 1. Purpose and scope

This document defines a reusable framework for writing CivicTheme upgrade specifications.

- It is for **planning and documentation**, not for performing an actual upgrade.
- It is intended to be copied and adapted when you create **per version** upgrade specifications.
- Any version numbers used in this framework (for example `1.5.0`, `1.9.0`, `1.12.1`) are **examples only**.
- Real upgrade steps, commands and decisions will be written later in separate files that follow this framework.
- It is a governance and planning document primarily for human maintainers;
  AI coding assistants MUST NOT depend on this file existing at run time
  and SHOULD instead rely on per-version `spec.md` / `tasks.md` /
  `playbook.md` files and the feature specification
  `specs/001-rely-contents-docs/spec.md`.

## 2. Audience

This framework is designed for:

- **AI coding assistants** that will:
  - Read these specification files.
  - Discover the current CivicTheme version and sub-theme structure.
  - Propose or generate upgrade steps.
  - Help reapply customisations on top of newer CivicTheme versions.

- **Human developers** who will:
  - Read and refine the specifications.
  - Decide which changes to apply.
  - Maintain the list of project-specific customisations.

## 3. Repository structure for documentation

Recommended directory layout for CivicTheme upgrade documentation in a
destination project:

```text
docs/
  civic-theme-upgrades/
    framework/
      civic-theme-upgrade-framework.md      # Global framework, not version specific
    versions/
      vX.Y.Z-to-vA.B.C/                     # Example: v1.5.0-to-v1.6.0
        spec.md                             # Per-version upgrade specification (planning & analysis)
        tasks.md                            # Per-version task list derived from the spec
        playbook.md                         # Per-version upgrade execution / runbook
      vA.B.C-to-vD.E.F/
        spec.md
        tasks.md
        playbook.md
```

Notes:
- `civic-theme-upgrade-framework.md` is the global framework that you are
  reading now.
- Each destination project MUST maintain a persistent "CivicTheme customisation
  register" at the canonical path `docs/civic-theme-upgrades/customisations.md`
  so that both humans and AI coding assistants can reliably find it.
- Each real upgrade lives in its own directory under `versions/`, for example:
  - `docs/civic-theme-upgrades/versions/v1.5.0-to-v1.6.0/spec.md`
  - `docs/civic-theme-upgrades/versions/v1.9.0-to-v1.10.0/spec.md`
  In this repository, the first canonical per-version step is implemented at
  `docs/civic-theme-upgrades/versions/v1.10.0-to-v1.11.0/`, which provides a
  real specification, task list and playbook for that upgrade path.
- For each version range directory:
  - `spec.md` captures planning, analysis, upstream references, project
    context and customisation inventory view.
  - `tasks.md` breaks the planned work into concrete tasks that reference
    the spec, customisation register and inventory IDs.
  - `playbook.md` is the executable runbook that turns the plan and tasks
    into ordered steps, commands and checks for performing the upgrade.
- Directory names MUST clearly show the source and target CivicTheme versions.

You may adjust directory names if your organisation has its own naming
conventions, but keep them consistent so both AI and humans can navigate
reliably.

4. Global link registry

Maintain a central list of important CivicTheme links that per version specs can reference.

## Global CivicTheme links

- CivicTheme main site  
  - URL: https://civictheme.io  
  - Purpose: High level information, news, and general overview.

- CivicTheme documentation  
  - URL: https://docs.civictheme.io  
  - Purpose: Main technical documentation for installation, configuration, components and upgrade guidance.

- CivicTheme changelog  
  - URL: <ADD DIRECT CHANGELOG URL WHEN CONFIRMED>  
  - Purpose: Version by version list of changes. Use this as the primary source for understanding what changed between versions.

- CivicTheme Drupal.org project  
  - URL: https://www.drupal.org/project/civictheme  
  - Purpose: Official Drupal.org project, including releases, release notes, issue queue and version history.

- CivicTheme Git repository  
  - URL: <ADD GITHUB OR GIT REPO URL>  
  - Purpose: Source code and commit history. Use to inspect changes between tags if needed.

When you discover more canonical links (for example, a dedicated upgrade guide), add them here so future specs can reuse them.

5. Environment assumptions

Each per version specification created from this framework should assume:
	•	Work is done in a non production or otherwise safe environment (for example feature branch, development environment, ephemeral test environment).
	•	Existing organisational safeguards (such as backups, deployment pipelines and code review) are already in place.
	•	This framework does not prescribe backup procedures, but authors may add a brief reminder if helpful.

Suggested wording for per version specs:

> Environment assumptions  
> - All upgrade work described in this document is performed in a non production environment.  
> - Your standard backup and deployment practices apply and are assumed to be in place.  
> - This document focuses on CivicTheme specific analysis and steps, not on environment protection procedures.

6. Template for per version specification

This section defines a template for the per version spec.md file under versions/.
All placeholders inside angle brackets should be replaced when creating a real spec.

You can copy the following block into a new file and customise it.

# CivicTheme upgrade spec: <PROJECT_NAME> <CURRENT_VERSION> to <TARGET_VERSION>

> IMPORTANT  
> - This specification is for planning and guidance only.  
> - It does not perform an upgrade by itself.  
> - Version numbers in this file are real for this spec, but content is descriptive, not executable.

## 1. Metadata

- Project name: `<PROJECT_NAME>`
- Source CivicTheme version: `<CURRENT_VERSION>`
- Target CivicTheme version: `<TARGET_VERSION>`
- Date created: `<YYYY-MM-DD>`
- Author: `<NAME>`
- Related Jira / ticket IDs: `<LINKS OR IDS>`

## 2. Goal of this spec

Describe at a high level what this spec is intended to achieve.

Example text to adapt:

> This document defines how to upgrade CivicTheme from `<CURRENT_VERSION>` to `<TARGET_VERSION>` for `<PROJECT_NAME>`, including:  
> - where to read upstream release notes and documentation  
> - which customisations must be considered and preserved  
> - what tasks an AI coding assistant should perform  
> - what checks human developers should run to confirm the upgrade is successful

## 3. Upstream references

### 3.1 Primary links

Fill these with the exact URLs relevant to `<CURRENT_VERSION>` and `<TARGET_VERSION>`.

- CivicTheme documentation index: `<LINK TO docs.civictheme.io SECTION>`
- CivicTheme release notes for `<TARGET_VERSION>`: `<DIRECT LINK>`
- CivicTheme changelog entries relevant to this upgrade path:  
  - `<LINK 1>`  
  - `<LINK 2>` (if upgrading across multiple intermediate versions)
- Drupal.org release page for `<TARGET_VERSION>`: `<DIRECT LINK>`
- Any official migration or SDC (Single Directory Components) guidance relevant to these versions: `<LINK>`

### 3.2 Secondary links (optional)

- CivicTheme Git diff between `<CURRENT_VERSION>` and `<TARGET_VERSION>`: `<GIT DIFF OR COMPARE LINK>`
- Issue queue items relevant to this upgrade:  
  - `<ISSUE LINK 1>`  
  - `<ISSUE LINK 2>`

## 4. Project specific context

This section is filled once per project and reused across multiple version specs if appropriate.

### 4.1 Theme structure

Describe the theme and sub-theme structure in the project.

Example structure (replace with real information):

```text
web/themes/custom/
  project_civictheme_subtheme/         # Sub-theme based on CivicTheme
web/themes/contrib/
  civictheme/                          # Upstream CivicTheme

Clarify:
	•	Name of the sub-theme: <SUB_THEME_NAME>
	•	Any additional custom themes or admin themes involved.

4.2 Known customisations (summary)

This is a summary only. Detailed listing will appear later.
	•	Custom Twig templates: <YES/NO, SHORT SUMMARY>
	•	Custom SCSS/CSS: <YES/NO, SHORT SUMMARY>
	•	Custom JavaScript for CivicTheme components: <YES/NO, SHORT SUMMARY>
	•	Custom configuration related to CivicTheme (views, blocks, layouts): <YES/NO, SHORT SUMMARY>

If this is the first spec for a project, you may leave this as a placeholder and fill it once you have done an initial discovery pass with AI and humans.

5. Customisation inventory

This section is central for AI and humans.

The goal is to maintain a machine readable and human readable list of all customisations that sit on top of CivicTheme and may be affected by upgrades.

5.1 Inventory table

Fill or update this table when preparing the spec.

| ID | Area            | File or config key                          | Description of change                                  | Risk level (L/M/H) | Notes |
|----|-----------------|----------------------------------------------|--------------------------------------------------------|--------------------|-------|
| C1 | Twig template   | web/themes/custom/<SUB_THEME_NAME>/templates/... | Override of <COMPONENT_NAME> markup                    | M                  |       |
| C2 | Styles (SCSS)   | web/themes/custom/<SUB_THEME_NAME>/scss/... | Adjusted spacing for <COMPONENT_NAME>                  | L                  |       |
| C3 | JavaScript      | web/themes/custom/<SUB_THEME_NAME>/js/...   | Custom behaviour for <COMPONENT_NAME>                  | H                  |       |
| C4 | Config override | view.<VIEW_ID>                               | Uses specific CivicTheme view display                  | M                  |       |

Notes:
	•	IDs (C1, C2, etc.) are used later in tasks and checklists.
	•	This table should be updated over time as new customisations are added.

5.2 Discovery prompts for AI

These prompts are examples that a human can copy into an AI coding assistant. They are written as if the AI already has access to the repository.

Prompt to identify custom Twig overrides:

Using the project repository, locate all Twig template overrides in the sub-theme <SUB_THEME_NAME> that relate to CivicTheme. For each file, output:
- file path
- guess of which CivicTheme component it affects
- one sentence summary of what it appears to change

Format the result as rows suitable for the "customisation inventory" table (ID left blank).

Prompt to identify SCSS/CSS and JS customisations:

Scan the sub-theme <SUB_THEME_NAME> for SCSS/CSS and JavaScript files that reference CivicTheme CSS classes, components or behaviours.

For each relevant file:
- list the file path
- summarise what it changes and which component or layout is affected
- indicate if it is likely to be impacted by a CivicTheme upgrade

Format the result as rows suitable for the "customisation inventory" table (ID left blank).

These prompts are examples. You can refine or expand them in each spec.

6. Version analysis template

This section explains how to document what is changing between <CURRENT_VERSION> and <TARGET_VERSION> without giving actual upgrade steps.

6.1 High level change summary

### 6.1 High level change summary

- Upstream CivicTheme changes between `<CURRENT_VERSION>` and `<TARGET_VERSION>`:
  - New features: `<SUMMARY FROM RELEASE NOTES>`
  - Breaking or structural changes: `<SUMMARY>`
  - Deprecations or removals: `<SUMMARY>`
  - Security related changes: `<SUMMARY>`

- Impact on this project:
  - Components likely affected: `<LIST>`
  - Layouts likely affected: `<LIST>`
  - Customisations from section 5 likely affected: `<LIST OF C-IDS>`

6.2 Component and pattern mapping

If CivicTheme introduces new component formats (for example, YAML descriptors or SDC structure), map them here.

### 6.2 Component and pattern mapping

- Old component format description:
  - Example: Twig templates and libraries defined in <OLD_PATH>.

- New component format description:
  - Example: SDC components with YAML metadata and template in <NEW_PATH>.

- Mapping actions required (to be elaborated in future concrete spec):
  - For each overridden component, record:
    - old template file
    - new component identifier / path
    - any new YAML or metadata required

This section remains descriptive. The concrete mapping will be filled once you work on a specific upgrade.

7. Task lists for AI coding assistant

This section defines reusable task lists, not actual implementation steps.

You will reference or copy these tasks into future per version specs and adapt them.

7.1 Discovery tasks

#### Discovery tasks (for AI)

1. Determine the current CivicTheme version used in the project.
2. Confirm the target CivicTheme version for this spec.
3. Identify the sub-theme based on CivicTheme and its location.
4. Build or update the customisation inventory (section 5).
5. Cross reference customisations with upstream changes from section 6 to identify potential conflicts.

7.2 Planning tasks

#### Planning tasks (for AI)

1. For each customisation in the inventory, assess whether it:
   - can remain as is
   - needs minor adjustment
   - needs a full rewrite
2. Suggest an ordered list of changes to the sub-theme so that:
   - the base CivicTheme can be updated cleanly
   - customisations are reapplied or adjusted in a controlled way
3. Mark any high risk items (for example, changes to structural templates) for explicit human review.

7.3 Validation tasks

#### Validation tasks (for AI)

1. Propose a checklist of pages, components and user flows to test after the upgrade.
2. Map each checklist item to one or more customisations (C IDs) or upstream changes.
3. Highlight any areas where automated testing is strongly recommended.

These tasks are deliberately generic and should be referenced by future specs and refined when real versions are known.

8. Human developer checklist template

Each per version spec should end with a short checklist for human developers.
This checklist is about reviewing the plan and results in `spec.md` and
`tasks.md`, not performing the upgrade directly.

## 8. Human checklist

Before starting work (planning phase, focusing on `spec.md`):

- [ ] Confirm `<CURRENT_VERSION>` and `<TARGET_VERSION>` values are correct.
- [ ] Confirm all relevant upstream links in section 3 are present and accessible.
- [ ] Review the customisation inventory in section 5 and add any missing items.

After planning with AI (still within `spec.md` and `tasks.md`):

- [ ] Review AI generated analysis of impacts and adjust where necessary.
- [ ] Confirm that high risk customisations are flagged for manual testing or refactoring.
- [ ] Ensure any proposed structural changes to the sub-theme follow project conventions.
- [ ] Ensure `tasks.md` accurately reflects the agreed plan and references
      relevant customisation IDs and upstream changes.

After the actual upgrade is implemented (following `playbook.md`):

- [ ] Confirm that all checklist items and steps in `playbook.md` are complete.
- [ ] Update the customisation inventory and register if any customisations were
      removed, replaced or added during the upgrade.
- [ ] Record any lessons learned in `playbook.md` or a follow-up section so
      that future specs and playbooks can be improved.

9. How to create a new per version spec, tasks and playbook

When a new CivicTheme version is released and you decide to plan an upgrade:

1. Create a new directory under `docs/civic-theme-upgrades/versions/`, for example:

```text
docs/
  civic-theme-upgrades/
    versions/
      v1.9.0-to-v1.10.0/
        spec.md
        tasks.md
        playbook.md
```

2. Copy the template from section 6 of this framework into `spec.md`.

3. Replace placeholders:
   - `<PROJECT_NAME>`
   - `<CURRENT_VERSION>`
   - `<TARGET_VERSION>`
   - Any placeholder URLs and paths.

4. Fill in `spec.md`:
   - Upstream links for the specific versions.
   - Initial high level change summary from official release notes.
   - Initial customisation inventory if known, referencing the project’s
     persistent customisation register.

5. Use the AI task lists (section 7) as prompts to:
   - Enrich the customisation inventory.
   - Analyse the impact of changes.
   - Propose a plan for the upgrade in terms of discrete tasks.

6. Create `tasks.md`:
   - Translate the agreed plan from `spec.md` into a list of tasks grouped by
     theme (discovery, changes, validation) or customisation IDs.
   - Make tasks specific enough that an AI coding assistant or human can act
     on them without re-deriving the plan.

7. Create `playbook.md` (upgrade execution document) once the plan is stable:
   - Turn tasks from `tasks.md` into ordered steps, including commands,
     configuration changes and checks.
   - Make clear environment assumptions (non-production) and any preconditions.

8. Once `spec.md`, `tasks.md` and `playbook.md` are stable, treat them as the
   input documents for actual upgrade work and future audits.

10. Separation of concerns

It is important to keep the following separation clear:

- **This framework**
  - Global patterns and templates for documentation.
  - No specific CivicTheme version logic.

- **Per version spec (`spec.md`, for example `v1.9.0-to-v1.10.0/spec.md`)**
  - Planning and analysis for a specific upgrade path.
  - Upstream references, project context, customisation inventory view and
    version analysis.
  - Still not the place where actual upgrade commands are executed.

- **Per version task list (`tasks.md`)**
  - Structured tasks derived from `spec.md`.
  - References to customisation register entries and inventory IDs.
  - A bridge between planning and execution that remains descriptive rather
    than a shell script.

- **Upgrade execution documentation (`playbook.md`)**
  - Concrete step by step instructions, commands and checks for performing the
    actual upgrade.
  - May be generated or heavily assisted by AI using `spec.md` and `tasks.md`
    as context.

In summary for each per-version directory:

- `spec.md` focuses on **what and why** (context, upstream changes, risks,
  customisations, success criteria).
- `tasks.md` is the **checklist** (concrete, tickable items derived from the
  spec).
- `playbook.md` describes **how** (ordered execution/runbook implementing
  those tasks in a safe environment).

By following this framework, you can incrementally build a library of well
structured CivicTheme upgrade specifications, task lists and playbooks that
support both AI coding assistants and human developers without mixing planning
and execution in the same document.
