# CivicTheme upgrade playbook: v1.11.0 → v1.12.0

**Directory**: `docs/civic-theme-upgrades/versions/v1.11.0-to-v1.12.0/`  
**Related documents**: `spec.md`, `tasks.md`, `docs/civic-theme-upgrades/customisations.md`

This playbook turns the tasks in `tasks.md` into an ordered runbook for a
non-production upgrade of CivicTheme from `1.11.0` to `1.12.0`.

**Important**: This is a **security release**. The primary driver for
upgrading is the fixes for SA-CONTRIB-2025-112 (Information disclosure)
and SA-CONTRIB-2025-113 (Cross-site Scripting). Prioritise accordingly.

---

## 1. Environment checks

**Related tasks**: T200, T220

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

```bash
# Check if ahoy is available
which ahoy && cat .ahoy.yml | head -20

# Example: Running drush via different methods
# Option A: Ahoy (recommended if available)
ahoy drush status

# Option B: Docker compose directly
docker compose exec cli drush status

# Option C: Native (non-Docker environments)
drush status
```

**Throughout this playbook**, commands are shown in native form. Prefix with
`ahoy` or `docker compose exec cli` as appropriate for your environment.

### 1.3 Create backups

- [ ] Database backup created and verified.
- [ ] Files backup created (if applicable).
- [ ] Able to restore to pre-upgrade state if needed.

### 1.4 Create dedicated feature branch

```bash
cd /path/to/drupal/project
git checkout develop  # or main
git pull origin develop
git checkout -b feature/civictheme-1.12-upgrade
```

### 1.5 Verify Drupal core version

```bash
# Check Drupal core version (use ahoy/docker as needed)
drush status --field=drupal-version

# Or via composer
composer show drupal/core | grep versions
```

**Requirement**: Drupal core must meet `^10.2 || ^11` (unchanged from 1.11.0).

### 1.6 Verify current CivicTheme version

```bash
composer show drupal/civictheme | grep versions
# Expected: 1.11.x
```

### 1.7 Discover available front-end commands

```bash
# Check for ahoy front-end commands
if [ -f ".ahoy.yml" ]; then
  echo "=== Ahoy front-end commands ==="
  grep -E "^\s*(fe|front|npm|build|storybook)" .ahoy.yml | head -20
  
  echo "=== Checking 'fe' command details ==="
  grep -A 10 "^\s*fe:" .ahoy.yml
fi
```

**Document discovered front-end commands**:

| Command | Description | Usage |
|---------|-------------|-------|
| `ahoy fe` | Full front-end build (standalone) | Runs npm install + build automatically |
| `ahoy fe <cmd>` | Run specific npm command | `ahoy fe npm run storybook` |

### 1.8 Discover available test commands

```bash
# Check for ahoy test commands
if [ -f ".ahoy.yml" ]; then
  echo "=== Ahoy test commands ==="
  grep -E "^\s*test" .ahoy.yml || grep -i "test" .ahoy.yml | head -20
fi

# Check for composer test scripts
echo "=== Composer test scripts ==="
grep -A 30 '"scripts"' composer.json | grep -i "test" || echo "No test scripts found"
```

### 1.9 Run tests BEFORE upgrade (baseline)

**Critical**: Establish that tests pass before making any changes.

```bash
# Run all available tests (adjust commands based on 1.8 discovery)
ahoy test-unit
ahoy test-bdd
composer test
```

Record test results:

- [ ] Unit tests: PASS / FAIL / N/A
- [ ] BDD tests: PASS / FAIL / N/A  
- [ ] E2E tests: PASS / FAIL / N/A
- [ ] Other tests: PASS / FAIL / N/A

**STOP CONDITION**: If critical tests fail before the upgrade, resolve those
issues first. Do not proceed with upgrade on a broken test baseline.

### 1.10 Review security advisories

Before proceeding, review the security context:

- [ ] Read SA-CONTRIB-2025-112 (Information disclosure).
- [ ] Read SA-CONTRIB-2025-113 (Cross-site Scripting).
- [ ] Understand which components are affected.
- [ ] Note any sub-theme overrides that may need security review.

---

## 2. Discovery

