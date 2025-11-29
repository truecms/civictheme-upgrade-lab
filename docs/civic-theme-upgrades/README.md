# CivicTheme upgrades: where to start

This folder contains the **global entry point** for CivicTheme upgrade
work in this repository. It is documentation-only and is intended to be
used by both human maintainers and AI coding assistants.

## 1. Global starting point

When you want to plan or execute a CivicTheme upgrade in a destination
Drupal project, start here:

1. **Read the framework**
   - Skim `docs/planning.md` for the global CivicTheme upgrade
     documentation framework and governance.
   - Check `.specify/memory/constitution.md` for any additional global
     rules that apply to AI tooling and documentation.

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

   - In this repository, the first canonical step is:

     ```text
     docs/civic-theme-upgrades/versions/v1.10.0-to-v1.11.0/
     ```

4. **If the version directory does not exist yet**
   - Create a new directory under `versions/` following the pattern
     `v<OLD>-to-v<NEW>/`.
   - Use `v1.10.0-to-v1.11.0` as a concrete reference for the
     `spec.md` / `tasks.md` / `playbook.md` / `planning.md` layout.
   - Populate `spec.md`, `tasks.md` and `playbook.md` using the
     templates and guidance in `docs/planning.md` Section “How to
     create a new per version spec, tasks and playbook”.

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
  - Includes commands, checks and notes about where human review is
    required.
  - Must assume a safe environment (feature branch, staging, etc.) and
    must remain consistent with `spec.md` and `tasks.md`.

- `planning.md` (optional) – **Planning scratchpad**
  - Free-form notes and prompts for humans and AI.
  - Used before or while refining `spec.md`, `tasks.md` and
    `playbook.md`.
  - **Not** a source of truth: once decisions are made, they must be
    reflected in the spec/tasks/playbook trio.

These roles are enforced by the CivicTheme Upgrade Assistant
constitution and the framework in `docs/planning.md`. Do **not** repurpose
these files for unrelated documentation.

## 3. Guidance for AI coding assistants

When operating in this repository or in a downstream project that adopts
this framework, AI coding assistants SHOULD follow these rules:

1. **Start at the right level**
   - For global understanding, read:
     - `docs/planning.md`
     - `.specify/memory/constitution.md`
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
   - `docs/planning.md` and any `planning.md` inside version
     directories are helpful for orientation, but AI tooling MUST be
     able to operate using only:
     - the per-version `spec.md`, `tasks.md`, `playbook.md`, and
     - the global customisation register at
       `docs/civic-theme-upgrades/customisations.md`.

By following this structure, both humans and AI can confidently answer
“where do I start?” and “how should I treat these files?” for any
CivicTheme upgrade step in this repository.

