# CivicTheme Upgrade Assistant

**Status: Experimental** – this repository houses a documentation‑first, planning‑first framework to help humans and AI assistants plan and execute CivicTheme upgrades in Drupal projects. It is not a plug‑and‑play module; it is a set of conventions, specs, checklists, and runbooks that still require careful human review.

## Disclaimer

This upgrade assistant framework is provided as a standalone utility and is **not** part of, nor officially supported by, the CivicTheme Drupal project or any CivicTheme maintainers. It is intended to assist with planning and executing common upgrade tasks but is provided “as is” without any warranties or guarantees of any kind, either expressed or implied.

Use this framework at your own risk:

- Always review suggested steps, diffs and commands before running them.
- Always use version control and backups before making changes to your projects.
- Always run upgrades in non‑production environments first.

Nothing in this repository should be interpreted as official CivicTheme guidance; when in doubt, defer to the upstream CivicTheme documentation and release notes.

## What this framework provides

- A canonical folder for upgrade documentation at `docs/civic-theme-upgrades/`.
- Per‑version upgrade “units” under `docs/civic-theme-upgrades/versions/vX.Y.Z-to-vA.B.C/` (for example `v1.10.0-to-v1.11.0`), each containing:
  - `spec.md` – **What + Why**: upstream changes, risks, customisations, success criteria.
  - `tasks.md` – **Checklist**: concrete, tickable items derived from the spec.
  - `playbook.md` – **How**: ordered runbook implementing those tasks (non‑prod first).
  - `planning.md` – optional scratchpad for planning (never the source of truth).
- A global `docs/civic-theme-upgrades/customisations.md` file acting as a **customisation register** for your CivicTheme sub‑theme and project‑specific overrides.
- Global planning and governance docs in:
  - `.specify/memory/constitution.md` – framework "constitution" and guardrails.
  - `docs/civic-theme-upgrades/planning.md` – how this assistant expects specs/tasks/playbooks to be structured.

Downstream projects are expected to adapt and extend these documents rather than treat them as generated code.

## Installing into your project

You typically add this framework **into an existing Drupal project that already uses CivicTheme**. The goal is to introduce the `docs/civic-theme-upgrades/` structure (plus supporting docs) into that project.

### Option 1: Download as a ZIP

1. Open this repository in your Git hosting UI and choose “Download ZIP”.
2. Extract the archive somewhere outside your project.
3. From the extracted folder, copy at minimum:
   - `docs/civic-theme-upgrades/` → into your project's `docs/` directory (create `docs/` if it does not exist).
   - Optionally, also copy:
     - `.specify/` and `specs/` (if you want the full planning framework and examples).
4. After copying, your project will contain, for example:
   - `docs/civic-theme-upgrades/README.md`
   - `docs/civic-theme-upgrades/customisations.md`
   - `docs/civic-theme-upgrades/planning.md` (global planning conventions)
   - `docs/civic-theme-upgrades/versions/v1.10.0-to-v1.11.0/` (and similar per‑version folders as you add them)

This is the simplest way to “vendor” the documentation into an existing repository without introducing another Git remote.

### Option 2: Clone inside your project

1. From the root of your Drupal project, choose a tools/vendor folder, for example:

   ```sh
   mkdir -p tools
   cd tools
   git clone https://github.com/truecms/civictheme-upgrade-lab.git civictheme-upgrade-assistant
   ```

2. Your project will now contain:
   - `tools/civictheme-upgrade-assistant/docs/civic-theme-upgrades/`
   - `tools/civictheme-upgrade-assistant/.specify/` and `specs/` (internal to the framework).
3. Decide how you want to expose the upgrade docs:
   - Either **reference them in place** (for example, point humans and AI assistants at `tools/civictheme-upgrade-assistant/docs/civic-theme-upgrades/`), or
   - Copy or symlink `tools/civictheme-upgrade-assistant/docs/civic-theme-upgrades/` into your main `docs/` directory so the canonical path in your project becomes `docs/civic-theme-upgrades/…`.

Cloning keeps this framework as a separate Git history while still making all docs available in your project.

## How to use it (high level)

Once the `docs/civic-theme-upgrades/` folder is present in your project:

- Keep your real CivicTheme customisations up to date in `docs/civic-theme-upgrades/customisations.md`.
- For each CivicTheme release step (for example, 1.10.0 → 1.11.0):
  - Create `docs/civic-theme-upgrades/versions/v1.10.0-to-v1.11.0/`.
  - Author or update:
    - `spec.md` (What + Why),
    - `tasks.md` (Checklist),
    - `playbook.md` (How/runbook),
    - optionally `planning.md` (scratchpad).
- Treat `spec.md`, `tasks.md`, and `playbook.md` for a given version step as **one coherent unit of work**.
- Use the playbook only in non‑production environments and always align it with your project’s deployment standards and governance.

For more detailed guidance, see `docs/civic-theme-upgrades/README.md` inside this repo.

## Human verification is required

This framework is intentionally conservative and documentation‑driven:

- It does **not** attempt to fully automate Drupal upgrades.
- Every spec, checklist, and playbook step should be reviewed and adapted to your project’s:
  - CivicTheme sub‑theme,
  - contrib modules,
  - custom modules and overrides,
  - infrastructure and deployment practices.
- AI assistants using this framework must be treated as helpers, not authorities: humans remain accountable for verifying diffs, running tests, validating environments, and deciding what is safe to deploy.

If you are not comfortable treating this as an experimental tool that needs human oversight, you should not run any commands suggested by the playbooks.

## Feedback and contributions

Because this framework is experimental, feedback is especially welcome:

- If you have suggestions for improving the upgrade structure, naming, or workflows, please open a GitHub issue in this repository.
- If you have concrete improvements (better specs, clearer tasks, safer playbooks, or additional version steps), please submit a pull request.
- If you successfully use this framework in a real project, consider sharing your experience so the guidance and examples can be refined.

Together we can evolve this into a more robust and reliable assistant for CivicTheme upgrades, while keeping human judgment at the center of every change.