**Related tasks**: T200, T201, T202, T203, T204, T205

### 2.1 Refresh customisation register (T202)

Open `docs/civic-theme-upgrades/customisations.md` and ensure all
CivicTheme-related customisations are documented with stable IDs.

### 2.2 Audit sub-theme for security-relevant overrides (T203)

Run these commands in the sub-theme directory to identify templates that
may need security review:

```bash
SUBTHEME_PATH=/path/to/your/subtheme

# T203a: Find templates handling user content (security review needed)
echo "=== Templates with content/field rendering ==="
grep -rn "{{ content" $SUBTHEME_PATH/templates/
grep -rn "{{ field" $SUBTHEME_PATH/templates/

echo "=== Templates using raw filter (HIGH RISK) ==="
grep -rn "{{ raw" $SUBTHEME_PATH/templates/
grep -rn "|raw" $SUBTHEME_PATH/templates/

# T203b: Find menu-related overrides
echo "=== Menu-related templates ==="
grep -rn "menu" $SUBTHEME_PATH/templates/
ls $SUBTHEME_PATH/templates/*menu* 2>/dev/null

# T203c: Find attachment component overrides
echo "=== Attachment/file-related templates ==="
grep -rn "attachment" $SUBTHEME_PATH/templates/
grep -rn "file" $SUBTHEME_PATH/templates/
```

**Record the output**. Flag any templates using `|raw` filter on user data
for immediate security review.

### 2.3 Audit build tooling and CSS variables (T204)

```bash
SUBTHEME_PATH=/path/to/your/subtheme

# Check current SCSS variable usage
echo "=== SCSS variables (potential migration to CSS custom properties) ==="
grep -rn "\$ct-" $SUBTHEME_PATH/scss/ | head -20

# Check if already using CSS custom properties
echo "=== CSS custom properties usage ==="
grep -rn "var(--ct-" $SUBTHEME_PATH/scss/ | head -20
```

### 2.4 Document findings (T205)

Update the customisation register with:

- List of templates requiring security review.
- List of menu-related overrides.
- List of attachment-related overrides.
- Current CSS/SCSS variable usage.

Assign impact levels:

- **HIGH**: Templates with `|raw` filter, user content handling.
- **MEDIUM**: Menu overrides, attachment overrides.
- **LOW**: CSS variable migration candidates.

---

## 3. Apply changes

**Related tasks**: T210–T215

### 3.1 Upgrade CivicTheme via Composer (T210)

```bash
cd /path/to/drupal/project

# Update CivicTheme
composer require drupal/civictheme:^1.12

# Verify the update
composer show drupal/civictheme | grep versions
# Expected: 1.12.x

# Commit changes
git add composer.json composer.lock
git commit -m "feat: Updated CivicTheme from 1.11.0 to 1.12.x (security release)"
```

### 3.2 Security review of sub-theme overrides (T211) – HIGH PRIORITY

This is the most critical step for this upgrade.

**For each template identified in Section 2.2 that handles user content**:

1. **Locate the upstream CivicTheme 1.12.0 template**:
   ```bash
   CIVICTHEME=/path/to/themes/contrib/civictheme
   # Example: find a specific template
   find $CIVICTHEME/templates -name "*.twig" | grep <template-name>
   ```

2. **Compare with sub-theme override**:
   ```bash
   diff -u $CIVICTHEME/templates/<path>/<template>.twig \
          $SUBTHEME_PATH/templates/<template>.twig
   ```

3. **Check for security-related changes** in the upstream template:
   - Look for changes to output escaping.
   - Look for removal of `|raw` filters.
   - Look for added sanitisation.

4. **Apply equivalent fixes to sub-theme** if the override circumvents
   security measures.

**Key areas to review**:

- [ ] Form input handling templates.
- [ ] Search result display templates.
- [ ] Comment/user content display templates.
- [ ] Any template using `|raw` on user data.

### 3.3 Update menu customisations (T212) – if applicable

If menu overrides were identified in T203b:

```bash
CIVICTHEME=/path/to/themes/contrib/civictheme
SUBTHEME_PATH=/path/to/your/subtheme

# Compare menu templates
diff -u $CIVICTHEME/templates/navigation/menu*.twig \
       $SUBTHEME_PATH/templates/navigation/menu*.twig 2>/dev/null
```

