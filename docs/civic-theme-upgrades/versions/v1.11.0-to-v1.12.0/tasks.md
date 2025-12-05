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
  - **STOP CONDITION**: If `composer show drupal/civictheme` does not report a
    `1.11.x` release (for example, if the site is still on `1.10.x` or has
    already moved to `1.12.x`), do **not** continue with this per-version
    upgrade. Align the environment with the documented "from" version or
    select the matching CivicTheme upgrade directory before proceeding.
  - **Note**: Use `ahoy drush` or `docker compose exec cli drush` if
    running in a Docker-based local environment.

- [ ] T200a [P] Discover available front-end commands
  - Check `.ahoy.yml` for front-end commands:
    ```bash
    grep -E "^\s*(fe|front|npm|build|storybook)" .ahoy.yml
    grep -A 10 "^\s*fe:" .ahoy.yml  # Check what 'fe' command does
    ```
    Common patterns: `ahoy fe`, `ahoy build`, `ahoy storybook`.
  - **Key**: `ahoy fe` is equivalent to `npm run build` from the theme directory, but can be invoked from the project root.
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

- [ ] T203 Audit sub-theme Twig templates for XSS vulnerabilities
  - **T203a**: Search for `|raw` filter usage (HIGH RISK):
    ```bash
    grep -rn "|raw" <subtheme>/templates/
    grep -rn "{{ raw" <subtheme>/templates/
    ```
    Each match needs review – `|raw` on user data is an XSS vulnerability.
  - **T203b**: Check for `heading.twig` override:
    ```bash
    ls <subtheme>/templates/*/heading* 2>/dev/null
    ls <subtheme>/components/*/heading* 2>/dev/null
    ```
    If exists, verify `|raw` is removed from `{{ content }}`.
  - **T203c**: Check for `button.twig` override:
    ```bash
    ls <subtheme>/templates/*/button* 2>/dev/null
    ls <subtheme>/components/*/button* 2>/dev/null
    ```
    If exists, verify `|raw` is removed from `{{ text }}`.
  - **T203d**: Search for templates with content/field rendering:
    ```bash
    grep -rn "{{ content" <subtheme>/templates/
    grep -rn "{{ field" <subtheme>/templates/
    ```
  - Record findings and flag high-risk overrides.

- [ ] T203e Audit sub-theme preprocess functions for API updates
  - First, list all entity reference fields in the project:
    ```bash
    grep -l "type: entity_reference" config/sync/field.storage*.yml | \
      xargs grep "^field_name:" | awk '{print $2}'
    ```
  - Search for CivicTheme API function calls that need `$variables` argument:
    ```bash
    grep -rn "civictheme_get_field_referenced_entities" <subtheme>/*.theme
    grep -rn "civictheme_get_field_referenced_entities" <subtheme>/includes/
    grep -rn "civictheme_get_field_referenced_entity" <subtheme>/*.theme
    grep -rn "civictheme_get_field_referenced_entity" <subtheme>/includes/
    grep -rn "civictheme_get_field_value" <subtheme>/*.theme
    grep -rn "civictheme_get_field_value" <subtheme>/includes/
    grep -rn "civictheme_get_referenced_entity_labels" <subtheme>/*.theme
    grep -rn "civictheme_get_referenced_entity_labels" <subtheme>/includes/
    ```
  - Each match needs updating to pass `$variables` as argument.
  - **Note**: If NOT using CivicTheme API functions for entity reference
    fields, you are responsible for implementing access checks.

- [ ] T203f Audit sub-theme preprocess for title/label XSS filtering
  - Search for title/label output without filtering:
    ```bash
    grep -rn "->getTitle(" <subtheme>/*.theme
    grep -rn "->getTitle(" <subtheme>/includes/
    grep -rn "->label(" <subtheme>/*.theme
    grep -rn "->label(" <subtheme>/includes/
    grep -rn "->getText(" <subtheme>/*.theme
    grep -rn "->getText(" <subtheme>/includes/
    grep -rn "\['title'\]" <subtheme>/*.theme
    grep -rn "\['title'\]" <subtheme>/includes/
    ```
  - Each match needs `Xss::filter()` applied.

- [ ] T203g Audit for menu preprocessing
  - Search for menu-related template overrides:
    ```bash
    grep -rn "menu" <subtheme>/templates/
    ls <subtheme>/templates/*menu* 2>/dev/null
    ```
  - Search for menu preprocess functions:
    ```bash
    grep -rn "preprocess_menu" <subtheme>/*.theme
    grep -rn "preprocess_menu" <subtheme>/includes/
    ```
  - Menu item titles must have `Xss::filter()` applied.

