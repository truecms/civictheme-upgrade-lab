# CivicTheme upgrades: where to start

This folder contains the **global entry point** for CivicTheme upgrade
work in this repository. It is documentation-only and is intended to be
used by both developers and AI coding assistants.

## 1. Global starting point

When you want to plan or execute a CivicTheme upgrade in a destination
Drupal project, start here:

1. **Read the framework**
   - Skim `docs/civic-theme-upgrades/planning.md` for the global CivicTheme upgrade
     documentation framework and governance.

2. **Locate or create the customisation register**
   - Ensure `docs/civic-theme-upgrades/customisations.md` exists.
   - In a real project, replace the example content with a
     project-specific inventory of CivicTheme customisations (Twig,
     SCSS/CSS, JS, configuration).
   - Treat this register as the **canonical index of upgrade risk**:
     per-version specs, tasks and playbooks should reference IDs from
     this file (for example `C1`, `C2`).

3. **Choose the CivicTheme upgrade step**
   - Upgrades are planned **one CivicTheme release at a time**.
   - For each contiguous step, there is (or will be) a directory under:

     ```text
     docs/civic-theme-upgrades/versions/
       vX.Y.Z-to-vA.B.C/
     ```

   - In this repository, the documented upgrade steps are:

     ```text
     docs/civic-theme-upgrades/versions/v1.10.0-to-v1.11.0/  # SDC migration
     docs/civic-theme-upgrades/versions/v1.11.0-to-v1.12.0/  # Security release
     ```

4. **If the version directory does not exist yet**
   - Create a new directory under `versions/` following the pattern
     `v<OLD>-to-v<NEW>/`.
   - Use `v1.10.0-to-v1.11.0` as a concrete reference for the
     `spec.md` / `tasks.md` / `playbook.md` / `planning.md` layout.
   - Populate `spec.md`, `tasks.md` and `playbook.md` using the
     templates and guidance in `docs/civic-theme-upgrades/planning.md` Section "How to
     create a new per version spec, tasks and playbook".

Once the relevant version directory is present, all day-to-day work for
that upgrade happens inside that directory.

## 2. How to treat files in each version directory

Each directory under `docs/civic-theme-upgrades/versions/` represents a
**single CivicTheme upgrade step**, for example:

```text
docs/civic-theme-upgrades/versions/v1.10.0-to-v1.11.0/
  spec.md
  tasks.md
  playbook.md
  planning.md        # optional scratchpad
```

For **all** such directories:

- `spec.md` – **What & Why**
  - Canonical per-version specification.
  - Describes environment assumptions, upstream links, project context,
    customisation inventory view and risk assessment.
  - Explains *what* needs to change and *why*, but does not contain
    copy-pasteable destructive commands for production.

- `tasks.md` – **Checklist**
  - Concrete, tickable tasks derived from `spec.md`.
  - Groups work into phases (for example discovery / change /
    validation).
  - References customisation IDs from
    `docs/civic-theme-upgrades/customisations.md` and any relevant
    sections of `spec.md`.

- `playbook.md` – **How / Runbook**
  - Ordered, non-production execution steps implementing the tasks.
  - Includes commands, checks and notes about where developer review is
    required.
  - Must assume a safe environment (feature branch, staging, etc.) and
    must remain consistent with `spec.md` and `tasks.md`.

- `planning.md` (optional) – **Planning scratchpad**
  - Free-form notes and prompts for developers and AI.
  - Used before or while refining `spec.md`, `tasks.md` and
    `playbook.md`.
  - **Not** a source of truth: once decisions are made, they must be
    reflected in the spec/tasks/playbook trio.

These roles are enforced by the framework in `docs/civic-theme-upgrades/planning.md`. Do **not** repurpose
these files for unrelated documentation.

## 3. Guidance for AI coding assistants

When operating in this repository or in a downstream project that adopts
this framework, AI coding assistants SHOULD follow these rules:

1. **Start at the right level**
   - For global understanding, read `docs/civic-theme-upgrades/planning.md`.
   - For a specific upgrade, work from the relevant directory under
     `docs/civic-theme-upgrades/versions/`.

