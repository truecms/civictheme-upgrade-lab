# CivicTheme upgrade tasks: v1.12.1 → v1.12.2

**Directory**: `docs/civic-theme-upgrades/versions/v1.12.1-to-v1.12.2/`  
**Related documents**: `spec.md`, `playbook.md`, `docs/civic-theme-upgrades/customisations.md`

Tasks are grouped into **Discovery**, **Change** and **Validation** phases.
They are intended to be adapted per project; IDs here are examples.

**Note**: This is a **bug-fix release with breaking changes**. The primary
change is the removal of the `|raw` Twig filter from all components, which
requires sub-theme updates.

---

## Discovery tasks

- [ ] T400 [P] Confirm current CivicTheme and Drupal core versions
  - Run `composer show drupal/civictheme | grep versions` to confirm
    CivicTheme `1.12.1` is installed.
  - Run `drush status` or check `core/lib/Drupal.php` for Drupal core
    version.
  - Verify Drupal core meets `^10.2 || ^11` requirement (unchanged from
    1.12.1).
  - **STOP CONDITION**: If `composer show drupal/civictheme` does not report
    `1.12.1` exactly, do **not** proceed with this per-version upgrade.
    Update the environment or select the correct CivicTheme upgrade
    directory so that the installed version matches the documented "from"
    version before continuing.
  - **Note**: Use `ahoy drush` or `docker compose exec cli drush` if
    running in a Docker-based local environment.

- [ ] T401 [P] Audit sub-theme Twig templates for `|raw` usage
  - Search for all `|raw` filter usages in your sub-theme:
    ```bash
    grep -rn "|raw" <subtheme>/templates/
    grep -rn "| raw" <subtheme>/templates/
    grep -rn "|raw" <subtheme>/components/
    grep -rn "| raw" <subtheme>/components/
    ```
  - Document each occurrence in `planning.md` for update in the change
    phase.
  - **Critical**: These templates MUST be updated before the upgrade can
    be considered complete.

- [ ] T402 [P] Audit sub-theme preprocessing for raw HTML strings
  - Review preprocessing files for HTML string returns:
    ```bash
    grep -rn "\.theme" <subtheme>/*.theme
    grep -rn "includes/" <subtheme>/includes/
    ```
  - Look for patterns like:
    - Direct HTML string assignment to variables
    - Concatenation of HTML tags with content
    - Use of `t()` with HTML embedded
  - Document functions needing Markup object conversion in `planning.md`.

- [ ] T403 Check for overridden CivicTheme components
  - List all overridden templates:
    ```bash
    ls -la <subtheme>/templates/
    ls -la <subtheme>/components/
    ```
  - Compare against upstream changes in 1.12.2 to identify conflicts.
  - Pay special attention to:
    - Promo card templates
    - Button templates
    - Menu templates
    - Breadcrumb templates

- [ ] T404 Refresh customisation register
  - Update `docs/civic-theme-upgrades/customisations.md` if any new
    customisations have been added since the 1.12.1 upgrade.
  - Mark any customisations affected by `|raw` removal.

- [ ] T405 [P] Capture custom library attachments pre-upgrade
  - Grep Twig and preprocess/hooks for `attach_library` / `#attached` usage to
    list custom sub-theme libraries.
  - List custom libraries defined in `<subtheme>.libraries.yml`.
  - Record library name, files, and attach locations in this version's
    `planning.md` so they can be restored after the upgrade.

---

## Change tasks

- [ ] T410 [P] Apply CivicTheme composer update
  - Run `composer require drupal/civictheme:1.12.2` in a non-production
    environment. **Use exact version (no ^/~)** to ensure sequential
    upgrade installs precisely the intended release.
  - Verify `composer.lock` is regenerated.
  - Run `composer install` to confirm lock file consistency.
  - Verify version is exactly 1.12.2:
    ```bash
    composer show drupal/civictheme | grep versions
    ```
  - **STOP CONDITION**: If the installed `drupal/civictheme` version is not
    `1.12.2` after this step, treat the upgrade as incomplete. Do **not**
    proceed to subsequent CivicTheme upgrades or mark this one as complete
    until the version mismatch is understood and corrected.
  - Commit changes to `composer.json` and `composer.lock` together.

- [ ] T411 [P] Update sub-theme Twig templates to remove `|raw`
  - For each template identified in T401:
    - Remove `|raw` filter from variable output.
    - Ensure the variable is passed as a renderable array or Markup object.
  - **Pattern to change**:
    ```twig
    {# Before #}
    {{ content|raw }}
    
    {# After #}
    {{ content }}
    ```
  - Commit template changes with descriptive message.

