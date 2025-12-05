# CivicTheme upgrade playbook: v1.12.1 → v1.12.2

**Directory**: `docs/civic-theme-upgrades/versions/v1.12.1-to-v1.12.2/`  
**Related documents**: `spec.md`, `tasks.md`, `docs/civic-theme-upgrades/customisations.md`

This playbook turns the tasks in `tasks.md` into an ordered runbook for a
non-production upgrade of CivicTheme from `1.12.1` to `1.12.2`.

**Important**: This is a **bug-fix release with breaking changes**. The
removal of the `|raw` Twig filter requires sub-theme template and
preprocessing updates.

---

## 1. Environment checks

**Related tasks**: T400

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
git checkout -b feature/civictheme-1.12.2-upgrade
```

### 1.6 Verify current CivicTheme version

```bash
composer show drupal/civictheme | grep versions
# Expected: 1.12.1
```

**STOP CONDITION**: If this command does not report `1.12.1` exactly, do **not**
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

**Requirement**: Drupal core must meet `^10.2 || ^11` (unchanged from 1.12.1).

---

## 2. Discovery

**Related tasks**: T401, T402, T403, T404, T405

### 2.1 Audit sub-theme Twig templates for `|raw` usage (T401)

**This is critical for this upgrade.** Search for all `|raw` filter usages:

```bash
SUBTHEME_PATH=/path/to/your/subtheme

echo "=== Searching for |raw in templates ==="
grep -rn "|raw" $SUBTHEME_PATH/templates/ 2>/dev/null
grep -rn "| raw" $SUBTHEME_PATH/templates/ 2>/dev/null

echo "=== Searching for |raw in components ==="
grep -rn "|raw" $SUBTHEME_PATH/components/ 2>/dev/null
grep -rn "| raw" $SUBTHEME_PATH/components/ 2>/dev/null

echo "=== Count of occurrences ==="
grep -r "|raw\|| raw" $SUBTHEME_PATH/templates/ $SUBTHEME_PATH/components/ 2>/dev/null | wc -l
```

**Document each occurrence** in `planning.md` with:

| File | Line | Variable | Action needed |
|------|------|----------|---------------|
|      |      |          |               |

### 2.2 Audit preprocessing for raw HTML strings (T402)

Search for patterns that may need Markup object conversion:

```bash
SUBTHEME_PATH=/path/to/your/subtheme

