# Planning notes: CivicTheme v1.10.0 → v1.11.0

**Directory**: `docs/civic-theme-upgrades/versions/v1.10.0-to-v1.11.0/`  
**Related documents**: `spec.md` (what/why), `tasks.md` (checklist), `playbook.md` (how)

This file is an optional planning scratchpad for humans and AI coding
assistants. It is intended to capture early thoughts and prompts **before**
or while refining `spec.md`, `tasks.md` and `playbook.md`. The canonical
source of truth for this upgrade remains the spec/tasks/playbook trio.

## 1. Planning focus

- Confirm environment assumptions for this run (non-production, backups,
  review process, feature branch).
- Summarise the key upstream changes for CivicTheme `1.11.0` that are
  relevant to this project (link back to `spec.md` Section 3).
- Identify high-risk customisations from the register
  (`docs/civic-theme-upgrades/customisations.md`) that are most likely to be
  impacted by this upgrade (list their IDs here, e.g. `C1`, `C2`).
- Decide the order in which to tackle impacted customisations (for example,
  structural template overrides first, then SCSS/JS, then configuration).
- Note any project-specific constraints or governance rules that should
  influence this upgrade (link to local docs if needed).

## 2. Draft task shaping

Use this section to sketch tasks before they are formalised in `tasks.md`:

- Discovery ideas:
  - …
- Change ideas:
  - …
- Validation ideas:
  - …

Once tasks are stable, ensure they are recorded in `tasks.md` with clear IDs
and that `playbook.md` is updated to reflect the agreed execution order.

## 3. Open questions / risks

Capture any open questions or risks that need resolution before or during the
upgrade. After they are resolved, reflect outcomes in `spec.md`,
`tasks.md`, `playbook.md` and the customisation register as appropriate.