**Key changes in 1.12.0**:

- Enhanced menu theming system.
- Fixed `menu_level_modifier_class` scope in Twig.
- Fixed mobile menu visibility on non-drawer/dropdown links.

If your override affects any of these areas, update accordingly.

### 3.4 Update attachment overrides (T213) – if applicable

If attachment overrides were identified in T203c:

```bash
# Compare attachment templates
diff -u $CIVICTHEME/templates/*/attachment*.twig \
       $SUBTHEME_PATH/templates/*/attachment*.twig 2>/dev/null
```

**Key change in 1.12.0**:

- Fixed attachment component not resetting file extension from previous
  items.

If your override handles file extensions, verify it works correctly with
this fix.

### 3.5 Consider CSS variable migration (T214) – optional

If your sub-theme uses SCSS variables extensively, consider migrating to
CSS custom properties:

```scss
/* Before (SCSS variable) */
.my-component {
  color: $ct-primary-color;
}

/* After (CSS custom property) */
.my-component {
  color: var(--ct-primary-color);
}
```

**Benefits**:

- Runtime customisation without SCSS recompilation.
- Easier theming via browser dev tools.
- Better alignment with modern CSS practices.

**Note**: This is optional and can be deferred to a separate task if time
is constrained. The security fixes are the priority for this upgrade.

### 3.6 Rebuild sub-theme assets (T215)

```bash
# RECOMMENDED: Using ahoy fe (standalone - does everything)
ahoy fe

# Alternative: Using ahoy fe with specific commands
ahoy fe npm install
ahoy fe npm run build

# Alternative: Native (non-Docker environments)
cd $SUBTHEME_PATH
npm install
npm run build

# Verify output files exist
ls -la $SUBTHEME_PATH/dist/
```

### 3.7 Commit changes

```bash
git add -A
git status  # Review all changes

# Commit in logical chunks
git commit -m "security: Updated sub-theme for CivicTheme 1.12 security fixes"
```

---

## 4. Validate

**Related tasks**: T220–T229

### 4.1 Clear caches and run updates (T220)

```bash
# Clear all Drupal caches
drush cr

# Check for and run database updates
drush updb -y

# Import configuration if pending
drush cim -y

# Check for errors
echo $?  # Should be 0
```

### 4.2 Verify security fixes are applied (T221)

```bash
# Confirm CivicTheme version
composer show drupal/civictheme | grep versions
# Expected: 1.12.x

# If security advisory test cases are available, run them
# (Consult security advisory documentation for specific tests)
```

### 4.3 Check logs for errors (T222)

```bash
# View recent watchdog entries
drush watchdog:show --count=50

# Filter for CivicTheme-related errors
drush watchdog:show --filter="civictheme" --count=50
drush watchdog:show --filter="component" --count=50
```

**Expected**: No errors related to CivicTheme or components.

### 4.4 Validate page rendering (T223)

Open the site in a browser and manually check:

- [ ] **Home page** (T223a):
  - Header renders correctly.
  - Footer renders correctly.
  - Navigation works.
  - Content displays properly.

- [ ] **Navigation** (T223b):
  - Primary navigation works.
  - Secondary navigation works.
  - Mobile menu opens and closes.
  - Mobile menu items visible (bug fix in 1.12.0).
  - Menu links navigate correctly.

- [ ] **Search** (T223c):
  - Search form displays.
  - Search results return.
  - **Inline filter**: Keyword displays in search results message.

- [ ] **Content pages** (T223d):
  - Article pages render.
  - Landing pages render.
  - Page content types render.

- [ ] **Attachments** (T223e):
  - Pages with file attachments display correctly.
  - File extensions display correctly (bug fix in 1.12.0).
  - Multiple attachments show correct extensions for each file.

- [ ] **Forms** (T223f):
  - Contact forms display.
  - Webforms display (bug fix for archived webforms in 1.12.0).
  - Webform messages render correctly (striptags fix in 1.12.0).
  - Form submissions work.

