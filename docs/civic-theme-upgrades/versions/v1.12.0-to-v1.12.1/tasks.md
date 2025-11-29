# CivicTheme upgrade tasks: v1.12.0 → v1.12.1

**Directory**: `docs/civic-theme-upgrades/versions/v1.12.0-to-v1.12.1/`  
**Related documents**: `spec.md`, `playbook.md`, `docs/civic-theme-upgrades/customisations.md`

Tasks are grouped into **Discovery**, **Change** and **Validation** phases.
They are intended to be adapted per project; IDs here are examples.

**Note**: This is a **bug-fix release** with no new features or breaking
changes. The primary fix addresses a regression in 1.12.0 where embedded
media was being stripped from WYSIWYG content.

---

## Discovery tasks

- [ ] T300 [P] Confirm current CivicTheme and Drupal core versions
  - Run `composer show drupal/civictheme | grep versions` to confirm
    CivicTheme `1.12.0` is installed.
  - Run `drush status` or check `core/lib/Drupal.php` for Drupal core
    version.
  - Verify Drupal core meets `^10.2 || ^11` requirement (unchanged from
    1.12.0).
  - **Note**: Use `ahoy drush` or `docker compose exec cli drush` if
    running in a Docker-based local environment.

- [ ] T301 Check for 1.12.0 workarounds
  - Search for any workarounds implemented for the WYSIWYG/embedded media
    issue in 1.12.0:
    ```bash
    grep -rn "iframe" <subtheme>/includes/
    grep -rn "figure" <subtheme>/includes/
    grep -rn "_civictheme_process__html_content" <subtheme>/
    ```
  - If workarounds exist, flag them for removal in T311.

- [ ] T302 Check text format configuration
  - Review text formats that should convert emails to links:
    ```bash
    drush config:get filter.format.civictheme_rich_text filters
    ```
  - Verify "Convert URLs into links" filter is enabled if email-to-link
    conversion is desired.

- [ ] T303 Refresh customisation register
  - Update `docs/civic-theme-upgrades/customisations.md` if any new
    customisations have been added since the 1.12.0 upgrade.
  - Mark any customisations affected by this release.

- [ ] T304 [P] Capture custom library attachments pre-upgrade
  - Grep Twig and preprocess/hooks for `attach_library` / `#attached` usage to
    list custom sub-theme libraries.
  - List custom libraries defined in `<subtheme>.libraries.yml`.
  - Record library name, files, and attach locations in this version’s
    `planning.md` so they can be restored after the upgrade.

---

## Change tasks

- [ ] T310 [P] Apply CivicTheme composer update
  - Run `composer require drupal/civictheme:^1.12` in a non-production
    environment.
  - Verify `composer.lock` is regenerated.
  - Run `composer install` to confirm lock file consistency.
  - Verify version is 1.12.1:
    ```bash
    composer show drupal/civictheme | grep versions
    ```
  - Commit changes to `composer.json` and `composer.lock` together.

- [ ] T311 Remove 1.12.0 workarounds (if applicable)
  - For each workaround identified in T301:
    - Remove the workaround code.
    - Test that the fix in 1.12.1 resolves the issue.
  - Commit removal of workarounds.

- [ ] T312 Rebuild sub-theme assets (optional)
  - Run front-end build using discovered command:
    - **Recommended**: `ahoy fe` (equivalent to `npm run build` from theme directory).
    - Or native: `npm run build` or `npm run dist`.
  - Verify build completes without errors.
  - This picks up the updated `@civictheme/sdc` 1.12.1 package.

- [ ] T313 Restore custom library attachments captured in `planning.md`
  - Re-add custom libraries (CSS/JS) to `<subtheme>.libraries.yml` if removed
    or renamed by the upgrade.
  - Restore Twig `attach_library()` and preprocess `#attached` entries noted
    in `planning.md`.
  - Clear caches and verify pages depending on those libraries still load the
    assets.

---

## Validation tasks

- [ ] T320 [P] Clear caches and run system updates
  - Clear Drupal caches: `drush cr` or equivalent.
  - Run database updates: `drush updb`.
    - This will run `civictheme_post_update_remove_civictheme_iframe_field_c_p_attributes()`
      which removes the iframe attributes field automatically.
  - Import configuration: `drush cim` (if pending).
  - Check for PHP errors during these operations.

- [ ] T321 [P] Verify CivicTheme version
  - Confirm CivicTheme version is 1.12.1:
    ```bash
    composer show drupal/civictheme | grep versions
    ```

- [ ] T322 [P] Test embedded media rendering
  - Navigate to pages with embedded media (iframes, figures, videos).
  - Verify that embedded content renders correctly.
  - **Expected result**: Embedded media that was stripped in 1.12.0 should
    now display correctly.

- [ ] T323 Test email handling in WYSIWYG content
  - Navigate to pages with email addresses in body content.
  - **If using `civictheme_rich_text` format**: Email addresses should be
    converted to `mailto:` links (filter enabled by default).
  - **If using custom text format without URL filter**: Email addresses
    should remain as plain text.

- [ ] T324 Verify iframe attributes field removal
  - Check that the field has been removed:
    ```bash
    drush config:get field.field.paragraph.civictheme_iframe.field_c_p_attributes 2>&1 | grep -i "not found"
    drush config:get field.storage.paragraph.field_c_p_attributes 2>&1 | grep -i "not found"
    ```
  - **Expected result**: Both config items should report "not found".
  - **Note**: If you already removed these manually in 1.12.0, this is
    expected behaviour.

- [ ] T325 Check logs for errors
  - Review Drupal watchdog/dblog for:
    - PHP errors or warnings.
    - CivicTheme-related errors.
  - **Expected clean state**: No new errors related to CivicTheme.

- [ ] T326 Validate page rendering
  - **T326a – Home page**: Check header, footer, navigation, content.
  - **T326b – Content pages**: Check pages with WYSIWYG content.
  - **T326c – Pages with iframes**: If using iframe paragraphs, verify
    they still function correctly.

- [ ] T327 Update customisation register
  - For each customisation impacted by the upgrade:
    - Mark status: `updated`, `retired`, `unchanged`.
    - Add notes on changes made.
  - Remove any workaround entries that are no longer needed.

- [ ] T328 Document follow-ups and lessons learned
  - Record any remaining issues in project issue tracker.
  - Update `spec.md` with project-specific findings.
  - Add notes to `playbook.md` on steps that required adjustment.

- [ ] T329 Prepare for review
  - Commit all changes to feature branch.
  - Create merge request referencing this upgrade documentation.
  - Include notes on:
    - Workarounds removed (if any).
    - Embedded media rendering verified.
    - Post-update hook execution confirmed.
