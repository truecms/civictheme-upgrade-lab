# CivicTheme upgrade tasks: v1.10.0 → v1.11.0

**Directory**: `docs/civic-theme-upgrades/versions/v1.10.0-to-v1.11.0/`  
**Related documents**: `spec.md`, `playbook.md`, `docs/civic-theme-upgrades/customisations.md`

Tasks are grouped into **Discovery**, **Change** and **Validation** phases.
They are intended to be adapted per project; IDs here are examples.

---

## Discovery tasks

- [ ] T100 [P] Confirm current CivicTheme and Drupal core versions
  - Run `composer show drupal/civictheme | grep versions` to confirm
    CivicTheme `1.10.0` is installed.
  - Run `drush status` or check `core/lib/Drupal.php` for Drupal core
    version.
  - Verify Drupal core is `>=10.2` (CivicTheme 1.11.0 requires
    `core_version_requirement: ^10.2 || ^11`).
  - **Blocker**: If Drupal is `<10.2`, this CivicTheme upgrade is blocked.
    Create a separate issue for Drupal core upgrade and complete that
    first.
  - **Note**: Use `ahoy drush` or `docker compose exec cli drush` if
    running in a Docker-based local environment.

- [ ] T100a [P] Discover available front-end commands
  - Check `.ahoy.yml` for front-end commands:
    ```bash
    grep -E "^\s*(fe|front|npm|build|storybook)" .ahoy.yml
    grep -A 10 "^\s*fe:" .ahoy.yml  # Check what 'fe' command does
    ```
    Common patterns: `ahoy fe`, `ahoy build`, `ahoy storybook`.
  - **Key**: `ahoy fe` is equivalent to `npm run build` from the theme directory, but can be invoked from the project root.
  - Document discovered commands in the planning notes.

- [ ] T100b [P] Discover available test commands
  - Check `.ahoy.yml` for test-related commands:
    ```bash
    grep -i "test" .ahoy.yml
    ```
    Common patterns: `ahoy test-bdd`, `ahoy test-unit`, `ahoy test-playwright`,
    `ahoy test-kernel`, `ahoy lint`.
  - Check `composer.json` for test scripts:
    ```bash
    grep -A 30 '"scripts"' composer.json | grep -i "test"
    ```
    Common patterns: `composer test`, `composer test-behat`,
    `composer test-phpunit`.
  - Document discovered commands in the planning notes.

- [ ] T100c [P] Run tests before upgrade (baseline)
  - Execute all discovered test commands to establish a passing baseline.
  - Record results: which tests pass, which fail (pre-existing).
  - **Blocker**: If critical tests fail, fix them before proceeding with
    the upgrade. Do not upgrade on a broken baseline.
  - Example commands (adjust based on T100b discovery):
    ```bash
    ahoy test-unit
    ahoy test-bdd
    composer test
    docker compose exec cli ./vendor/bin/phpunit
    ```

- [ ] T101 [P] Refresh customisation register
  - Update `docs/civic-theme-upgrades/customisations.md` using Git history
    and maintainer input.
  - Ensure all CivicTheme-related customisations are listed with stable
    IDs (Twig overrides, SCSS/JS, configuration).
  - Mark each customisation with impact level: `HIGH`, `MEDIUM`, `LOW`.

- [ ] T102 Audit sub-theme Twig templates for breaking patterns
  - **T102a**: Search for old include patterns:
    ```bash
    grep -rn "include '@atoms/" <subtheme>/templates/
    grep -rn "include '@molecules/" <subtheme>/templates/
    grep -rn "include '@organisms/" <subtheme>/templates/
    grep -rn "include '@base/" <subtheme>/templates/
    ```
    Record file count and locations.
  - **T102b**: Search for extended templates (HIGH RISK):
    ```bash
    grep -rn "{% extends " <subtheme>/templates/
    grep -rn "{{ parent() }}" <subtheme>/templates/
    ```
    Each match requires refactoring – partial extension is not supported.
  - **T102c**: Search for old block names:
    ```bash
    grep -rn "_slot %}" <subtheme>/templates/
    ```
    Each `_slot` must be renamed to `_block`.