### 4.5 Validate specific components (T224)

- [ ] **Menus** (T224a):
  - Desktop menu renders correctly.
  - Mobile menu visibility is correct.
  - Menu level classes apply correctly.

- [ ] **Cards** (T224b):
  - Card components render with accessibility updates.
  - Card interactions work as expected.

- [ ] **Videos** (T224c):
  - If using video with transcripts, verify transcript functionality.

- [ ] **Links** (T224d):
  - Email addresses in content are NOT incorrectly converted to links.
  - Actual links work correctly.

- [ ] **Logo** (T224e):
  - If logo path contains spaces, verify it displays correctly.

### 4.6 Test new features (T225) – optional, if adopting

- [ ] **Multi-line header** (T225a):
  - Enable via theme settings.
  - Verify logo displays in container above navigation.
  - Verify navigation uses full container width.
  - Test with more than 4 navigation items.

- [ ] **Message organism** (T225b):
  - Create content using Message component.
  - Test all four styling options.
  - Verify accessibility.

- [ ] **Fast fact card** (T225c):
  - Create content using Fast fact card.
  - Verify display and styling.

### 4.7 Test sub-theme build (T226)

```bash
cd $SUBTHEME_PATH

# Run full build
ahoy fe
# Or: npm run build

# Check exit code
echo $?  # Should be 0

# Verify output files
ls -la dist/
```

### 4.8 Run tests AFTER upgrade (T226b)

Re-run the same tests discovered in Section 1.8 to verify the upgrade has
not introduced regressions.

```bash
# Run all available tests
ahoy test-unit
ahoy test-bdd
composer test
```

**Compare results with pre-upgrade baseline (Section 1.9)**:

| Test Suite | Before Upgrade | After Upgrade | Status |
|------------|----------------|---------------|--------|
| Unit tests | PASS/FAIL | PASS/FAIL | ✅/❌ |
| BDD tests | PASS/FAIL | PASS/FAIL | ✅/❌ |
| E2E tests | PASS/FAIL | PASS/FAIL | ✅/❌ |
| Other | PASS/FAIL | PASS/FAIL | ✅/❌ |

**Action required if tests fail**:

1. If a test passed before and fails after → **Regression introduced**.
   Investigate and fix before proceeding.
2. If a test failed before and fails after → **Pre-existing issue**.
   Document but do not block upgrade.
3. If a test failed before and passes after → **Bonus improvement**.
   Document the fix.

### 4.9 Document validation results

Create a checklist of validation results:

```markdown
## Validation Results – CivicTheme 1.11.0 → 1.12.0

Date: YYYY-MM-DD
Tester: [Name]

### Security Review
- [ ] Security advisories reviewed
- [ ] Sub-theme overrides audited for vulnerabilities
- [ ] No reintroduction of patched vulnerabilities

### Test Suite Results
| Test Suite | Before Upgrade | After Upgrade | Notes |
|------------|----------------|---------------|-------|
| Unit tests | PASS/FAIL | PASS/FAIL | |
| BDD tests | PASS/FAIL | PASS/FAIL | |
| E2E tests | PASS/FAIL | PASS/FAIL | |

### Page Rendering
- [ ] Home page: PASS/FAIL
- [ ] Navigation: PASS/FAIL
- [ ] Search (inline filter): PASS/FAIL
- [ ] Content pages: PASS/FAIL
- [ ] Attachments: PASS/FAIL
- [ ] Forms/Webforms: PASS/FAIL

### Components
- [ ] Menus: PASS/FAIL
- [ ] Cards: PASS/FAIL
- [ ] Videos/Transcripts: PASS/FAIL
- [ ] Links (email fix): PASS/FAIL
- [ ] Logo: PASS/FAIL

### Build
- [ ] npm run build: PASS/FAIL
- [ ] Storybook: PASS/FAIL (if applicable)

### Issues Found
1. [Description of any issues]

### Notes
[Any additional observations]
```

---

## 5. Capture outcomes and follow-ups

**Related tasks**: T227, T228, T229

### 5.1 Update customisation register (T227)

Edit `docs/civic-theme-upgrades/customisations.md` and update each entry
affected by this upgrade:

