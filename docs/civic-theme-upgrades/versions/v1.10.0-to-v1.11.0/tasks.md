# CivicTheme upgrade tasks: v1.10.0 → v1.11.0

**Directory**: `docs/civic-theme-upgrades/versions/v1.10.0-to-v1.11.0/`  
**Related documents**: `spec.md`, `playbook.md`, `docs/civic-theme-upgrades/customisations.md`

Tasks are grouped into **Discovery**, **Change** and **Validation** phases.
They are intended to be adapted per project; IDs here are examples.

---

## Discovery tasks

- [ ] T100 [P] Confirm current CivicTheme version
  - Verify via composer and code inspection that the project is currently
    using CivicTheme `1.10.0`.

- [ ] T101 [P] Refresh customisation register
  - Update `docs/civic-theme-upgrades/customisations.md` using Git history
    and maintainer input to ensure all CivicTheme-related customisations
    (Twig overrides, SCSS/JS, configuration) are listed with stable IDs.

- [ ] T102 Analyse upstream 1.11.0 changes
  - Review:
    - CivicTheme 1.11.0 release notes on drupal.org.
    - Relevant sections of `https://docs.civictheme.io/`.
    - The 1.10.0 → 1.11.0 git diff.
  - Record in `spec.md` which areas (components, templates, assets,
    configuration) are affected.

- [ ] T103 Map upstream changes to customisations
  - For each customisation in the register:
    - Identify whether it touches components or templates that changed in
      1.11.0 (especially SDC-related changes).
    - Mark affected customisations in the register and/or `spec.md`
      (e.g. “impacted by 1.11.0”).

---

## Change tasks

- [ ] T110 Apply CivicTheme composer update
  - Update CivicTheme from `1.10.0` to `1.11.0` (and any required
    supporting packages) using composer in a non-production environment.

- [ ] T111 Reconcile overridden Twig templates
  - For each affected Twig override in the customisation register:
    - Compare local overrides with the updated upstream templates.
    - Adjust overrides to match new structures or retire them if they are
      no longer needed.

- [ ] T112 Review SDC and component structure changes
  - Where the project defines custom SDC components or relies on
    component directories:
    - Ensure directory structures and annotations are compatible with
      CivicTheme 1.11.0 expectations.

- [ ] T113 Align SCSS and JS with updated markup
  - For custom SCSS/JS tied to CivicTheme components:
    - Verify class names and markup used by scripts/styles still match the
      updated templates.
    - Adjust or simplify custom code where CivicTheme provides improved
      defaults.

- [ ] T114 Adjust relevant configuration
  - Review configuration (view displays, blocks, menus, layouts) that rely
    on CivicTheme components or view modes touched by the upgrade and
    adjust as needed.

---

## Validation tasks

- [ ] T120 Verify basic system health
  - After applying changes:
    - Clear caches.
    - Run database updates and configuration import if required.
    - Confirm there are no fatal errors or obvious issues in logs.

- [ ] T121 Validate key pages and components
  - Manually review key pages (e.g. home, search, article/landing pages)
    and components that heavily rely on CivicTheme.
  - Confirm there are no layout breakages or missing functionality.

- [ ] T122 Review impacted customisations
  - For each customisation flagged as impacted:
    - Confirm it behaves as intended after the upgrade.
    - Update the customisation register with notes (e.g. “updated for
      1.11.0”, “retired”).

- [ ] T123 Capture follow-ups and lessons learned
  - Record any remaining issues or follow-up work in:
    - Project issue tracker, and/or
    - Notes within `playbook.md` and `spec.md`.
  - Summarise key findings so that future upgrades can build on this step.