- [ ] T412 [P] Update preprocessing to use Markup objects
  - For each function identified in T402:
    - Import Markup class: `use Drupal\Core\Render\Markup;`
    - Wrap HTML content with `Markup::create()`.
  - **Pattern to change**:
    ```php
    // Before
    $variables['title'] = '<span>' . $title . '</span>';
    
    // After
    use Drupal\Core\Render\Markup;
    use Drupal\Component\Utility\Html;
    
    $variables['title'] = Markup::create('<span>' . Html::escape($title) . '</span>');
    ```
  - **Security note**: Only use `Markup::create()` for trusted content.
    Use `Html::escape()` for user-provided content.
  - Commit preprocessing changes.

- [ ] T413 Verify overridden component compatibility
  - For each overridden component identified in T403:
    - Check if upstream template has changed.
    - Merge upstream changes if necessary.
    - Remove any `|raw` filters.
  - Document any conflicts in `planning.md`.

- [ ] T414 Rebuild sub-theme assets
  - Run front-end build using discovered command:
    - **Recommended**: `ahoy fe` (equivalent to `npm run build` from theme directory).
    - Or native: `npm run build` or `npm run dist`.
  - Verify build completes without errors.
  - This picks up the updated `@civictheme/sdc` 1.12.2 package.

- [ ] T415 Restore custom library attachments captured in `planning.md`
  - Re-add custom libraries (CSS/JS) to `<subtheme>.libraries.yml` if removed
    or renamed by the upgrade.
  - Restore Twig `attach_library()` and preprocess `#attached` entries noted
    in `planning.md`.
  - Clear caches and verify pages depending on those libraries still load the
    assets.

---

## Validation tasks

- [ ] T420 [P] Clear caches and run system updates
  - Clear Drupal caches: `drush cr` or equivalent.
  - Run database updates: `drush updb`.
  - Import configuration: `drush cim` (if pending).
  - Check for PHP errors during these operations.

- [ ] T421 [P] Verify CivicTheme version
  - Confirm CivicTheme version is 1.12.2:
    ```bash
    composer show drupal/civictheme | grep versions
    ```
  - **Blocker / STOP CONDITION**: If the reported version is not `1.12.2`,
    treat this per-version upgrade as failed. Do **not** proceed to the next
    CivicTheme upgrade step or close this task until the version mismatch is
    understood and corrected.

- [ ] T422 [P] Test template rendering (no double-escaping)
  - Navigate to pages with overridden templates.
  - Verify that HTML content renders correctly (not escaped).
  - **Expected result**: Content displays as intended without `&lt;`,
    `&gt;`, or `&amp;` appearing in rendered text.

- [ ] T423 [P] Test special character rendering
  - Navigate to pages with content containing special characters (`&`, `<`,
    `>`, quotes).
  - Test specifically:
    - Promo card titles with `&` (e.g. "News & Events").
    - Button titles with special characters.
    - Menu items with special characters.
    - Breadcrumb items with special characters.
  - **Expected result**: Characters display correctly, not as HTML entities.

- [ ] T424 Test overridden component rendering
  - For each overridden component (from T403):
    - Navigate to a page using that component.
    - Verify content renders correctly.
    - Check for any console errors in browser developer tools.
  - Document any issues found.

- [ ] T425 Check logs for errors
  - Review Drupal watchdog/dblog for:
    - PHP errors or warnings.
    - CivicTheme-related errors.
    - Twig rendering errors.
  - **Expected clean state**: No new errors related to CivicTheme.

- [ ] T426 Validate page rendering
  - **T426a – Home page**: Check header, footer, navigation, content.
  - **T426b – Content pages**: Check pages with various components.
  - **T426c – Promo cards**: Verify promo card titles render correctly.
  - **T426d – Buttons**: Verify button labels render correctly.
  - **T426e – Navigation**: Verify menu items render correctly.
  - **T426f – Breadcrumbs**: Verify breadcrumb items render correctly.

- [ ] T427 Update customisation register
  - For each customisation impacted by the upgrade:
    - Mark status: `updated`, `retired`, `unchanged`.
    - Add notes on changes made.
  - Document any `|raw` removal updates.

- [ ] T428 Document follow-ups and lessons learned
  - Record any remaining issues in project issue tracker.
  - Update `spec.md` with project-specific findings.
  - Add notes to `playbook.md` on steps that required adjustment.

- [ ] T429 Prepare for review
  - Commit all changes to feature branch.
  - Create merge request referencing this upgrade documentation.
  - Include notes on:
    - Templates updated to remove `|raw`.
    - Preprocessing functions updated to use Markup objects.
    - Special character rendering verified.