```markdown
| ID | Description | Status | Notes |
|----|-------------|--------|-------|
| C001 | Menu override | Updated | Verified compatible with 1.12.0 |
| C002 | Attachment template | Reviewed | No changes needed |
| C003 | Search results | Security reviewed | No vulnerabilities found |
```

### 5.2 Document lessons learned (T228)

Add a "Lessons Learned" section:

```markdown
## Lessons Learned – CivicTheme 1.11.0 → 1.12.0

### What went well
- [List things that worked smoothly]

### Challenges encountered
- [List difficulties and how they were resolved]

### Security review findings
- [Document any security-related discoveries]

### Follow-up tasks
- [ ] CSS variable migration (if deferred)
- [ ] Adopt new features (Message organism, Fast fact card, etc.)

### Time spent
- Discovery: X hours
- Changes: X hours
- Validation: X hours
- Documentation: X hours
- Total: X hours
```

### 5.3 Commit all documentation

```bash
cd /path/to/drupal/project

# Add all documentation changes
git add docs/civic-theme-upgrades/

# Commit
git commit -m "docs: Completed CivicTheme 1.11.0 to 1.12.0 upgrade documentation"
```

### 5.4 Prepare for review (T229)

Create a merge request with the following information:

```markdown
## CivicTheme 1.11.0 → 1.12.0 Upgrade (Security Release)

### Summary
This PR upgrades CivicTheme from 1.11.0 to 1.12.0, addressing security
vulnerabilities SA-CONTRIB-2025-112 and SA-CONTRIB-2025-113.

### Security
- [ ] Security advisories reviewed
- [ ] Sub-theme overrides audited
- [ ] No reintroduction of patched vulnerabilities

### Changes Made
- Updated CivicTheme via Composer
- Reviewed X sub-theme template overrides for security
- Updated menu customisations (if applicable)
- Updated attachment overrides (if applicable)

### Customisations Impacted
- C001: [Description]
- C002: [Description]

### Testing Performed
- [ ] All pages render correctly
- [ ] Navigation works (including mobile menu fix)
- [ ] Search returns correct results with inline filter
- [ ] Attachments display correct file extensions
- [ ] Forms/webforms function properly

### Known Issues / Follow-ups
- [List any remaining issues]

### Documentation
See `docs/civic-theme-upgrades/versions/v1.11.0-to-v1.12.0/` for full
upgrade documentation.
```

---

## Appendix: Quick reference

### New features in 1.12.0

| Feature | Description | Adoption |
|---------|-------------|----------|
| Multi-line header | Logo above, full-width navigation | Optional configuration |
| Message organism | Highlight messages with 4 styling options | Optional new component |
| Inline filter | Keyword shown in search results | Automatic |
| Fast fact card | Simple fact display card | Optional new component |
| GovCMS automation | Setup script for GovCMS | GovCMS projects only |

### Bug fixes in 1.12.0

| Issue | Fix |
|-------|-----|
| Menu theming | Enhanced menu theming system |
| Archived webforms | Site no longer breaks on archived webform nodes |
| Search form | Search form element issues resolved |
| Email links | Email addresses not incorrectly converted to links |
| Logo paths | Spaces in logo paths handled correctly |
| Webform messages | Striptags error fixed |
| Menu Twig scope | `menu_level_modifier_class` now in scope |
| Attachment extension | File extension resets correctly between items |
| Mobile menu | Visibility fixed for non-drawer/dropdown links |

### Security advisories addressed

| Advisory | Type | Severity |
|----------|------|----------|
| SA-CONTRIB-2025-112 | Information disclosure | Moderately critical |
| SA-CONTRIB-2025-113 | Cross-site Scripting | Moderately critical |

### CSS custom properties (new in 1.12.0)

CivicTheme 1.12.0 adds CSS versions of SCSS variables. Consider migrating:

```scss
/* SCSS variable (still works) */
$ct-primary-color

/* CSS custom property (new) */
var(--ct-primary-color)
```

---

This completes the runbook for the CivicTheme `1.11.0` → `1.12.0` upgrade.
For questions or issues, consult the upstream documentation at
https://docs.civictheme.io/changelog