echo "=== Checking .theme file ==="
cat $SUBTHEME_PATH/*.theme 2>/dev/null | head -100

echo "=== Checking includes directory ==="
ls -la $SUBTHEME_PATH/includes/ 2>/dev/null

echo "=== Searching for HTML string patterns ==="
grep -rn "\$variables\[" $SUBTHEME_PATH/*.theme $SUBTHEME_PATH/includes/ 2>/dev/null | grep -E "(<[a-z]|'>|\">)" | head -50
```

**Look for patterns like**:

```php
// These patterns need updating
$variables['title'] = '<span>' . $value . '</span>';
$variables['content'] = t('Hello <strong>@name</strong>', ['@name' => $name]);
```

**Document each function** needing updates in `planning.md`.

### 2.3 Check for overridden CivicTheme components (T403)

```bash
SUBTHEME_PATH=/path/to/your/subtheme

echo "=== Overridden templates ==="
ls -la $SUBTHEME_PATH/templates/ 2>/dev/null

echo "=== Overridden components ==="
ls -la $SUBTHEME_PATH/components/ 2>/dev/null

echo "=== Counting overrides ==="
find $SUBTHEME_PATH/templates/ $SUBTHEME_PATH/components/ -name "*.twig" 2>/dev/null | wc -l
```

**Components to pay special attention to**:

- `promo-card.twig` (title rendering)
- `button.twig` (title rendering)
- `navigation-*` templates (menu item rendering)
- `breadcrumb.twig` (item rendering)

### 2.4 Refresh customisation register (T404)

Review `docs/civic-theme-upgrades/customisations.md` and ensure it's up to
date. For this release, focus on:

- Any templates using `|raw` filter.
- Any preprocessing returning HTML strings.
- Any overridden components.

### 2.5 Capture custom library attachments (T405)

```bash
SUBTHEME_PATH=/path/to/your/subtheme

echo "=== attach_library / #attached usage ==="
grep -rn "attach_library" $SUBTHEME_PATH/templates/ $SUBTHEME_PATH | head -100
grep -rn "#attached" $SUBTHEME_PATH | head -100

echo "=== Custom libraries defined ==="
grep -E "^[A-Za-z0-9_.-]+:" $SUBTHEME_PATH/*.libraries.yml | head -50

# Record library name, files, and attach locations in
# docs/civic-theme-upgrades/versions/v1.12.1-to-v1.12.2/planning.md to restore
# them after the upgrade if needed.
```

---

## 3. Apply changes

**Related tasks**: T410, T411, T412, T413, T414, T415

### 3.1 Upgrade CivicTheme via Composer (T410)

```bash
cd /path/to/drupal/project

# Update CivicTheme to exact target version
# IMPORTANT: Use exact version constraint (no ^ or ~) to ensure
# sequential upgrades install precisely the intended release.
composer require drupal/civictheme:1.12.2

# Verify the update
composer show drupal/civictheme | grep versions
# Expected: 1.12.2 (exact version)

# Commit changes
git add composer.json composer.lock
git commit -m "chore: Updated CivicTheme from 1.12.1 to 1.12.2"
```

**STOP CONDITION**: After running the above commands, if
`composer show drupal/civictheme | grep versions` does **not** report
`1.12.2`, treat the upgrade as incomplete. Do **not** proceed with later
steps or start the next CivicTheme upgrade until the version mismatch is
resolved.

### 3.2 Update sub-theme Twig templates (T411)

For each template identified in Section 2.1, remove the `|raw` filter:

**Example changes**:

```twig
{# Before (1.12.1) #}
<h1>{{ title|raw }}</h1>
<div class="content">{{ content|raw }}</div>

{# After (1.12.2) #}
<h1>{{ title }}</h1>
<div class="content">{{ content }}</div>
```

**Important considerations**:

1. **Simple variable output**: Just remove `|raw`.
2. **HTML content**: Ensure the preprocessing passes a Markup object.
3. **User-generated content**: Should already be escaped by Drupal.

```bash
# After updating templates, commit
git add -A
git commit -m "style: Removed |raw filter from sub-theme templates (CivicTheme 1.12.2)"
```

### 3.3 Update preprocessing to use Markup objects (T412)

For each function identified in Section 2.2, update to use Markup objects:

**Example changes**:

```php
<?php
// Before (1.12.1)
function MYTHEME_preprocess_node(&$variables) {
  $title = $variables['node']->getTitle();
  $variables['custom_title'] = '<span class="custom">' . $title . '</span>';
}

// After (1.12.2)
use Drupal\Core\Render\Markup;
use Drupal\Component\Utility\Html;

function MYTHEME_preprocess_node(&$variables) {
  $title = $variables['node']->getTitle();
  // Escape user content, wrap in Markup for trusted HTML structure
  $variables['custom_title'] = Markup::create(
    '<span class="custom">' . Html::escape($title) . '</span>'
  );
}
```

**Security guidelines**:

| Content type | How to handle |
|--------------|---------------|
| User-provided text | Use `Html::escape()` |
| Trusted HTML (e.g. from config) | Use `Markup::create()` directly |
| Mixed content | Escape user parts, wrap whole in `Markup::create()` |
| Renderable arrays | Return array instead of string (preferred) |

**Add imports to the top of your .theme file**:

```php
use Drupal\Core\Render\Markup;
use Drupal\Component\Utility\Html;
```

```bash
# After updating preprocessing, commit
git add -A
git commit -m "refactor: Updated preprocessing to use Markup objects (CivicTheme 1.12.2)"
```

### 3.4 Verify overridden component compatibility (T413)

For each overridden component from Section 2.3:

1. **Compare with upstream template**:
   ```bash
   # Compare your template with the upstream 1.12.2 version
   diff $SUBTHEME_PATH/templates/component.twig \
        docroot/themes/contrib/civictheme/templates/component.twig
   ```

2. **Merge any upstream changes** that are relevant.

3. **Remove any `|raw` filters** in overridden templates.

4. **Document conflicts** in `planning.md`.

### 3.5 Restore custom library attachments (T415)

Using the notes captured in `planning.md`, re-attach any custom libraries and
their CSS/JS assets:

- Re-add libraries to `<subtheme>.libraries.yml` if they were removed or
  renamed.
- Restore Twig `attach_library()` and preprocess `#attached` entries.
- Clear caches and spot-check pages that rely on these libraries.

### 3.6 Rebuild sub-theme assets (T414)

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

### 3.7 Commit all changes

```bash
git add -A
git status  # Review all changes

git commit -m "chore: Rebuilt sub-theme assets for CivicTheme 1.12.2"
```

---

## 4. Validate

**Related tasks**: T420–T429

### 4.1 Clear caches and run updates (T420)

```bash
# Clear all Drupal caches
drush cr

# Check for and run database updates
drush updb -y

# Export configuration (captures any changes)
drush cex -y

# Import any pending configuration changes
drush cim -y

# Check for errors
echo $?  # Should be 0
```

### 4.2 Verify CivicTheme version (T421)

```bash
composer show drupal/civictheme | grep versions
# Expected: 1.12.2
```

**Blocker / STOP CONDITION**: If this final check does **not** report
`1.12.2`, treat the upgrade as failed. Do **not** start the next CivicTheme
upgrade step or close this task until the version mismatch is understood and
corrected.

### 4.3 Test template rendering (T422)

Navigate to pages with overridden templates and verify:

- [ ] HTML content renders correctly (not as escaped text).
- [ ] No `&lt;`, `&gt;`, or `&amp;` appearing in rendered output.
- [ ] No unescaped user content (security check).

### 4.4 Test special character rendering (T423)

This is a key validation for this release. Test content with special
characters:

| Content type | Test case | Expected result |
|--------------|-----------|-----------------|
| Promo card title | "News & Events" | Displays as "News & Events" |
| Button title | "Save & Continue" | Displays as "Save & Continue" |
| Menu item | "Products & Services" | Displays correctly |
| Breadcrumb | "FAQ & Help" | Displays correctly |
| Title with quotes | 'Welcome to "Our Site"' | Displays correctly |

**Create or find test content** with these special characters and verify.

### 4.5 Test overridden component rendering (T424)

For each overridden component:

| Component | Test URL | Status | Notes |
|-----------|----------|--------|-------|
|           |          |        |       |

- [ ] All overridden templates render correctly.
- [ ] No browser console errors.

### 4.6 Check logs for errors (T425)

```bash
# View recent watchdog entries
drush watchdog:show --count=50

# Filter for CivicTheme-related errors
drush watchdog:show --filter="civictheme" --count=50

# Check for Twig errors
drush watchdog:show --filter="twig" --count=50
```

**Expected**: No new errors related to CivicTheme or Twig rendering.

### 4.7 Validate page rendering (T426)

Open the site in a browser and manually check:

- [ ] **Home page** (T426a):
  - Header renders correctly.
  - Footer renders correctly.
  - Navigation works.
  - Content displays properly.

- [ ] **Content pages** (T426b):
  - Various component types render correctly.
  - No escaped HTML visible.

- [ ] **Promo cards** (T426c):
  - Titles with special characters display correctly.
  - Links work.

- [ ] **Buttons** (T426d):
  - Labels display correctly.
  - Special characters in labels render properly.

- [ ] **Navigation** (T426e):
  - Menu items display correctly.
  - Dropdowns work.

- [ ] **Breadcrumbs** (T426f):
  - Items display correctly.
  - Links work.

### 4.8 Update customisation register (T427)

Edit `docs/civic-theme-upgrades/customisations.md` and update any entries
affected by this upgrade:

| Status | When to use |
|--------|-------------|
| `unchanged` | Customisation not affected by this release |
| `updated` | Template/preprocessing was modified for `\|raw` removal |
| `retired` | Customisation no longer needed |

### 4.9 Verify no XSS vulnerabilities introduced

After removing `|raw` filters, ensure:

- [ ] User-generated content is properly escaped.
- [ ] Only trusted content uses `Markup::create()`.
- [ ] HTML injection is not possible in any fields.

---

## 5. Capture outcomes and follow-ups

**Related tasks**: T428, T429

### 5.1 Document lessons learned (T428)

Add a "Lessons Learned" section if anything notable occurred:

```markdown
## Lessons Learned – CivicTheme 1.12.1 → 1.12.2

### What went well
- [List things that worked smoothly]

### Challenges encountered
- [List difficulties and how they were resolved]

### Templates updated
- [List templates where |raw was removed]

### Preprocessing updated
- [List functions updated to use Markup objects]

### Follow-up tasks
- [List any remaining work]

### Time spent
- Discovery: X minutes
- Template updates: X minutes
- Preprocessing updates: X minutes
- Validation: X minutes
- Total: X minutes
```

### 5.2 Commit all documentation

```bash
cd /path/to/drupal/project

# Add all documentation changes
git add docs/civic-theme-upgrades/

# Commit
git commit -m "docs: Completed CivicTheme 1.12.1 to 1.12.2 upgrade documentation"
```

### 5.3 Prepare for review (T429)

Create a merge request with the following information:

```markdown
## CivicTheme 1.12.1 → 1.12.2 Upgrade

### Summary
This PR upgrades CivicTheme from 1.12.1 to 1.12.2, addressing the removal
of the `|raw` Twig filter from all components.

### Breaking Changes Addressed
- [ ] Removed `|raw` filter from X sub-theme templates
- [ ] Updated X preprocessing functions to use Markup objects

### Changes Made
- [ ] Updated CivicTheme via Composer
- [ ] Updated sub-theme templates to remove `|raw`
- [ ] Updated preprocessing to use Markup objects
- [ ] Rebuilt sub-theme assets

### Testing Performed
- [ ] Special character rendering (& < > ") verified
- [ ] Promo card titles render correctly
- [ ] Button titles render correctly
- [ ] Menu items render correctly
- [ ] Breadcrumbs render correctly
- [ ] No XSS vulnerabilities introduced
- [ ] No errors in logs

### Templates Updated
| File | Change made |
|------|-------------|
| [list templates] | Removed `\|raw` |

### Preprocessing Updated
| Function | Change made |
|----------|-------------|
| [list functions] | Added Markup::create() |

### Known Issues / Follow-ups
- [List any remaining issues, or "None"]

### Documentation
See `docs/civic-theme-upgrades/versions/v1.12.1-to-v1.12.2/` for full
upgrade documentation.
```

---

## Appendix A: Quick reference

### Changes in 1.12.2

| Area | Change | Action required |
|------|--------|-----------------|
| Twig `\|raw` filter | Removed from all components | **Yes**: Update sub-theme templates |
| Preprocessing | Return Markup objects | **Yes**: Update functions returning HTML |
| Promo card | Fixed ampersand rendering | None (upstream fix) |
| Button | Fixed title rendering | None (upstream fix) |
| Menu items | Fixed entity escaping | None (upstream fix) |
| Breadcrumbs | Added Markup objects | Review if overridden |
| Starter kit | Added packaging info removal | None (new sub-themes only) |
| NPM package | Updated to 1.12.2 | Rebuild assets |

### Key commits

| Commit | Description |
|--------|-------------|
| `bf21757d` | Updated preprocessors to support UI Kit raw removal |
| `619b5a23` | Added Markup objects to breadcrumbs and title |
| `bf9854c4` | Fixed rendering of HTML entities in string fields |
| `4c6b193e` | Fixed button title ampersand bug |
| `5a35e1aa` | Added feature to remove Drupal packaging info in starter kit |

### Files commonly affected

| File pattern | What to check |
|--------------|---------------|
| `*.twig` | Remove `\|raw` filters |
| `*.theme` | Add Markup imports, update string returns |
| `includes/*.php` | Add Markup imports, update preprocessing |
| `*.libraries.yml` | Verify library definitions intact |

### Markup object quick reference

```php
use Drupal\Core\Render\Markup;
use Drupal\Component\Utility\Html;

// Trusted HTML (e.g. from code)
$safe = Markup::create('<strong>Bold text</strong>');

// User content (MUST escape)
$safe = Markup::create('<span>' . Html::escape($userInput) . '</span>');

// Already rendered HTML (from Drupal render)
$safe = \Drupal::service('renderer')->render($render_array);
// (render() returns MarkupInterface)

// Preferred: Return renderable array
$variables['content'] = [
  '#markup' => '<span>' . Html::escape($text) . '</span>',
];
```

---

## Appendix B: Common `|raw` removal patterns

### Pattern 1: Simple variable output

```twig
{# Before #}
{{ title|raw }}

{# After #}
{{ title }}
```

**Preprocessing change**:

```php
// Before
$variables['title'] = '<span>' . $title . '</span>';

// After
$variables['title'] = Markup::create('<span>' . Html::escape($title) . '</span>');
```

### Pattern 2: Conditional with raw

```twig
{# Before #}
{% if content %}
  {{ content|raw }}
{% endif %}

{# After #}
{% if content %}
  {{ content }}
{% endif %}
```

### Pattern 3: Link with HTML

```twig
{# Before #}
<a href="{{ url }}">{{ link_text|raw }}</a>

{# After #}
<a href="{{ url }}">{{ link_text }}</a>
```

**Preprocessing change** (if link_text contains HTML):

```php
$variables['link_text'] = Markup::create($html_content);
```

### Pattern 4: Translation with HTML

```php
// Before
$variables['message'] = t('Welcome to <strong>@site</strong>', ['@site' => $site_name]);

// After (if you need HTML in translation)
$variables['message'] = Markup::create(
  t('Welcome to <strong>@site</strong>', ['@site' => Html::escape($site_name)])
);
```

---

This completes the runbook for the CivicTheme `1.12.1` → `1.12.2` upgrade.
For questions or issues, consult the upstream documentation at
https://docs.civictheme.io/changelog

