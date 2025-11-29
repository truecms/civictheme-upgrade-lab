# CivicTheme upgrade tasks: v1.11.0 → v1.12.0

**Directory**: `docs/civic-theme-upgrades/versions/v1.11.0-to-v1.12.0/`  
**Related documents**: `spec.md`, `playbook.md`, `docs/civic-theme-upgrades/customisations.md`

Tasks are grouped into **Discovery**, **Change** and **Validation** phases.
They are intended to be adapted per project; IDs here are examples.

**Note**: This is a **security-focused release**. The primary driver for
upgrading is the fixes for SA-CONTRIB-2025-112 (Information disclosure)
and SA-CONTRIB-2025-113 (Cross-site Scripting). Tasks are prioritised
accordingly.

---

## Discovery tasks

- [ ] T200 [P] Confirm current CivicTheme and Drupal core versions
  - Run `composer show drupal/civictheme | grep versions` to confirm
    CivicTheme `1.11.x` is installed.
  - Run `drush status` or check `core/lib/Drupal.php` for Drupal core
    version.
  - Verify Drupal core meets `^10.2 || ^11` requirement (unchanged from
    1.11.0).
  - **Note**: Use `ahoy drush` or `docker compose exec cli drush` if
    running in a Docker-based local environment.

- [ ] T200a [P] Discover available front-end commands
  - Check `.ahoy.yml` for front-end commands:
    ```bash
    grep -E "^\s*(fe|front|npm|build|storybook)" .ahoy.yml
    grep -A 10 "^\s*fe:" .ahoy.yml  # Check what 'fe' command does
    ```
    Common patterns: `ahoy fe`, `ahoy build`, `ahoy storybook`.
  - **Key**: `ahoy fe` as a standalone command typically runs the complete
    front-end build (npm install + npm run build).
  - Document discovered commands in the planning notes.

- [ ] T200b [P] Discover available test commands
  - Check `.ahoy.yml` for test-related commands:
    ```bash
    grep -i "test" .ahoy.yml
    ```
    Common patterns: `ahoy test-bdd`, `ahoy test-unit`, `ahoy test-playwright`.
  - Check `composer.json` for test scripts:
    ```bash
    grep -A 30 '"scripts"' composer.json | grep -i "test"
    ```
  - Document discovered commands in the planning notes.

- [ ] T200c [P] Run tests before upgrade (baseline)
  - Execute all discovered test commands to establish a passing baseline.
  - Record results: which tests pass, which fail (pre-existing).
  - **Blocker**: If critical tests fail, fix them before proceeding with
    the upgrade. Do not upgrade on a broken baseline.

- [ ] T201 [P] Review security advisories
  - Read SA-CONTRIB-2025-112 (Information disclosure).
  - Read SA-CONTRIB-2025-113 (Cross-site Scripting).
  - Note which components/templates are affected by these vulnerabilities.
  - **Action**: If sub-theme overrides any affected components, flag for
    security review in T210.

- [ ] T202 [P] Refresh customisation register
  - Update `docs/civic-theme-upgrades/customisations.md` using Git history
    and maintainer input.
  - Ensure all CivicTheme-related customisations are listed with stable
    IDs (Twig overrides, SCSS/JS, configuration).
  - Mark each customisation with impact level: `HIGH`, `MEDIUM`, `LOW`.

- [ ] T203 Audit sub-theme for security-relevant overrides
  - **T203a**: Search for templates handling user input or user-generated
    content that may need security review:
    ```bash
    grep -rn "{{ content" <subtheme>/templates/
    grep -rn "{{ field" <subtheme>/templates/
    grep -rn "{{ raw" <subtheme>/templates/
    grep -rn "|raw" <subtheme>/templates/
    ```
  - **T203b**: Search for menu-related template overrides:
    ```bash
    grep -rn "menu" <subtheme>/templates/
    ls <subtheme>/templates/*menu* 2>/dev/null
    ```
  - **T203c**: Search for attachment component overrides:
    ```bash
    grep -rn "attachment" <subtheme>/templates/
    grep -rn "file" <subtheme>/templates/
    ```
  - Record findings and flag high-risk overrides.

- [ ] T204 Audit library and build tooling
  - **T204a**: Check `<subtheme>.info.yml` for library configuration.
  - **T204b**: Check if sub-theme uses SCSS variables that could be
    migrated to CSS custom properties:
    ```bash
    grep -rn "\$ct-" <subtheme>/scss/
    grep -rn "var(--ct-" <subtheme>/scss/
    ```
  - Document current state and potential migration opportunities.

- [ ] T205 Map findings to customisation register
  - For each finding in T203 and T204:
    - Add or update entry in customisation register.
    - Assign impact level based on Section 5 of `spec.md`.
    - Note required action (e.g. "security review", "menu update",
      "CSS variable migration").

---

## Change tasks

- [ ] T210 [P] Apply CivicTheme composer update
  - Run `composer require drupal/civictheme:^1.12` in a non-production
    environment.
  - Verify `composer.lock` is regenerated.
  - Run `composer install` to confirm lock file consistency.
  - Commit changes to `composer.json` and `composer.lock` together.