2. **Respect the spec → tasks → playbook flow**
   - Plan and analyse in `spec.md` first.
   - Translate agreed work into tickable items in `tasks.md`.
   - Only then update `playbook.md` with ordered execution steps.
   - Keep all three documents in sync; if reality diverges, update the
     spec and tasks, not just the playbook.

3. **Use the customisation register as the risk map**
   - Always consult `docs/civic-theme-upgrades/customisations.md` when
     planning or modifying per-version docs.
   - Reference customisation IDs (for example `C5`, `C7`) in
     `spec.md`, `tasks.md` and `playbook.md` where relevant.

4. **One upgrade step per directory**
   - Do not mix multiple CivicTheme target versions in a single
     directory.
   - When a project needs to cross several versions, create one
     directory per step and repeat the spec → tasks → playbook pattern.

5. **Planning files are not runtime dependencies**
   - `docs/civic-theme-upgrades/planning.md` and any `planning.md` inside version
     directories are helpful for orientation, but AI tooling MUST be
     able to operate using only:
     - the per-version `spec.md`, `tasks.md`, `playbook.md`, and
     - the global customisation register at
       `docs/civic-theme-upgrades/customisations.md`.

By following this structure, both developers and AI can confidently answer
"where do I start?" and "how should I treat these files?" for any
CivicTheme upgrade step in this repository.

---

## 4. Starting prompt for AI assistants

When you begin working with the CivicTheme Upgrade Assistant in a
destination project, copy and paste the following prompt into your AI
coding assistant (Cursor, Copilot, Claude, etc.) to bootstrap the process:

---

**Copy the prompt below** (everything between the triple backticks):

```text
I am working on a Drupal project that uses CivicTheme. This project has
adopted the CivicTheme Upgrade Assistant framework for managing upgrades.

Please help me plan and execute a CivicTheme upgrade by following these
steps:

1. **Orientation**: Read the following files to understand the framework:
   - `docs/civic-theme-upgrades/README.md` (entry point)
   - `docs/civic-theme-upgrades/planning.md` (global framework)
   - `docs/civic-theme-upgrades/customisations.md` (customisation register)

2. **Discovery**: Identify the project's current state:
   - Locate the CivicTheme installation (usually `web/themes/contrib/civictheme/`).
   - Identify the sub-theme (usually under `web/themes/custom/`).
   - Determine the current CivicTheme version from `composer.lock` or
     `civictheme.info.yml`.
   - Scan for CivicTheme customisations (Twig overrides, SCSS/CSS, JS,
     configuration).

3. **Customisation register**: Update `docs/civic-theme-upgrades/customisations.md`:
   - If it is a template, populate it with the discovered customisations.
   - If it already exists, verify it is current and add any missing items.
   - Assign stable IDs (C1, C2, etc.) to each customisation.

4. **Target version**: Confirm the target CivicTheme version I want to
   upgrade to. If I have not specified it, suggest the next incremental
   release.

5. **Per-version documentation**: Navigate to or create the appropriate
   version directory under `docs/civic-theme-upgrades/versions/`:
   - If the directory exists, read `spec.md`, `tasks.md` and `playbook.md`.
   - If it does not exist, create it following the pattern
     `v<CURRENT>-to-v<TARGET>/` and scaffold the spec/tasks/playbook trio.

6. **Planning**: Work through the spec → tasks → playbook flow:
   - Analyse upstream changes for the target version.
   - Cross-reference with the customisation register to identify risks.
   - Propose concrete tasks and execution steps.

7. **Execution** (when I confirm): Follow the playbook steps in a safe
   environment (feature branch, dev/staging), pausing for my review at
   high-risk steps.

Ask me any clarifying questions before proceeding. If you need information
about the target CivicTheme version, I can provide release notes or you can
search for them.
```

---

### Tips for using this prompt

- **Specify your target version** if you know it (for example, "I want to
  upgrade from 1.11.0 to 1.12.0").
- **Point to your sub-theme** if the AI cannot locate it automatically.
- **Review AI-proposed changes** before accepting them – the framework is
  designed to keep developers in control.
- **Update the customisation register** after every upgrade so that future
  upgrades start from accurate information.

