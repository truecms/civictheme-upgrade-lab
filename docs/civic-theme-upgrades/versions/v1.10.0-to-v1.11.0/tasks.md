# CivicTheme upgrade tasks: v1.10.0 → v1.11.0

**Directory**: `docs/civic-theme-upgrades/versions/v1.10.0-to-v1.11.0/`  
**Related documents**: `spec.md`, `playbook.md`, `docs/civic-theme-upgrades/customisations.md`

Tasks are grouped into **Discovery**, **Change** and **Validation** phases.
They are intended to be adapted per project; IDs here are examples.

---

## Discovery tasks

- [ ] T100 [P] Confirm current CivicTheme and Drupal core versions
  - Verify via composer and code inspection that the project is currently
    using CivicTheme `1.10.0`.
  - Record the running Drupal core version and confirm it satisfies the
    CivicTheme 1.11.0 requirement
    `core_version_requirement: ^10.2 || ^11`.
  - If Drupal is `<10.2`, create a separate issue/spec for the required
    core upgrade and mark this CivicTheme step as **blocked** until that
    work is complete.

- [ ] T101 [P] Refresh customisation register
  - Update `docs/civic-theme-upgrades/customisations.md` using Git history
    and maintainer input to ensure all CivicTheme-related customisations
    (Twig overrides, SCSS/JS, configuration) are listed with stable IDs.

- [ ] T102 Analyse upstream 1.11.0 changes
  - Review:
    - CivicTheme 1.11.0 release notes on drupal.org.
    - Relevant sections of `https://docs.civictheme.io/` including the
      1.11.0 release documentation and SDC guidance.
    - The 1.10.0 → 1.11.0 git diff.
  - Record in `spec.md` which areas (components, templates, assets,
    configuration) are affected.

- [ ] T103 Map upstream changes to customisations
  - For each customisation in the register:
    - Identify whether it touches components or templates that changed in
      1.11.0 (especially SDC-related changes).
    - Note any sub-theme libraries or build scripts that reference
      `dist/civictheme.css` or `dist/civictheme.js`, or that embed a
      copy of the starter-kit `build.js`.
    - Mark affected customisations in the register and/or `spec.md`
      (e.g. “impacted by 1.11.0”).

---

## Change tasks

- [ ] T110 Apply CivicTheme composer update
  - Update CivicTheme from `1.10.0` to `1.11.0` (and any required
    supporting packages) using composer in a non-production environment.
  - Ensure both `composer.json` and `composer.lock` are updated together
    following local composer management conventions.

- [ ] T111 Reconcile overridden Twig templates
  - For each affected Twig override in the customisation register:
    - Compare local overrides with the updated upstream templates.
    - Adjust overrides to match new structures or retire them if they are
      no longer needed.

- [ ] T112 Review SDC and component structure changes
  - Where the project defines custom SDC components or relies on
    component directories:
    - Ensure directory structures and annotations are compatible with
      CivicTheme 1.11.0 expectations and Drupal’s SDC system.
    - Where appropriate, use the CivicTheme SDC Update Tool (the
      `sdc-update` tool from the CivicTheme `upgrade-tools` repository)
      to help generate or update SDC metadata:
      - Confirm prerequisites:
        - Sub-theme is already on CivicTheme 1.10.x.
        - Node.js 22+ is available.
        - An Anthropic API key is available for the tool.
      - In a **separate working copy or feature branch**:
        - Clone `https://github.com/civictheme/upgrade-tools`.
        - Run `npm install` in the cloned repository.
        - Create/update `.env` with:
          - The absolute path to the CivicTheme sub-theme directory.
          - The Anthropic API key (and optional model override).
        - Run `npm run update-components` and follow the interactive
          prompts.
      - After the tool completes:
        - Inspect the `.logs/` directory in the tool repo for errors
          or unexpected changes.
        - Review and curate the resulting changes in the sub-theme
          (updated component directories, `.component.yml` files,
          generated schemas and CSS) before committing anything.

- [ ] T113 Align SCSS and JS with updated markup
  - For custom SCSS/JS tied to CivicTheme components:
    - Verify class names and markup used by scripts/styles still match the
      updated templates.
    - Adjust or simplify custom code where CivicTheme provides improved
      defaults.
    - Update any sub-theme libraries, asset paths or build tooling that
      refer to `dist/civictheme.css` or `dist/civictheme.js` so they
      align with the new split bundles and file names used by
      `civictheme.libraries.yml`.

- [ ] T114 Adjust relevant configuration
  - Review configuration (view displays, blocks, menus, layouts) that rely
    on CivicTheme components or view modes touched by the upgrade and
    adjust as needed.
  - Pay particular attention to:
    - Components that now have SDC definitions (alerts, table-of-contents,
      key cards/lists, forms and fields).
    - Any use of the automated list API or
      `hook_civictheme_automated_list_view_info_alter()` and confirm
      content-type machine names (for example `civictheme_event`) are
      still correct for the project.

---

## Validation tasks

- [ ] T120 Verify basic system health
  - After applying changes:
    - Clear caches.
    - Run database updates and configuration import if required.
    - Confirm there are no fatal errors or obvious issues in logs,
      including errors from the SDC plugin manager or missing components
      such as `civictheme:alert`.

- [ ] T121 Validate key pages and components
  - Manually review key pages (e.g. home, search, article/landing pages)
    and components that heavily rely on CivicTheme.
  - Confirm there are no layout breakages or missing functionality.
  - Where Storybook/design-system is used, confirm the Storybook build
    completes and updated components (including SDC-backed ones) render
    correctly.

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