- [ ] T203h Audit attachment component overrides
  - Search for attachment/file-related templates:
    ```bash
    grep -rn "attachment" <subtheme>/templates/
    grep -rn "file" <subtheme>/templates/
    ```
  - Record findings for T213 update.

- [ ] T204 Audit library and build tooling
  - **T204a**: Check `<subtheme>.info.yml` for library configuration.
  - **T204b**: Check if sub-theme uses SCSS variables that could be
    migrated to CSS custom properties:
    ```bash
    grep -rn "\$ct-" <subtheme>/scss/
    grep -rn "var(--ct-" <subtheme>/scss/
    ```
  - Document current state and potential migration opportunities.
  - **T204c [P]**: Capture custom library attachments **before any changes**:
    - Grep Twig and preprocess/hooks for `attach_library` / `#attached` usage
      to list custom sub-theme libraries.
    - List custom libraries defined in `<subtheme>.libraries.yml`.
    - Record library name, files, and attach locations in
      `planning.md` for this version so they can be restored post-upgrade.

- [ ] T205 Map findings to customisation register
  - For each finding in T203 and T204:
    - Add or update entry in customisation register.
    - Assign impact level based on Section 5 of `spec.md`.
    - Note required action (e.g. "security review", "menu update",
      "CSS variable migration").

---

## Change tasks

- [ ] T210 [P] Apply CivicTheme composer update
  - Run `composer require drupal/civictheme:1.12.0` in a non-production
    environment. **Use exact version (no ^/~)** to ensure sequential
    upgrade installs precisely the intended release.
  - Verify `composer.lock` is regenerated.
  - Run `composer install` to confirm lock file consistency.
  - Verify version is exactly 1.12.0: `composer show drupal/civictheme | grep versions`
  - **STOP CONDITION**: If the installed `drupal/civictheme` version is not
    `1.12.0` after this step, treat the upgrade as incomplete. Do **not**
    move on to later CivicTheme upgrades or mark these security advisories as
    addressed until the version mismatch is understood and corrected.
  - Commit changes to `composer.json` and `composer.lock` together.

- [ ] T211 [P] Remove `|raw` filter from Twig template overrides (CRITICAL)
  - **T211a**: For `heading.twig` override (if exists from T203b):
    ```diff
    - {{- content|raw -}}
    + {{- content -}}
    ```
  - **T211b**: For `button.twig` override (if exists from T203c):
    ```diff
    - {{ text|raw }}
    + {{ text }}
    ```
  - **T211c**: For any other templates identified in T203a using `|raw`:
    - Remove `|raw` from user-entered content fields.
    - If HTML rendering was relied upon, implement safe alternative
      (e.g. allowed tags list, render arrays).

- [ ] T212 [P] Update CivicTheme API calls in preprocess functions
  - For each function call identified in T203e, add `$variables` argument:
  - **T212a**: Update `civictheme_get_field_referenced_entities()` calls:
    ```php
    // Before
    civictheme_get_field_referenced_entities($entity, 'field_name');
    // After
    civictheme_get_field_referenced_entities($entity, 'field_name', $variables);
    ```
  - **T212b**: Update `civictheme_get_field_referenced_entity()` calls:
    ```php
    // Before
    civictheme_get_field_referenced_entity($item, 'field_c_p_reference');
    // After
    civictheme_get_field_referenced_entity($item, 'field_c_p_reference', $variables);
    ```
  - **T212c**: Update `civictheme_get_field_value()` calls:
    ```php
    // Before
    civictheme_get_field_value($block, 'field_name', TRUE);
    // After
    civictheme_get_field_value($block, 'field_name', TRUE, build: $variables);
    ```
  - **T212d**: Enable verbose error reporting to find missed updates:
    ```php
    // In settings.local.php
    error_reporting(E_ALL);
    ini_set('display_errors', TRUE);
    ```

- [ ] T213 [P] Add XSS filtering to title/label outputs
  - For each preprocess function identified in T203f:
  - **T213a**: Add `use` statement for Xss class:
    ```php
    use Drupal\Component\Utility\Xss;
    ```
  - **T213b**: Filter node titles:
    ```php
    $title = Xss::filter($title);
    $title = strip_tags($title);
    ```
  - **T213c**: Filter link text from `getText()`:
    ```php
    $link_text = $link->getText();
    $text = is_array($link_text) ? $link_text : Xss::filter((string) $link_text);
    ```
  - **T213d**: Filter entity labels:
    ```php
    $label = Xss::filter($entity->label());
    ```