- [ ] T211 [P] Security review of sub-theme overrides (HIGH PRIORITY)
  - For each template identified in T203a:
    - Compare with upstream CivicTheme 1.12.0 template.
    - Verify security fixes are not circumvented by overrides.
    - Apply equivalent fixes to sub-theme templates if needed.
  - **Key areas**:
    - User input handling (forms, search, comments).
    - User-generated content display.
    - Any use of `|raw` filter on user data.

- [ ] T212 Update menu customisations (if applicable)
  - For menu-related templates identified in T203b:
    - Compare with upstream CivicTheme 1.12.0 menu templates.
    - Verify compatibility with enhanced menu theming system.
    - Test menu rendering on desktop and mobile.
  - **Note**: Bug fix for `menu_level_modifier_class` scope may affect
    custom menu templates.

- [ ] T213 Update attachment overrides (if applicable)
  - For attachment templates identified in T203c:
    - Verify file extension handling works correctly.
    - Test with multiple file types to confirm extension reset fix.

- [ ] T214 Consider CSS variable migration (optional)
  - If sub-theme uses SCSS variables (`$ct-*`):
    - Evaluate migrating to CSS custom properties (`var(--ct-*)`).
    - Benefits: Runtime customisation without SCSS recompilation.
  - **Note**: This is optional and can be deferred to a separate task.

- [ ] T215 Rebuild sub-theme assets
  - Run front-end build using discovered command from T200a:
    - **Recommended**: `ahoy fe` (standalone - does npm install + build).
    - Or native: `npm run build` or `npm run dist`.
  - Verify build completes without errors.
  - Check that output files are generated correctly.

---

## Validation tasks

- [ ] T220 [P] Clear caches and run system updates
  - Clear Drupal caches: `drush cr` or equivalent.
  - Run database updates: `drush updb`.
  - Import configuration: `drush cim` (if pending).
  - Check for PHP errors during these operations.

- [ ] T221 [P] Verify security fixes are applied
  - **T221a**: Confirm CivicTheme version is 1.12.x:
    ```bash
    composer show drupal/civictheme | grep versions
    ```
  - **T221b**: If security advisory test cases are available, run them.
  - **T221c**: Review sub-theme overrides once more to confirm no
    reintroduction of vulnerabilities.

- [ ] T222 [P] Check logs for errors
  - Review Drupal watchdog/dblog for:
    - PHP errors or warnings.
    - CivicTheme-related errors.
    - SDC component errors.
    - Twig compilation errors.
  - **Expected clean state**: No CivicTheme-related errors in logs.

- [ ] T223 Validate page rendering
  - **T223a – Home page**: Check header, footer, navigation, content.
  - **T223b – Navigation**: Test primary nav, secondary nav, mobile menu.
    Pay special attention if menu customisations exist.
  - **T223c – Search**: Submit search query, verify inline filter
    messaging displays keyword correctly.
  - **T223d – Content pages**: Check article, page, landing page renders.
  - **T223e – Attachments**: Test pages with file attachments, verify
    file extensions display correctly.
  - **T223f – Forms**: Test contact forms, webforms. Verify striptags
    error fix if webform messages were previously affected.

- [ ] T224 Validate specific components
  - **T224a – Menus**: Verify menu rendering, especially mobile menu
    visibility (bug fix in 1.12.0).
  - **T224b – Cards**: Check card components render correctly with
    accessibility updates.
  - **T224c – Videos**: If using video with transcripts, verify
    transcript functionality.
  - **T224d – Links**: Verify email addresses are not incorrectly
    converted to links.
  - **T224e – Logo**: If logo path contains spaces, verify it displays
    correctly (bug fix in 1.12.0).

- [ ] T225 Test new features (optional – if adopting)
  - **T225a – Multi-line header**: If enabling, test configuration and
    verify logo/navigation layout works as expected.
  - **T225b – Message organism**: If using, test all four styling options.
  - **T225c – Fast fact card**: If using, test card display and styling.

- [ ] T226 Test sub-theme build
  - Run front-end build using discovered command from T200a.
  - Verify build completes without errors.
  - Test Storybook (if used): `ahoy fe npm run storybook`.

- [ ] T226b [P] Run tests after upgrade (regression check)
  - Execute the same test commands discovered in T200b.
  - Compare results with baseline recorded in T200c:
    - Tests that passed before and fail now = **REGRESSION** (must fix).
    - Tests that failed before and fail now = **PRE-EXISTING** (document).
    - Tests that failed before and pass now = **IMPROVEMENT** (document).
  - **Blocker**: Do not merge if regressions are introduced. Fix failing
    tests before proceeding.

- [ ] T227 Update customisation register
  - For each customisation impacted by the upgrade:
    - Mark status: `updated`, `retired`, `unchanged`.
    - Add notes on changes made.
    - Record any new customisations introduced during the upgrade.

- [ ] T228 Document follow-ups and lessons learned
  - Record remaining issues in project issue tracker.
  - Update `spec.md` with project-specific findings.
  - Add notes to `playbook.md` on steps that required adjustment.
  - Note whether CSS variable migration should be planned as follow-up.

- [ ] T229 Prepare for review
  - Commit all changes to feature branch.
  - Create merge request referencing this upgrade documentation.
  - Include notes on:
    - Security review performed.
    - Customisations reviewed/updated.
    - Any remaining issues or known regressions.

