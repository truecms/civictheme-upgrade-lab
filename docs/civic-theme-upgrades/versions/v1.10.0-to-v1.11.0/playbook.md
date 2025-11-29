# CivicTheme upgrade playbook: v1.10.0 → v1.11.0

**Directory**: `docs/civic-theme-upgrades/versions/v1.10.0-to-v1.11.0/`  
**Related documents**: `spec.md`, `tasks.md`, `docs/civic-theme-upgrades/customisations.md`

This playbook turns the tasks in `tasks.md` into an ordered runbook for a
non-production upgrade of CivicTheme from `1.10.0` to `1.11.0`.

---

## 1. Environment checks

1. Confirm you are working in a **non-production** environment (local,
   development, or staging).
2. Ensure you have:
   - Current database and file backups or snapshots.
   - A clean Git working tree or a feature branch dedicated to this
     upgrade.
3. Review `spec.md` to understand the high-level changes and risks for
   this upgrade step.

Related tasks: T100, T120

---

## 2. Discovery

1. **Confirm current CivicTheme version (T100)**
   - Use composer (or equivalent) to confirm the project is currently
     using CivicTheme `1.10.0`.
2. **Refresh customisation register (T101)**
   - Open `docs/civic-theme-upgrades/customisations.md`.
   - With help from Git history and maintainers, ensure all CivicTheme-
     related customisations are present with stable IDs.
3. **Review upstream changes (T102)**
   - Read the 1.11.0 release notes on drupal.org.
   - Skim relevant sections of `https://docs.civictheme.io/`.
   - Glance at the upstream git diff for 1.10.0 → 1.11.0.
   - Note key changes in `spec.md` (components, templates, assets).
4. **Map changes to customisations (T103)**
   - For each customisation in the register, decide whether it is likely
     impacted by 1.11.0 (e.g. due to SDC or template changes).
   - Annotate the register and/or `spec.md` accordingly.

---

## 3. Apply changes

1. **Upgrade CivicTheme via composer (T110)**
   - In the project root, update CivicTheme from `1.10.0` to `1.11.0`
     using composer, ensuring the change is committed to your feature
     branch.
   - Follow project conventions for dependency updates.
2. **Reconcile Twig overrides (T111)**
   - For each overridden Twig template marked as impacted:
     - Compare local overrides with the updated upstream version.
     - Update the override to match new structures or remove it if no
       longer necessary, keeping the customisation register in sync.
3. **Review SDC/component structure (T112)**
   - For custom components or SDC usage:
     - Ensure directories, annotations and naming conventions conform to
       CivicTheme 1.11.0 expectations.
4. **Align SCSS/JS with updated markup (T113)**
   - Inspect custom SCSS and JS tied to CivicTheme components.
   - Update selectors or logic if upstream markup or class names changed.
5. **Adjust configuration (T114)**
   - Review view displays, blocks, menus and other configuration that
     depend on CivicTheme components.
   - Apply necessary adjustments to keep behaviour consistent.

---

## 4. Validate

1. **System health checks (T120)**
   - Clear caches.
   - Run database updates and configuration import if required.
   - Check watchdog/logs for errors related to CivicTheme or components.
2. **Validate key pages and components (T121)**
   - Manually review:
     - Home page.
     - Representative content pages (articles, landing pages).
     - Navigation, search and any other heavily themed components.
   - Note any regressions in layout, styling or behaviour.
3. **Review impacted customisations (T122)**
   - For each customisation marked as impacted:
     - Confirm it behaves as intended.
     - Update the customisation register entry with the outcome (updated,
       retired, unchanged).

---

## 5. Capture outcomes and follow-ups

1. **Record lessons learned (T123)**
   - Summarise key findings from this upgrade in:
     - `spec.md` (e.g. which areas were most fragile).
     - The project’s issue tracker for any follow-up work.
2. **Finalise documentation**
   - Ensure `spec.md`, `tasks.md` and `playbook.md` accurately reflect the
     steps taken and decisions made.
   - Commit the updated documentation and customisation register to Git.
3. **Prepare for review**
   - Open a merge request or equivalent, referencing this directory and
     highlighting any risks or remaining issues for reviewers.

This completes the runbook for the CivicTheme `1.10.0` → `1.11.0` upgrade
step in this repository.