- [ ] T214 Add XSS filtering to menu item titles
  - For menu preprocessing identified in T203g:
    ```php
    use Drupal\Component\Utility\Xss;
    
    $item['title'] = isset($item['title']) ? Xss::filter($item['title']) : '';
    ```

- [ ] T215 Update manual list preprocess for access checking
  - If sub-theme overrides manual list preprocessing:
    ```php
    $items = civictheme_get_field_referenced_entities($paragraph, 'field_c_p_list_items', $variables);
    $builder = \Drupal::entityTypeManager()->getViewBuilder('paragraph');
    if ($items) {
      foreach ($items as $item) {
        if (!$item->hasField('field_c_p_reference')) {
          $variables['rows'][] = $builder->view($item);
          continue;
        }
        $referenced_item = civictheme_get_field_referenced_entity($item, 'field_c_p_reference', $variables);
        if ($referenced_item instanceof EntityInterface) {
          $variables['rows'][] = $builder->view($item);
      }
    }
  }

- [ ] T217 Restore custom library attachments captured in `planning.md`
  - Re-add custom libraries (CSS/JS) to `<subtheme>.libraries.yml` if removed
    or renamed by the upgrade.
  - Restore Twig `attach_library()` and preprocess `#attached` entries noted
    in `planning.md`.
  - Clear caches and verify pages depending on those libraries still load the
    assets (events, filters, feedback blocks, etc.).
    ```

- [ ] T216 Remove iframe paragraph attributes field (if applicable)
  - **T216a**: Check if any iframe paragraphs have values in Attributes field:
    ```sql
    SELECT * FROM paragraph__field_c_p_attributes;
    ```
    Or create a View to check.
  - **T216b**: Export and delete field configuration:
    ```bash
    drush cex -y
    rm config/sync/field.field.paragraph.civictheme_iframe.field_c_p_attributes.yml
    rm config/sync/field.storage.paragraph.field_c_p_attributes.yml
    drush cim -y
    ```
  - **T216c**: Update preprocess to add required iframe attributes in code.

- [ ] T218 Update user permissions for Icons media type
  - Remove from Content Author role:
    - "create media" for Icons
    - "edit any media" for Icons
  - Remove from Approver role:
    - "create media" for Icons
    - "edit any media" for Icons
  - Export configuration: `drush cex -y`

- [ ] T219 Update menu customisations (if applicable)
  - For menu-related templates identified in T203g:
    - Compare with upstream CivicTheme 1.12.0 menu templates.
    - Verify compatibility with enhanced menu theming system.
    - Test menu rendering on desktop and mobile.
  - **Note**: Bug fix for `menu_level_modifier_class` scope may affect
    custom menu templates.

- [ ] T219 Update attachment overrides (if applicable)
  - For attachment templates identified in T203h:
    - Verify file extension handling works correctly.
    - Test with multiple file types to confirm extension reset fix.

- [ ] T220 Consider CSS variable migration (optional)
  - If sub-theme uses SCSS variables (`$ct-*`):
    - Evaluate migrating to CSS custom properties (`var(--ct-*)`).
    - Benefits: Runtime customisation without SCSS recompilation.
  - **Note**: This is optional and can be deferred to a separate task.

- [ ] T221 Rebuild sub-theme assets
  - Run front-end build using discovered command from T200a:
    - **Recommended**: `ahoy fe` (equivalent to `npm run build` from theme directory).
    - Or native: `npm run build` or `npm run dist`.
  - Verify build completes without errors.
  - Check that output files are generated correctly.

---

## Validation tasks

- [ ] T230 [P] Clear caches and run system updates
  - Clear Drupal caches: `drush cr` or equivalent.
  - Run database updates: `drush updb`.
  - Import configuration: `drush cim` (if pending).
  - Check for PHP errors during these operations.

- [ ] T231 [P] Verify security fixes are applied
  - **T231a**: Confirm CivicTheme version is 1.12.x:
    ```bash
    composer show drupal/civictheme | grep versions
    ```
    **Blocker / STOP CONDITION**: If the reported version is not within the
    expected `1.12.x` range, do **not** proceed to subsequent CivicTheme
    upgrades or close this one. Investigate and correct the mismatch (for
    example, fix composer constraints or rerun the upgrade) before
    continuing.
  - **T231b**: Check for deprecation warnings (indicates missed API updates):
    ```bash
    drush watchdog:show --filter="deprecated" --count=50
    ```
  - **T231c**: Review sub-theme overrides once more to confirm no
    reintroduction of vulnerabilities.

- [ ] T232 [P] XSS testing (CRITICAL)
  - Test with XSS payload: `<script>alert('☠️');</script>`
  - **T232a**: Enter payload in CivicTheme component text fields.
  - **T232b**: Enter payload in node titles.
  - **T232c**: Enter payload in taxonomy term names.
  - **T232d**: Enter payload in link text fields.
  - **T232e**: Enter payload in menu item titles.
  - **Expected result**: NO alert dialog appears. Content should be
    escaped or stripped.

- [ ] T233 [P] Verify information disclosure fix
  - Test entity reference fields with restricted content:
  - **T233a**: Create a node that references unpublished content.
  - **T233b**: View the node as anonymous user.
  - **Expected result**: Referenced unpublished content should NOT be
    visible or leak information.

- [ ] T234 [P] Check logs for errors and deprecations
  - Review Drupal watchdog/dblog for:
    - PHP errors or warnings.
    - CivicTheme-related errors.
    - SDC component errors.
    - Twig compilation errors.
    - Deprecation warnings about missing `$build` argument.
  - **Expected clean state**: No CivicTheme-related errors in logs.
  - **Action if deprecations found**: Return to T212 and fix missed API calls.

- [ ] T235 Validate page rendering
  - **T235a – Home page**: Check header, footer, navigation, content.
  - **T235b – Navigation**: Test primary nav, secondary nav, mobile menu.
    Pay special attention if menu customisations exist.
  - **T235c – Search**: Submit search query, verify inline filter
    messaging displays keyword correctly.
  - **T235d – Content pages**: Check article, page, landing page renders.
  - **T235e – Attachments**: Test pages with file attachments, verify
    file extensions display correctly.
  - **T235f – Forms**: Test contact forms, webforms. Verify striptags
    error fix if webform messages were previously affected.

- [ ] T236 Validate specific components
  - **T236a – Menus**: Verify menu rendering, especially mobile menu
    visibility (bug fix in 1.12.0).
  - **T236b – Cards**: Check card components render correctly with
    accessibility updates.
  - **T236c – Videos**: If using video with transcripts, verify
    transcript functionality.
  - **T236d – Links**: Verify email addresses are not incorrectly
    converted to links.
  - **T236e – Logo**: If logo path contains spaces, verify it displays
    correctly (bug fix in 1.12.0).

- [ ] T237 Test new features (optional – if adopting)
  - **T237a – Multi-line header**: If enabling, test configuration and
    verify logo/navigation layout works as expected.
  - **T237b – Message organism**: If using, test all four styling options.
  - **T237c – Fast fact card**: If using, test card display and styling.

- [ ] T238 Test sub-theme build
  - Run front-end build using discovered command from T200a.
  - Verify build completes without errors.
  - Test Storybook (if used): Use appropriate Storybook command (e.g., `ahoy storybook` if available).

- [ ] T239 [P] Run tests after upgrade (regression check)
  - Execute the same test commands discovered in T200b.
  - Compare results with baseline recorded in T200c:
    - Tests that passed before and fail now = **REGRESSION** (must fix).
    - Tests that failed before and fail now = **PRE-EXISTING** (document).
    - Tests that failed before and pass now = **IMPROVEMENT** (document).
  - **Blocker**: Do not merge if regressions are introduced. Fix failing
    tests before proceeding.

- [ ] T240 Update customisation register
  - For each customisation impacted by the upgrade:
    - Mark status: `updated`, `retired`, `unchanged`.
    - Add notes on changes made.
    - Record any new customisations introduced during the upgrade.
  - Record security-specific changes:
    - Templates with `|raw` removed.
    - Preprocess functions with API calls updated.
    - XSS filtering added to title/label outputs.

- [ ] T241 Document follow-ups and lessons learned
  - Record remaining issues in project issue tracker.
  - Update `spec.md` with project-specific findings.
  - Add notes to `playbook.md` on steps that required adjustment.
  - Note whether CSS variable migration should be planned as follow-up.
  - Document any XSS test failures and remediation.

- [ ] T242 Prepare for review
  - Commit all changes to feature branch.
  - Create merge request referencing this upgrade documentation.
  - Include notes on:
    - Security review performed (XSS testing results).
    - Templates updated to remove `|raw` filter.
    - Preprocess functions updated with `$variables` argument.
    - XSS filtering added to title/label outputs.
    - Customisations reviewed/updated.
    - Any remaining issues or known regressions.
