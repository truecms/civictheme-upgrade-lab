# CivicTheme upgrade playbook: v1.12.0 → v1.12.1

**Directory**: `docs/civic-theme-upgrades/versions/v1.12.0-to-v1.12.1/`  
**Related documents**: `spec.md`, `tasks.md`, `docs/civic-theme-upgrades/customisations.md`

This playbook turns the tasks in `tasks.md` into an ordered runbook for a
non-production upgrade of CivicTheme from `1.12.0` to `1.12.1`.

**Important**: This is a **bug-fix release** with no new features or
breaking changes. The upgrade is straightforward and low-risk.

---

## 1. Environment checks

**Related tasks**: T300

### 1.1 Confirm environment is non-production

```bash
# Verify you're not on production
echo $ENVIRONMENT  # Should be 'local', 'dev', 'staging', etc.
```

### 1.2 Docker/local development environment notes

Most CivicTheme projects use Docker-based local development with containers
for nginx, php-fpm, and a CLI container. Commands that interact with Drupal
(drush, composer, npm) typically run **inside the CLI container**.

**Common patterns**:

- **Ahoy wrapper** (if `.ahoy.yml` exists): Use `ahoy <command>` which wraps
  docker-compose exec calls.
- **Direct docker-compose**: Use `docker compose exec cli <command>`.
- **Native** (if not using Docker): Run commands directly.

**Throughout this playbook**, commands are shown in native form. Prefix with
`ahoy` or `docker compose exec cli` as appropriate for your environment.

### 1.3 Preserve existing `.gitignore`

**IMPORTANT**: Do NOT modify the existing `.gitignore` file in the sub-theme or
project. The theme is already operational and its ignored files and folders
(such as `node_modules/`, `dist/`, vendor directories, and build artefacts) are
correctly configured.

- [ ] Confirmed `.gitignore` will not be modified during this upgrade.

### 1.4 Create backups

- [ ] Database backup created and verified.
- [ ] Files backup created (if applicable).
- [ ] Able to restore to pre-upgrade state if needed.

### 1.5 Create dedicated feature branch

```bash
cd /path/to/drupal/project
git checkout develop  # or main
git pull origin develop
git checkout -b feature/civictheme-1.12.1-upgrade
```

### 1.6 Verify current CivicTheme version

```bash
composer show drupal/civictheme | grep versions
# Expected: 1.12.0
```

**STOP CONDITION**: If this command does not report `1.12.0` exactly, do **not**
continue with this per-version upgrade. Adjust the environment or select the
matching CivicTheme upgrade directory so that the installed version aligns
with the documented "from" version before proceeding.

### 1.7 Verify Drupal core version

```bash
# Check Drupal core version (use ahoy/docker as needed)
drush status --field=drupal-version

# Or via composer
composer show drupal/core | grep versions
```

**Requirement**: Drupal core must meet `^10.2 || ^11` (unchanged from 1.12.0).

---

## 2. Discovery

**Related tasks**: T301, T302, T303

### 2.1 Check for 1.12.0 workarounds (T301)

If you implemented any workarounds for the WYSIWYG/embedded media issue in
1.12.0, identify them now:

```bash
SUBTHEME_PATH=/path/to/your/subtheme

# Search for potential workarounds
echo "=== Checking for iframe-related workarounds ==="
grep -rn "iframe" $SUBTHEME_PATH/includes/ 2>/dev/null

echo "=== Checking for figure-related workarounds ==="
grep -rn "figure" $SUBTHEME_PATH/includes/ 2>/dev/null

echo "=== Checking for _civictheme_process__html_content overrides ==="
grep -rn "_civictheme_process__html_content" $SUBTHEME_PATH/ 2>/dev/null
```

**Document any workarounds found** for removal in Section 3.2.

### 2.2 Check text format configuration (T302)

If you want email addresses to be converted to `mailto:` links, verify
your text format has the URL filter enabled:

```bash
# Check civictheme_rich_text format
drush config:get filter.format.civictheme_rich_text filters

# Look for filter_url in the output
# It should have: status: true
```

**Note**: The `civictheme_rich_text` format has this enabled by default.
If you use a custom text format and want email-to-link conversion, enable
the "Convert URLs into links" filter.

### 2.3 Refresh customisation register (T303)

Review `docs/civic-theme-upgrades/customisations.md` and ensure it's up to
date. For this release, focus on:

- Any WYSIWYG preprocessing customisations.
- Any email handling customisations.
- Any iframe paragraph customisations.

### 2.4 Capture custom library attachments (T304)

```bash
SUBTHEME_PATH=/path/to/your/subtheme

echo "=== attach_library / #attached usage ==="
grep -rn "attach_library" $SUBTHEME_PATH/templates/ $SUBTHEME_PATH | head -100
grep -rn "#attached" $SUBTHEME_PATH | head -100

echo "=== Custom libraries defined ==="
grep -E "^[A-Za-z0-9_.-]+:" $SUBTHEME_PATH/*.libraries.yml | head -50

# Record library name, files, and attach locations in
# docs/civic-theme-upgrades/versions/v1.12.0-to-v1.12.1/planning.md to restore
# them after the upgrade if needed.
```

