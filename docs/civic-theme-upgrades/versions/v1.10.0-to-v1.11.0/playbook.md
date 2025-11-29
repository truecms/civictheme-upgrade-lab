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
   this upgrade step, including the new Drupal core requirement
   `core_version_requirement: ^10.2 || ^11`.
4. Using composer or the Drupal status report, note the current Drupal
   core version:
   - If core is `<10.2`, **stop here**, record the dependency in
     `spec.md`/`tasks.md` (T100) and plan a separate core upgrade
     before proceeding with CivicTheme 1.11.0.

Related tasks: T100, T120

---

## 2. Discovery

1. **Confirm current CivicTheme version (T100)**
   - Use composer (or equivalent) to confirm the project is currently
     using CivicTheme `1.10.0`.
   - Record the running Drupal core version and outcome of the
     `^10.2 || ^11` compatibility check.
2. **Refresh customisation register (T101)**
   - Open `docs/civic-theme-upgrades/customisations.md`.
   - With help from Git history and maintainers, ensure all CivicTheme-
     related customisations are present with stable IDs.
3. **Review upstream changes (T102)**
   - Read the 1.11.0 release notes on drupal.org.
   - Skim relevant sections of `https://docs.civictheme.io/`, including
     the 1.11.0 release notes and SDC guidance.
   - Glance at the upstream git diff for 1.10.0 → 1.11.0.
   - Note key changes in `spec.md` (components, templates, assets).
4. **Map changes to customisations (T103)**
   - For each customisation in the register, decide whether it is likely
     impacted by 1.11.0 (e.g. due to SDC or template changes).
   - Identify any sub-theme libraries or build scripts that reference
     `dist/civictheme.css` or `dist/civictheme.js`, or that embed a
     copy of the starter-kit `build.js`.
   - Annotate the register and/or `spec.md` accordingly.

---

## 3. Apply changes

1. **Upgrade CivicTheme via composer (T110)**
   - In the project root, update CivicTheme from `1.10.0` to `1.11.0`
     using composer, ensuring the change is committed to your feature
     branch.
   - Ensure both `composer.json` and `composer.lock` are updated
     together and follow local conventions for dependency updates.
   - Run `composer install` afterwards to confirm the lock file is
     consistent.
2. **Reconcile Twig overrides (T111)**
   - For each overridden Twig template marked as impacted:
     - Compare local overrides with the updated upstream version.
     - Update the override to match new structures or remove it if no
       longer necessary, keeping the customisation register in sync.
3. **Review SDC/component structure (T112)**
   - For custom components or SDC usage:
     - Ensure directories, annotations and naming conventions conform to
       CivicTheme 1.11.0 expectations and Drupal’s SDC rules.
     - Where appropriate, run the CivicTheme SDC update tooling
       (from the CivicTheme upgrade-tools repository) against the
       project’s custom components:
       - Configure it to target the relevant theme(s) only.
       - Review all generated `.component.yml` and `.css` files
         carefully before committing.
4. **Align SCSS/JS with updated markup (T113)**
   - Inspect custom SCSS and JS tied to CivicTheme components.
   - Update selectors or logic if upstream markup or class names changed.
   - For any sub-theme libraries or build scripts that referenced
     `dist/civictheme.css` or `dist/civictheme.js`, update them to use
     the new split bundles exposed by `civictheme.libraries.yml`
     (for example `dist/civictheme.base.css`,
     `dist/civictheme.theme.css`, `dist/civictheme.variables.css`,
     `dist/civictheme.drupal.base.js`).
5. **Adjust configuration (T114)**
   - Review view displays, blocks, menus and other configuration that
     depend on CivicTheme components.
   - Apply necessary adjustments to keep behaviour consistent.
   - Pay particular attention to:
     - Components that now have SDC definitions (alerts, table-of-
       contents, key cards/lists, forms and fields).
     - Any use of the automated list API or
       `hook_civictheme_automated_list_view_info_alter()` and validate
       that content-type machine names and view displays still align
       with project requirements.

---

## 4. Validate

1. **System health checks (T120)**
   - Clear caches.
   - Run database updates and configuration import if required.
   - Check watchdog/logs for errors related to CivicTheme or components,
     including SDC-related errors or missing component libraries such as
     `civictheme:alert`.
2. **Validate key pages and components (T121)**
   - Manually review:
     - Home page.
     - Representative content pages (articles, landing pages).
     - Navigation, search and any other heavily themed components.
     - Pages or blocks that expose alerts or table-of-contents.
   - Note any regressions in layout, styling or behaviour.
   - Where Storybook/design-system is used:
     - Run the Storybook build.
     - Confirm updated components (including SDC-backed ones) render as
       expected.
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