- [ ] T103 Audit library overrides and build tooling
  - **T103a**: Check `<subtheme>.info.yml` for `libraries-override`
    referencing `dist/civictheme.css` or `dist/civictheme.js`.
  - **T103b**: Check `<subtheme>.libraries.yml` for old file references.
  - **T103c**: Check if `build.js` exists and compare with 1.11.0 starter
    kit version.
  - **T103d**: Check `package.json` for `@civictheme/sdc` dependency
    (should be added for 1.11.0).
  - **T103e [P]**: Capture custom library attachments **before any upgrade changes**:
    - Grep for `attach_library` in Twig and `#attached` in preprocess/hooks to
      list custom sub-theme libraries (e.g. `sts/event-page`, `component.*`).
    - Grep `<subtheme>.libraries.yml` for custom library names.
    - Record findings (library name, file path, attach location) in
      `planning.md` under a dedicated table so they can be restored after the
      upgrade.

- [ ] T104 Map findings to customisation register
  - For each finding in T102 and T103:
    - Add or update entry in customisation register.
    - Assign impact level based on Section 5 of `spec.md`.
    - Note required action (e.g. "update include syntax", "refactor
      extended template", "update library override").

---

## Change tasks

- [ ] T110 [P] Apply CivicTheme composer update
  - Run `composer require drupal/civictheme:1.11.0` in a non-production
    environment. **Use exact version (no ^/~)** to ensure sequential
    upgrade installs precisely the intended release.
  - Verify `composer.lock` is regenerated.
  - Run `composer install` to confirm lock file consistency.
  - Verify version is exactly 1.11.0: `composer show drupal/civictheme | grep versions`
  - Commit changes to `composer.json` and `composer.lock` together.

- [ ] T111 [P] Update Twig include syntax (HIGH PRIORITY)
  - For each file identified in T102a, update include statements:
    - `{% include '@atoms/<comp>/<comp>.twig'` →
      `{% include 'civictheme:<comp>'`
    - `{% include '@molecules/<comp>/<comp>.twig'` →
      `{% include 'civictheme:<comp>'`
    - `{% include '@organisms/<comp>/<comp>.twig'` →
      `{% include 'civictheme:<comp>'`
    - `{% include '@base/<comp>/<comp>.twig'` →
      `{% include 'civictheme:<comp>'`
  - Common component mappings:
    - `@atoms/paragraph/paragraph.twig` → `civictheme:paragraph`
    - `@atoms/button/button.twig` → `civictheme:button`
    - `@atoms/link/link.twig` → `civictheme:link`
    - `@atoms/heading/heading.twig` → `civictheme:heading`
    - `@molecules/logo/logo.twig` → `civictheme:logo`
    - `@base/icon/icon.twig` → `civictheme:icon`
    - `@base/item-list/item-list.twig` → `civictheme:item-list`

- [ ] T112 [P] Refactor extended templates (HIGH PRIORITY)
  - For each file identified in T102b with `{% extends %}` or
    `{{ parent() }}`:
    - **Option A – Override completely**:
      - Copy the full upstream CivicTheme 1.11.0 template to sub-theme.
      - Apply customisations directly to the copied template.
      - Remove the `{% extends %}` and `{{ parent() }}` patterns.
    - **Option B – Remove override**:
      - Delete the sub-theme template if upstream defaults are acceptable.
  - **Note**: Partial extension is NOT supported in SDC architecture.

- [ ] T113 [P] Update Twig block names
  - For each file identified in T102c, rename blocks:
    - `{% block content_slot %}` → `{% block content_block %}`
    - `{% block content_top_slot %}` → `{% block content_top_block %}`
    - `{% block content_bottom_slot %}` → `{% block content_bottom_block %}`
    - `{% block sidebar_top_left_slot %}` → `{% block sidebar_top_left_block %}`
    - `{% block sidebar_top_right_slot %}` → `{% block sidebar_top_right_block %}`
    - `{% block sidebar_bottom_left_slot %}` → `{% block sidebar_bottom_left_block %}`
    - `{% block sidebar_bottom_right_slot %}` → `{% block sidebar_bottom_right_block %}`
    - `{% block rows_slot %}` → `{% block rows_block %}`
    - `{% block footer_slot %}` → `{% block footer_block %}`
    - `{% block hidden_slot %}` → `{% block hidden_block %}`

- [ ] T114 [P] Update library overrides in `<subtheme>.info.yml`
  - Locate the `libraries-override` section.
  - Replace old file references with new split bundles:
    ```yaml
    libraries-override:
      civictheme/global:
        css:
          theme:
            dist/civictheme.base.css: dist/styles.base.css
            dist/civictheme.theme.css: dist/styles.theme.css
            dist/civictheme.variables.css: dist/styles.variables.css
        js:
          dist/civictheme.drupal.base.js: dist/scripts.drupal.base.js
      civictheme/css-variables:
        css:
          theme:
            dist/civictheme.variables.css: dist/styles.variables.css
    ```

- [ ] T115 Update `<subtheme>.libraries.yml`
  - Update global library definition:
    ```yaml
    global:
      css:
        theme:
          dist/styles.base.css: {}
          dist/styles.theme.css: {}
          dist/styles.variables.css: {}
      js:
        dist/scripts.drupal.base.js: {}
      dependencies:
        - core/drupal
        - core/once
        - core/drupalSettings
    css-variables:
      css:
        theme:
          dist/styles.variables.css: { preprocess: false, weight: 10 }

- [ ] T115a Restore custom library attachments captured in `planning.md`
  - Re-add any custom libraries removed or renamed during the upgrade to
    `<subtheme>.libraries.yml` and related Twig/preprocess attaches.
  - Validate that each captured attach point still loads after cache clear.
    ```

- [ ] T116 Update build tooling (choose one approach)
  - **T116a – Use CivicTheme upgrade-tools (storybook-v8-update / sdc-update)**
    - Prereqs for this optional helper: Node.js 22+ and an Anthropic API key.
      If you prefer not to provide a key, skip T116a and complete T116b
      (manual update) instead – the upgrade remains valid.
    - **⛔ STOP CONDITION – Anthropic API Key Required**
      - AI assistants MUST stop and request developer action before proceeding.
      - The story conversion script requires `ANTHROPIC_API_KEY` to call the
        Anthropic Claude API.
      - **Preferred:** Add `ANTHROPIC_API_KEY=sk-ant-...` to the destination
        project's `.env` file (ensure `.env` is in `.gitignore`).
      - **Alternative:** Export in shell session or add to `~/.bashrc` /
        `~/.zshrc` and source the file.
      - Wait for developer to confirm key is available before proceeding; if
        they do not confirm or decline to provide a key, do **not** run T116a
        and instead follow T116b.
      - **Security:** Remind developer to remove/unset the key after upgrade.
    - Clone tools: `git clone https://github.com/civictheme/upgrade-tools.git`.
    - `cd upgrade-tools/storybook-v8-update && npm install`.
    - Update build + Storybook: `SUBTHEME_DIRECTORY=/abs/path/to/subtheme ./scripts/update-build-and-storybook.sh`.
    - (Optional) Convert stories: `SUBTHEME_DIRECTORY=/abs/path/to/subtheme ANTHROPIC_API_KEY=$ANTHROPIC_API_KEY node -e "import('./scripts/convert-subtheme-storybook.mjs').then(m=>m.default())"`.
    - Common issues/workarounds:
      - Model unavailable → set `model` inside `convert-subtheme-storybook.mjs` to an available Anthropic model (e.g. `claude-sonnet-4-5-20250929`).
      - Missing sass-embedded binary → install optional `sass-embedded-*` packages or temporarily set `build.js` to use `sass`.
      - Wrong `civicthemePath` after script → update `build-config.json` to point to contrib Civictheme directory (e.g. `../contrib/civictheme`).
      - Interactive prompt failure → run the direct script commands above instead of `npm run update-storybook`.
    - Review `.logs/` output and git diff before committing.
    - **Post-completion:** Remove `ANTHROPIC_API_KEY` from `.env` or unset from
      shell environment.
  - **T116b – Manual update**:
    - Copy `build.js` from CivicTheme 1.11.0 starter kit.
    - Copy `build-config.json` from starter kit.
    - Update `package.json` dependencies to match starter kit.
    - Copy `.storybook/sdc-plugin.js` from starter kit.
    - Update `.storybook/preview.js` to match starter kit.
    - Run `npm install` to update node_modules.

- [ ] T117 Add SDC prop schema enforcement (optional)
  - Add to `<subtheme>.info.yml`:
    ```yaml
    enforce_prop_schemas: true
    ```

- [ ] T118 Review and update hook implementations
  - If the project implements `hook_civictheme_automated_list_view_info_alter()`:
    - Update content type references (e.g. `'event'` → `'civictheme_event'`
      if using CivicTheme content types).

---

## Validation tasks

- [ ] T120 [P] Clear caches and run system updates
  - Clear Drupal caches: `drush cr` or equivalent.
  - Run database updates: `drush updb`.
  - Import configuration: `drush cim` (if pending).
  - Check for PHP errors during these operations.

- [ ] T121 [P] Check logs for SDC-related errors
  - Review Drupal watchdog/dblog for:
    - `ComponentNotFoundException` errors.
    - Errors mentioning `civictheme:alert` or other components.
    - Twig compilation errors.
    - Missing library warnings.
  - **Expected clean state**: No CivicTheme-related errors in logs.

- [ ] T122 Validate page rendering
  - **T122a – Home page**: Check header, footer, navigation, content.
  - **T122b – Navigation**: Test primary nav, secondary nav, mobile menu.
  - **T122c – Search**: Submit search query, verify results display in
    correct relevance order (fixed in 1.11.0).
  - **T122d – Content pages**: Check article, page, landing page renders.
  - **T122e – Forms**: Test contact forms, webforms if present.

- [ ] T123 Validate specific components
  - **T123a – Alerts**: Check alert component renders (SDC integration).
  - **T123b – Table of contents**: If used, verify new TOC component.
  - **T123c – Cards**: Check card components (event, news, promo, etc.).
  - **T123d – Automated lists**: Verify list rendering and CSS classes.
  - **T123e – Buttons**: Check button variants and states.
  - **T123f – Icons**: Verify icon rendering.

- [ ] T124 Validate Storybook (if used)
  - Run `npm run storybook` in sub-theme directory.
  - Verify build completes without errors.
  - Check that components render correctly in Storybook.
  - Verify SDC-backed components have correct prop interfaces.

- [ ] T125 Test sub-theme build
  - Run front-end build using discovered command from T100a:
    - **Recommended**: `ahoy fe` (equivalent to `npm run build` from theme directory).
    - Or native: `npm run build` or `npm run dist`.
    - Or docker: `docker compose exec cli bash -c "cd web/themes/custom/yourtheme && npm run build"`.
  - Verify build completes without errors.
  - Check that output files are generated:
    - `dist/styles.base.css`
    - `dist/styles.theme.css`
    - `dist/styles.variables.css`
    - `dist/scripts.drupal.base.js`

- [ ] T125b [P] Run tests after upgrade (regression check)
  - Execute the same test commands discovered in T100b.
  - Compare results with baseline recorded in T100c:
    - Tests that passed before and fail now = **REGRESSION** (must fix).
    - Tests that failed before and fail now = **PRE-EXISTING** (document).
    - Tests that failed before and pass now = **IMPROVEMENT** (document).
  - Example commands (adjust based on project):
    ```bash
    ahoy test-unit
    ahoy test-bdd
    ahoy test-playwright
    composer test
    docker compose exec cli ./vendor/bin/phpunit
    docker compose exec cli ./vendor/bin/behat
    ```
  - **Blocker**: Do not merge if regressions are introduced. Fix failing
    tests before proceeding.

- [ ] T126 Update customisation register
  - For each customisation impacted by the upgrade:
    - Mark status: `updated`, `retired`, `unchanged`.
    - Add notes on changes made.
    - Record any new customisations introduced during the upgrade.

- [ ] T127 Document follow-ups and lessons learned
  - Record remaining issues in project issue tracker.
  - Update `spec.md` with project-specific findings.
  - Add notes to `playbook.md` on steps that required adjustment.
  - Summarise key lessons for future upgrades.

- [ ] T128 Prepare for review
  - Commit all changes to feature branch.
  - Create merge request referencing this upgrade documentation.
  - Include notes on:
    - Number of templates updated.
    - Customisations impacted.
    - Any remaining issues or known regressions.