---

## 3. Apply changes

**Related tasks**: T310, T311, T312, T313

### 3.1 Upgrade CivicTheme via Composer (T310)

```bash
cd /path/to/drupal/project

# Update CivicTheme to exact target version
# IMPORTANT: Use exact version constraint (no ^ or ~) to ensure
# sequential upgrades install precisely the intended release.
composer require drupal/civictheme:1.12.1

# Verify the update
composer show drupal/civictheme | grep versions
# Expected: 1.12.1 (exact version)

# Commit changes
git add composer.json composer.lock
git commit -m "chore: Updated CivicTheme from 1.12.0 to 1.12.1 (bug fix release)"
```

**STOP CONDITION**: After running the above commands, if
`composer show drupal/civictheme | grep versions` does **not** report
`1.12.1`, treat the upgrade as incomplete. Do **not** proceed with later
steps or start the next CivicTheme upgrade until the version mismatch is
resolved.

### 3.2 Remove 1.12.0 workarounds (T311) – if applicable

If you identified workarounds in Section 2.1, remove them now:

```bash
# Example: If you had a workaround in your theme file
# Edit the file to remove the workaround code
# Then commit the removal

git add -A
git commit -m "chore: Removed 1.12.0 WYSIWYG workarounds (fixed in 1.12.1)"
```

### 3.3 Restore custom library attachments (T313)

Using the notes captured in `planning.md`, re-attach any custom libraries and
their CSS/JS assets:

- Re-add libraries to `<subtheme>.libraries.yml` if they were removed or
  renamed.
- Restore Twig `attach_library()` and preprocess `#attached` entries.
- Clear caches and spot-check pages that rely on these libraries.

### 3.4 Rebuild sub-theme assets (T312) – optional

This step is optional but recommended to pick up the updated
`@civictheme/sdc` 1.12.1 package:

```bash
# RECOMMENDED: Using ahoy fe (equivalent to npm run build from theme directory)
ahoy fe

# Alternative: Native (non-Docker environments)
cd $SUBTHEME_PATH
npm install
npm run build

# Verify output files exist
ls -la $SUBTHEME_PATH/dist/
```

### 3.5 Commit changes

```bash
git add -A
git status  # Review all changes

git commit -m "chore: Rebuilt sub-theme assets for CivicTheme 1.12.1"
```

---

## 4. Validate

**Related tasks**: T320–T329

### 4.1 Clear caches and run updates (T320)

```bash
# Clear all Drupal caches
drush cr

# Check for and run database updates
drush updb -y
# This will run civictheme_post_update_remove_civictheme_iframe_field_c_p_attributes()
# which removes the iframe attributes field automatically

# IMPORTANT: Export configuration BEFORE importing
# This captures the field removal in the sync directory, preventing the
# stale 1.12.0 field config from being re-imported and defeating the fix.
drush cex -y

# Now import any other pending configuration changes
drush cim -y

# Check for errors
echo $?  # Should be 0
```

**Why export before import?** On config-managed sites, the sync directory may
still contain the `field_c_p_attributes` configuration from 1.12.0. Running
`drush cim` without first exporting would restore the field that the post-update
hook just removed. Exporting first ensures the field removal is captured in the
sync directory.

### 4.2 Verify CivicTheme version (T321)

```bash
composer show drupal/civictheme | grep versions
# Expected: 1.12.1
```

**Blocker / STOP CONDITION**: If this final check does **not** report
`1.12.1`, treat the upgrade as failed. Do **not** start the next CivicTheme
upgrade step or close this task until the version mismatch is understood and
corrected.

### 4.3 Test embedded media rendering (T322)

This is the key validation for this release. Navigate to pages with
embedded media and verify they render correctly.

**Test locations**:

| Content type | What to check |
|--------------|---------------|
| Pages with iframes | YouTube embeds, maps, other iframes |
| Pages with figures | Image figures, captions |
| Pages with video embeds | Embedded video players |
| WYSIWYG content | Any rich text with embedded media |

**Expected result**: Embedded media that was stripped in 1.12.0 should now
display correctly.

### 4.4 Test email handling (T323)

Navigate to pages with email addresses in body content:

| Text format | Expected behaviour |
|-------------|-------------------|
| `civictheme_rich_text` | Emails converted to `mailto:` links (filter enabled by default) |
| Custom format without URL filter | Emails remain as plain text |
| Custom format with URL filter | Emails converted to `mailto:` links |

### 4.5 Verify iframe attributes field removal (T324)

```bash
# Check that the field configuration has been removed
drush config:get field.field.paragraph.civictheme_iframe.field_c_p_attributes 2>&1
# Expected: "No matching key found"

drush config:get field.storage.paragraph.field_c_p_attributes 2>&1
# Expected: "No matching key found"
```

**Note**: If you already removed these fields manually during your 1.12.0
upgrade, this is expected behaviour. The post-update hook checks if the
config exists before attempting removal.

### 4.6 Check logs for errors (T325)

```bash
# View recent watchdog entries
drush watchdog:show --count=50

# Filter for CivicTheme-related errors
drush watchdog:show --filter="civictheme" --count=50
```

**Expected**: No new errors related to CivicTheme.

### 4.7 Validate page rendering (T326)

Open the site in a browser and manually check:

- [ ] **Home page** (T326a):
  - Header renders correctly.
  - Footer renders correctly.
  - Navigation works.
  - Content displays properly.

- [ ] **Content pages** (T326b):
  - WYSIWYG content renders correctly.
  - No missing embedded media.
  - Links work as expected.

- [ ] **Pages with iframes** (T326c):
  - Iframe paragraphs render correctly.
  - Embedded content displays.

### 4.8 Update customisation register (T327)

Edit `docs/civic-theme-upgrades/customisations.md` and update any entries
affected by this upgrade:

| Status | When to use |
|--------|-------------|
| `unchanged` | Customisation not affected by this release |
| `updated` | Customisation was modified as part of this upgrade |
| `retired` | Workaround removed because fix is now in upstream |

---

## 5. Capture outcomes and follow-ups

**Related tasks**: T328, T329

### 5.1 Document lessons learned (T328)

Add a "Lessons Learned" section if anything notable occurred:

```markdown
## Lessons Learned – CivicTheme 1.12.0 → 1.12.1

### What went well
- [List things that worked smoothly]

### Challenges encountered
- [List difficulties and how they were resolved]

### Workarounds removed
- [List any workarounds that were removed]

### Follow-up tasks
- [List any remaining work]

### Time spent
- Discovery: X minutes
- Changes: X minutes
- Validation: X minutes
- Total: X minutes
```

### 5.2 Commit all documentation

```bash
cd /path/to/drupal/project

# Add all documentation changes
git add docs/civic-theme-upgrades/

# Commit
git commit -m "docs: Completed CivicTheme 1.12.0 to 1.12.1 upgrade documentation"
```

### 5.3 Prepare for review (T329)

Create a merge request with the following information:

```markdown
## CivicTheme 1.12.0 → 1.12.1 Upgrade (Bug Fix Release)

### Summary
This PR upgrades CivicTheme from 1.12.0 to 1.12.1, addressing a bug where
embedded media was being stripped from WYSIWYG content.

### Changes Made
- [ ] Updated CivicTheme via Composer
- [ ] Ran database updates (post-update hook removed iframe attributes field)
- [ ] Rebuilt sub-theme assets (optional)
- [ ] Removed 1.12.0 workarounds (if applicable)

### Testing Performed
- [ ] Embedded media (iframes, figures) renders correctly
- [ ] WYSIWYG content displays as expected
- [ ] Email handling works correctly
- [ ] No errors in logs

### Workarounds Removed
- [List any workarounds removed, or "None"]

### Known Issues / Follow-ups
- [List any remaining issues, or "None"]

### Documentation
See `docs/civic-theme-upgrades/versions/v1.12.0-to-v1.12.1/` for full
upgrade documentation.
```

---

## Appendix: Quick reference

### Changes in 1.12.1

| Area | Change |
|------|--------|
| WYSIWYG preprocessing | Fixed filtering of embedded media (iframes, figures) |
| Email link conversion | Removed from CivicTheme; delegated to text format filters |
| iframe attributes field | Automated removal via post-update hook |
| GovCMS script | Fixed directory path (GovCMS only) |
| NPM package | Updated `@civictheme/sdc` to 1.12.1 |

### Key files changed

| File | Change |
|------|--------|
| `civictheme.post_update.php` | Added hook to remove iframe attributes field |
| `includes/link.inc` | Removed email-to-link function; updated URL processing |
| `includes/process.inc` | Added `$format` parameter to `_civictheme_process__html_content()` |
| `includes/wysiwyg.inc` | Pass text format to processing function |
| `includes/node.inc` | Updated alert node preprocessing |
| `includes/utilities.inc` | Removed `components.link.email` opt-out flag |
| `package.json` | Updated `@civictheme/sdc` to 1.12.1 |

### Post-update hook

The following post-update hook runs automatically when you execute
`drush updb`:

```php
civictheme_post_update_remove_civictheme_iframe_field_c_p_attributes()
```

This removes:
- `field.field.paragraph.civictheme_iframe.field_c_p_attributes`
- `field.storage.paragraph.field_c_p_attributes`

If these were already removed manually, the hook does nothing.

### Email-to-link conversion

| Scenario | Behaviour |
|----------|-----------|
| Using `civictheme_rich_text` format | Emails converted to links (filter enabled by default) |
| Custom format with "Convert URLs into links" filter enabled | Emails converted to links |
| Custom format without URL filter | Emails remain as plain text |

To enable email-to-link conversion in a custom text format:

1. Go to `/admin/config/content/formats`
2. Edit your text format
3. Enable "Convert URLs into links" filter
4. Save

---

This completes the runbook for the CivicTheme `1.12.0` → `1.12.1` upgrade.
For questions or issues, consult the upstream documentation at
https://docs.civictheme.io/changelog
