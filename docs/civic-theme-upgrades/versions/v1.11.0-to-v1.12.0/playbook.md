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
git checkout -b feature/civictheme-1.12-upgrade
```

### 1.6 Verify Drupal core version

```bash
# Check Drupal core version (use ahoy/docker as needed)
drush status --field=drupal-version

# Or via composer
composer show drupal/core | grep versions
```

**Requirement**: Drupal core must meet `^10.2 || ^11` (unchanged from 1.11.0).

### 1.7 Verify current CivicTheme version

```bash
composer show drupal/civictheme | grep versions
# Expected: 1.11.x
```

### 1.8 Discover available front-end commands

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
| `ahoy fe` | Front-end build (equivalent to `npm run build` from theme directory) | Can be invoked from project root |

### 1.9 Discover available test commands

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

### 1.10 Run tests BEFORE upgrade (baseline)

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

### 1.11 Review security advisories

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

### 2.2 Audit sub-theme Twig templates for XSS vulnerabilities (T203)

Run these commands in the sub-theme directory to identify templates that
need security updates:

```bash
SUBTHEME_PATH=/path/to/your/subtheme

# T203a: Find ALL uses of |raw filter (HIGH RISK - XSS vulnerability)
echo "=== Templates using |raw filter (CRITICAL) ==="
grep -rn "|raw" $SUBTHEME_PATH/templates/
grep -rn "|raw" $SUBTHEME_PATH/components/ 2>/dev/null

# T203b: Check for heading.twig override
echo "=== Checking for heading.twig override ==="
find $SUBTHEME_PATH -name "heading.twig" -o -name "heading*.twig" 2>/dev/null
# If found, must remove |raw from {{ content }}

# T203c: Check for button.twig override
echo "=== Checking for button.twig override ==="
find $SUBTHEME_PATH -name "button.twig" -o -name "button*.twig" 2>/dev/null
# If found, must remove |raw from {{ text }}

# T203d: Find templates with content/field rendering
echo "=== Templates with content/field rendering ==="
grep -rn "{{ content" $SUBTHEME_PATH/templates/
grep -rn "{{ field" $SUBTHEME_PATH/templates/
```

**Record the output**. Each `|raw` match needs review – `|raw` on user data
is an XSS vulnerability.

### 2.3 Audit sub-theme preprocess functions for API updates (T203e)

First, identify all entity reference fields in your project (useful for
understanding scope of potential information disclosure):

```bash
# Find all entity reference fields in config
grep -l "type: entity_reference" config/sync/field.storage*.yml | \
  xargs grep "^field_name:" | awk '{print $2}'
```

Then find CivicTheme API function calls that need `$variables` argument:

```bash
SUBTHEME_PATH=/path/to/your/subtheme

# Find CivicTheme API function calls that need $variables argument
echo "=== civictheme_get_field_referenced_entities calls ==="
grep -rn "civictheme_get_field_referenced_entities" $SUBTHEME_PATH/*.theme
grep -rn "civictheme_get_field_referenced_entities" $SUBTHEME_PATH/includes/

echo "=== civictheme_get_field_referenced_entity calls ==="
grep -rn "civictheme_get_field_referenced_entity" $SUBTHEME_PATH/*.theme
grep -rn "civictheme_get_field_referenced_entity" $SUBTHEME_PATH/includes/

echo "=== civictheme_get_field_value calls ==="
grep -rn "civictheme_get_field_value" $SUBTHEME_PATH/*.theme
grep -rn "civictheme_get_field_value" $SUBTHEME_PATH/includes/

echo "=== civictheme_get_referenced_entity_labels calls ==="
grep -rn "civictheme_get_referenced_entity_labels" $SUBTHEME_PATH/*.theme
grep -rn "civictheme_get_referenced_entity_labels" $SUBTHEME_PATH/includes/
```

**Record the output**. Each match needs updating to pass `$variables` argument.

**Important**: If you do NOT use these CivicTheme API functions and instead
access entity reference fields directly, you are responsible for ensuring
users have adequate access to those entities in your sub-theme.

### 2.4 Audit sub-theme preprocess for title/label XSS filtering (T203f)

```bash
SUBTHEME_PATH=/path/to/your/subtheme

# Find title/label output without filtering
echo "=== Title retrieval without filtering ==="
grep -rn "->getTitle(" $SUBTHEME_PATH/*.theme
grep -rn "->getTitle(" $SUBTHEME_PATH/includes/

echo "=== Entity label without filtering ==="
grep -rn "->label(" $SUBTHEME_PATH/*.theme
grep -rn "->label(" $SUBTHEME_PATH/includes/

echo "=== Link getText without filtering ==="
grep -rn "->getText(" $SUBTHEME_PATH/*.theme
grep -rn "->getText(" $SUBTHEME_PATH/includes/

echo "=== Direct title array access ==="
grep -rn "\['title'\]" $SUBTHEME_PATH/*.theme
grep -rn "\['title'\]" $SUBTHEME_PATH/includes/
```

**Record the output**. Each match needs `Xss::filter()` applied.

### 2.5 Audit menu preprocessing (T203g)

```bash
SUBTHEME_PATH=/path/to/your/subtheme

# Find menu-related templates
echo "=== Menu-related templates ==="
grep -rn "menu" $SUBTHEME_PATH/templates/
find $SUBTHEME_PATH/templates -name "*menu*" 2>/dev/null

# Find menu preprocess functions
echo "=== Menu preprocess functions ==="
grep -rn "preprocess_menu" $SUBTHEME_PATH/*.theme
grep -rn "preprocess_menu" $SUBTHEME_PATH/includes/
```

**Record the output**. Menu item titles must have `Xss::filter()` applied.

### 2.6 Audit attachment component overrides (T203h)

```bash
SUBTHEME_PATH=/path/to/your/subtheme

echo "=== Attachment/file-related templates ==="
grep -rn "attachment" $SUBTHEME_PATH/templates/
grep -rn "file" $SUBTHEME_PATH/templates/
find $SUBTHEME_PATH -name "*attachment*" -o -name "*file*" 2>/dev/null
```

### 2.8 Audit build tooling and CSS variables (T204)

```bash
SUBTHEME_PATH=/path/to/your/subtheme

# Check current SCSS variable usage
echo "=== SCSS variables (potential migration to CSS custom properties) ==="
grep -rn "\$ct-" $SUBTHEME_PATH/scss/ | head -20

# Check if already using CSS custom properties
echo "=== CSS custom properties usage ==="
grep -rn "var(--ct-" $SUBTHEME_PATH/scss/ | head -20
```

### 2.9 Document findings (T205)

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

**Related tasks**: T210–T221

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

### 3.2 Remove `|raw` filter from Twig templates (T211) – CRITICAL

This is the most critical security fix. The `|raw` filter on user-entered
content enables XSS attacks.

#### 3.2.1 Fix `heading.twig` override (if exists)

If your sub-theme overrides `heading.twig` (found in Section 2.2):

```twig
{# BEFORE (vulnerable) #}
<h{{ level }} class="ct-heading {{ modifier_class -}}" {% if attributes is not empty %}{{- attributes|raw -}}{% endif %}>
  {{- content|raw -}}
</h{{ level }}>

{# AFTER (secure) #}
<h{{ level }} class="ct-heading {{ modifier_class -}}" {% if attributes is not empty %}{{- attributes|raw -}}{% endif %}>
  {{- content -}}
</h{{ level }}>
```

**Note**: `attributes|raw` is acceptable as it's not user-entered content.

#### 3.2.2 Fix `button.twig` override (if exists)

If your sub-theme overrides `button.twig`:

```twig
{# BEFORE (vulnerable) #}
{% if icon_placement == 'before' %}
  {{- icon_markup -}}{{ text|raw }}
{% else %}
  {{ text|raw }}{{- icon_markup -}}
{% endif %}

{# AFTER (secure) #}
{% if icon_placement == 'before' %}
  {{- icon_markup -}}{{ text }}
{% else %}
  {{ text }}{{- icon_markup -}}
{% endif %}
```

#### 3.2.3 Fix other templates with `|raw` on user data

For each template identified in Section 2.2 using `|raw`:

1. Determine if the variable contains user-entered data.
2. If yes, remove `|raw` filter.
3. If HTML rendering was required, implement safe alternative:
   - Use Drupal render arrays.
   - Use allowed tags list with `|striptags('<p><a><strong>')`.
   - Sanitise in preprocess before passing to template.

### 3.3 Update CivicTheme API calls in preprocess (T212) – HIGH PRIORITY

For each API call identified in Section 2.3, add the `$variables` argument.

#### 3.3.1 Update `civictheme_get_field_referenced_entities()` calls

```php
// BEFORE (deprecated - triggers warning in 1.12.0, error in 1.13.0)
$items = civictheme_get_field_referenced_entities($entity, 'field_c_b_social_icons');

// AFTER (required)
$items = civictheme_get_field_referenced_entities($entity, 'field_c_b_social_icons', $variables);
```

#### 3.3.2 Update `civictheme_get_field_referenced_entity()` calls

```php
// BEFORE
$referenced_item = civictheme_get_field_referenced_entity($item, 'field_c_p_reference');

// AFTER
$referenced_item = civictheme_get_field_referenced_entity($item, 'field_c_p_reference', $variables);
```

#### 3.3.3 Update `civictheme_get_field_value()` calls

```php
// BEFORE
$featured_image = civictheme_get_field_value($block, 'field_c_b_featured_image', TRUE);

// AFTER (note: named parameter for build)
$featured_image = civictheme_get_field_value($block, 'field_c_b_featured_image', TRUE, build: $variables);
```

#### 3.3.4 Enable verbose error reporting to find missed calls

Add to `settings.local.php` during development:

```php
error_reporting(E_ALL);
ini_set('display_errors', TRUE);
ini_set('display_startup_errors', TRUE);
```

The deprecation warnings will identify any missed API calls:

```
'Calling civictheme_get_field_referenced_entities without the $build
argument is deprecated in civictheme:1.12.0. It will be required in
civictheme:1.13.0. Triggered when getting entity for <field_name>.'
```

### 3.4 Add XSS filtering to title/label outputs (T213) – HIGH PRIORITY

For each preprocess function identified in Section 2.4:

#### 3.4.1 Add use statement for Xss class

At the top of your `.theme` file or includes:

```php
use Drupal\Component\Utility\Xss;
```

#### 3.4.2 Filter node titles

```php
// Example from _civictheme_preprocess_block__civictheme_banner
$title = \Drupal::service('title_resolver')->getTitle(
  \Drupal::request(),
  \Drupal::routeMatch()->getRouteObject()
);
$title = (string) (is_array($title) ? reset($title) : ((string) $title));
$title = Xss::filter($title);
$title = strip_tags($title);
$variables['title'] = $title;
```

#### 3.4.3 Filter link text from `getText()`

```php
// Example for breadcrumb links
foreach ($breadcrumb->getLinks() as $link) {
  $link_text = $link->getText();
  $variables['breadcrumb']['links'][] = [
    'text' => is_array($link_text) ? $link_text : Xss::filter((string) $link_text),
    'url' => $link->getUrl()->toString(),
  ];
}
```

#### 3.4.4 Filter entity labels

```php
$label = Xss::filter($entity->label());
```

### 3.5 Add XSS filtering to menu item titles (T214)

For menu preprocessing identified in Section 2.5:

```php
use Drupal\Component\Utility\Xss;

// In your menu preprocessing function
$item['title'] = isset($item['title']) ? Xss::filter($item['title']) : '';
```

### 3.6 Update manual list preprocess for access checking (T215)

If your sub-theme overrides manual list preprocessing, update to check
access to referenced entities:

```php
use Drupal\Core\Entity\EntityInterface;

/**
 * Preprocess manual list paragraph.
 */
function yourtheme_preprocess_paragraph__civictheme_manual_list(&$variables) {
  /** @var \Drupal\paragraphs\Entity\Paragraph $paragraph */
  $paragraph = $variables['paragraph'];
  
  /** @var \Drupal\Core\Entity\ContentEntityInterface[] $items */
  $items = civictheme_get_field_referenced_entities($paragraph, 'field_c_p_list_items', $variables);
  $builder = \Drupal::entityTypeManager()->getViewBuilder('paragraph');
  
  if ($items) {
    foreach ($items as $item) {
      // Non-reference cards can be added directly
      if (!$item->hasField('field_c_p_reference')) {
        $variables['rows'][] = $builder->view($item);
        continue;
      }
      
      // Reference cards need access check
      $referenced_item = civictheme_get_field_referenced_entity($item, 'field_c_p_reference', $variables);
      if ($referenced_item instanceof EntityInterface) {
        $variables['rows'][] = $builder->view($item);
      }
      // If no access to referenced entity, skip the card (prevents empty columns)
    }
  }
}
```

### 3.7 Remove iframe paragraph attributes field (T216) – if applicable

If your project uses iframe paragraphs with the Attributes field:

#### 3.7.1 Check for existing values

```bash
# Via drush (use ahoy/docker as needed)
drush sql:query "SELECT * FROM paragraph__field_c_p_attributes LIMIT 10;"
```

Or create a View in the admin UI to check.

#### 3.7.2 Export and delete field configuration

```bash
# Export current config
drush cex -y

# Remove field configuration files
rm config/sync/field.field.paragraph.civictheme_iframe.field_c_p_attributes.yml
rm config/sync/field.storage.paragraph.field_c_p_attributes.yml

# Import to delete from database
drush cim -y
```

#### 3.7.3 Add required attributes in preprocess instead

```php
/**
 * Preprocess iframe paragraph.
 */
function yourtheme_preprocess_paragraph__civictheme_iframe(&$variables) {
  // Add any required iframe attributes here instead of in the field
  $variables['attributes']['loading'] = 'lazy';
  $variables['attributes']['sandbox'] = 'allow-scripts allow-same-origin';
}
```

### 3.8 Update user permissions for Icons media type (T217)

**Important**: The `civictheme_embed_svg()` function does NOT protect
against XSS – it relies on an appropriate level of trust in users managing
SVG Icons. Therefore, Content Authors and Approvers should not have
permission to create or edit SVG icons.

Remove permissions that allow Content Authors/Approvers to edit SVG icons:

```bash
# Check current permissions
drush role:perm:list content_author | grep -i icon
drush role:perm:list approver | grep -i icon

# Remove permissions via drush
drush role:perm:remove content_author 'create media:icon'
drush role:perm:remove content_author 'edit any media:icon'
drush role:perm:remove approver 'create media:icon'
drush role:perm:remove approver 'edit any media:icon'

# Export updated permissions
drush cex -y
```

Or update via admin UI: `/admin/people/permissions`

Only trusted administrators should be able to upload and manage SVG icons.

### 3.9 Update menu customisations (T218) – if applicable

If menu overrides were identified in Section 2.5:

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

### 3.10 Update attachment overrides (T219) – if applicable

If attachment overrides were identified in Section 2.6:

```bash
# Compare attachment templates
diff -u $CIVICTHEME/templates/*/attachment*.twig \
       $SUBTHEME_PATH/templates/*/attachment*.twig 2>/dev/null
```

**Key change in 1.12.0**: Fixed attachment component not resetting file
extension from previous items.

### 3.11 Consider CSS variable migration (T220) – optional

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

**Note**: This is optional and can be deferred if time is constrained.
Security fixes are the priority.

### 3.12 Rebuild sub-theme assets (T221)

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

### 3.13 Commit changes

```bash
git add -A
git status  # Review all changes

# Commit in logical chunks
git commit -m "security: Updated sub-theme for CivicTheme 1.12 security fixes"
```

---

## 4. Validate

**Related tasks**: T230–T242

### 4.1 Clear caches and run updates (T230)

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

### 4.2 Verify security fixes are applied (T231)

```bash
# Confirm CivicTheme version
composer show drupal/civictheme | grep versions
# Expected: 1.12.x

# Check for deprecation warnings (indicates missed API updates)
drush watchdog:show --filter="deprecated" --count=50
```

### 4.3 XSS testing (T232) – CRITICAL

This is the most important validation step. Test that XSS vulnerabilities
have been patched.

#### 4.3.1 XSS test payload

Use this payload for all tests:

```html
<script>alert('☠️');</script>
```

#### 4.3.2 Test locations

Enter the XSS payload in each of these locations:

| Location | How to test |
|----------|-------------|
| Component text fields | Edit any CivicTheme paragraph, enter payload in text/content field |
| Node titles | Create/edit a node, enter payload as title |
| Taxonomy term names | Create/edit a taxonomy term, enter payload as name |
| Link text fields | In any link field with title enabled, enter payload as title |
| Menu item titles | Create/edit a menu link, enter payload as title |

#### 4.3.3 Expected results

- **PASS**: The page renders without showing an alert dialog. The script
  tag is either:
  - Escaped and displayed as visible text: `<script>alert('☠️');</script>`
  - Stripped entirely from output
- **FAIL**: An alert dialog appears with the skull emoji. This indicates
  XSS vulnerability exists.

#### 4.3.4 If tests fail

1. Identify which component/field is vulnerable.
2. Check if sub-theme has an override for that component.
3. If yes, apply the security fixes from Section 3.
4. If no, report to CivicTheme maintainers.

### 4.4 Verify information disclosure fix (T233)

Test that entity reference fields respect access control:

1. **Create test content**:
   - Create an unpublished node (e.g. "Secret Article").
   - Create a published node that references the unpublished node.

2. **Test as anonymous user**:
   - Log out or use incognito/private browsing.
   - Visit the published node.

3. **Expected result**:
   - The unpublished "Secret Article" should NOT appear.
   - No title, teaser, or link to unpublished content should be visible.

4. **If test fails**:
   - Check sub-theme preprocess functions for entity reference fields.
   - Ensure using `civictheme_get_field_referenced_entities()` with
     `$variables` argument.

### 4.5 Check logs for errors and deprecations (T234)

```bash
# View recent watchdog entries
drush watchdog:show --count=50

# Filter for CivicTheme-related errors
drush watchdog:show --filter="civictheme" --count=50
drush watchdog:show --filter="component" --count=50

# Check for deprecation warnings about missing $build argument
drush watchdog:show --filter="deprecated" --count=50
drush watchdog:show --filter="civictheme_get_field" --count=50
```

**Expected**: No errors related to CivicTheme or components.

**If deprecation warnings found**:

```
'Calling civictheme_get_field_referenced_entities without the $build
argument is deprecated in civictheme:1.12.0...'
```

Return to Section 3.3 and update the specified field's API call.

### 4.6 Validate page rendering (T235)

Open the site in a browser and manually check:

- [ ] **Home page** (T235a):
  - Header renders correctly.
  - Footer renders correctly.
  - Navigation works.
  - Content displays properly.

- [ ] **Navigation** (T235b):
  - Primary navigation works.
  - Secondary navigation works.
  - Mobile menu opens and closes.
  - Mobile menu items visible (bug fix in 1.12.0).
  - Menu links navigate correctly.

- [ ] **Search** (T235c):
  - Search form displays.
  - Search results return.
  - **Inline filter**: Keyword displays in search results message.

- [ ] **Content pages** (T235d):
  - Article pages render.
  - Landing pages render.
  - Page content types render.

- [ ] **Attachments** (T235e):
  - Pages with file attachments display correctly.
  - File extensions display correctly (bug fix in 1.12.0).
  - Multiple attachments show correct extensions for each file.

- [ ] **Forms** (T235f):
  - Contact forms display.
  - Webforms display (bug fix for archived webforms in 1.12.0).
  - Webform messages render correctly (striptags fix in 1.12.0).
  - Form submissions work.

### 4.7 Validate specific components (T236)

- [ ] **Menus** (T236a):
  - Desktop menu renders correctly.
  - Mobile menu visibility is correct.
  - Menu level classes apply correctly.

- [ ] **Cards** (T236b):
  - Card components render with accessibility updates.
  - Card interactions work as expected.

- [ ] **Videos** (T236c):
  - If using video with transcripts, verify transcript functionality.

- [ ] **Links** (T236d):
  - Email addresses in content are NOT incorrectly converted to links.
  - Actual links work correctly.

- [ ] **Logo** (T236e):
  - If logo path contains spaces, verify it displays correctly.

### 4.8 Test new features (T237) – optional, if adopting

- [ ] **Multi-line header** (T237a):
  - Enable via theme settings.
  - Verify logo displays in container above navigation.
  - Verify navigation uses full container width.
  - Test with more than 4 navigation items.

- [ ] **Message organism** (T237b):
  - Create content using Message component.
  - Test all four styling options.
  - Verify accessibility.

- [ ] **Fast fact card** (T237c):
  - Create content using Fast fact card.
  - Verify display and styling.

### 4.9 Test sub-theme build (T238)

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

### 4.10 Run tests AFTER upgrade (T239)

Re-run the same tests discovered in Section 1.9 to verify the upgrade has
not introduced regressions.

```bash
# Run all available tests
ahoy test-unit
ahoy test-bdd
composer test
```

**Compare results with pre-upgrade baseline (Section 1.10)**:

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

### 4.11 Document validation results (T240)

Create a checklist of validation results:

```markdown
## Validation Results – CivicTheme 1.11.0 → 1.12.0

Date: YYYY-MM-DD
Tester: [Name]

### Security Review (CRITICAL)

#### XSS Testing
| Location | Payload entered | Alert shown? | Result |
|----------|-----------------|--------------|--------|
| Component text field | `<script>alert('☠️');</script>` | Yes/No | PASS/FAIL |
| Node title | `<script>alert('☠️');</script>` | Yes/No | PASS/FAIL |
| Taxonomy term name | `<script>alert('☠️');</script>` | Yes/No | PASS/FAIL |
| Link text field | `<script>alert('☠️');</script>` | Yes/No | PASS/FAIL |
| Menu item title | `<script>alert('☠️');</script>` | Yes/No | PASS/FAIL |

#### Information Disclosure Testing
- [ ] Unpublished referenced content NOT visible to anonymous: PASS/FAIL

#### Security Code Review
- [ ] heading.twig `|raw` removed from content: PASS/N/A
- [ ] button.twig `|raw` removed from text: PASS/N/A
- [ ] Other `|raw` uses reviewed: PASS/N/A
- [ ] CivicTheme API calls updated with $variables: PASS/N/A
- [ ] Title/label outputs use Xss::filter(): PASS/N/A
- [ ] Menu titles use Xss::filter(): PASS/N/A
- [ ] No deprecation warnings in logs: PASS/FAIL

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
- [ ] Menus (mobile visibility fix): PASS/FAIL
- [ ] Cards (accessibility updates): PASS/FAIL
- [ ] Videos/Transcripts: PASS/FAIL
- [ ] Links (email fix): PASS/FAIL
- [ ] Logo (spaces in path): PASS/FAIL

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

**Related tasks**: T240, T241, T242

### 5.1 Update customisation register (T240)

Edit `docs/civic-theme-upgrades/customisations.md` and update each entry
affected by this upgrade:

```markdown
| ID | Description | Status | Notes |
|----|-------------|--------|-------|
| C001 | heading.twig override | Updated | Removed |raw from content |
| C002 | button.twig override | Updated | Removed |raw from text |
| C003 | Menu preprocess | Updated | Added Xss::filter() to titles |
| C004 | Banner preprocess | Updated | API calls updated with $variables |
| C005 | Attachment template | Reviewed | No changes needed |
```

### 5.2 Document lessons learned (T241)

Add a "Lessons Learned" section:

```markdown
## Lessons Learned – CivicTheme 1.11.0 → 1.12.0

### What went well
- [List things that worked smoothly]

### Challenges encountered
- [List difficulties and how they were resolved]

### Security review findings
- Templates with |raw filter: [count] found, [count] fixed
- CivicTheme API calls updated: [count]
- Preprocess functions with XSS filtering added: [count]
- XSS tests: all passed / [count] failed

### Follow-up tasks
- [ ] CSS variable migration (if deferred)
- [ ] Adopt new features (Message organism, Fast fact card, etc.)
- [ ] Address any remaining deprecation warnings before 1.13.0

### Time spent
- Discovery: X hours
- Security review & fixes: X hours
- Validation (including XSS testing): X hours
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

### 5.4 Prepare for review (T242)

Create a merge request with the following information:

```markdown
## CivicTheme 1.11.0 → 1.12.0 Upgrade (Security Release)

### Summary
This PR upgrades CivicTheme from 1.11.0 to 1.12.0, addressing security
vulnerabilities SA-CONTRIB-2025-112 and SA-CONTRIB-2025-113.

### Security Changes Made
- [ ] heading.twig: Removed |raw from content output
- [ ] button.twig: Removed |raw from text output
- [ ] Other templates: Reviewed X files for |raw usage
- [ ] API calls: Updated X calls to pass $variables argument
- [ ] Preprocess: Added Xss::filter() to X title/label outputs
- [ ] Menu preprocessing: Added Xss::filter() to menu titles
- [ ] Permissions: Removed Icons media edit from Content Author/Approver

### XSS Testing Results
| Location | Result |
|----------|--------|
| Component text fields | PASS |
| Node titles | PASS |
| Taxonomy terms | PASS |
| Link text fields | PASS |
| Menu item titles | PASS |

### Information Disclosure Testing
- [ ] Unpublished referenced content not visible to anonymous: PASS

### Customisations Impacted
- C001: [Description] - [Action taken]
- C002: [Description] - [Action taken]

### Other Testing Performed
- [ ] All pages render correctly
- [ ] Navigation works (including mobile menu fix)
- [ ] Search returns correct results with inline filter
- [ ] Attachments display correct file extensions
- [ ] Forms/webforms function properly
- [ ] No deprecation warnings in logs

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

### Security: Twig template changes

| Component | Before (vulnerable) | After (secure) |
|-----------|---------------------|----------------|
| heading.twig | `{{- content\|raw -}}` | `{{- content -}}` |
| button.twig | `{{ text\|raw }}` | `{{ text }}` |

### Security: CivicTheme API updates

| Function | Before (deprecated) | After (required) |
|----------|---------------------|------------------|
| `civictheme_get_field_referenced_entities` | `($entity, 'field')` | `($entity, 'field', $variables)` |
| `civictheme_get_field_referenced_entity` | `($item, 'field')` | `($item, 'field', $variables)` |
| `civictheme_get_field_value` | `($block, 'field', TRUE)` | `($block, 'field', TRUE, build: $variables)` |

### Security: XSS filtering patterns

```php
use Drupal\Component\Utility\Xss;

// Filter node/page titles
$title = Xss::filter($title);
$title = strip_tags($title);

// Filter link text
$text = is_array($link_text) ? $link_text : Xss::filter((string) $link_text);

// Filter menu item titles
$item['title'] = isset($item['title']) ? Xss::filter($item['title']) : '';

// Filter entity labels
$label = Xss::filter($entity->label());
```

### Security: XSS test payload

```html
<script>alert('☠️');</script>
```

Enter in text fields, titles, link text, and menu items. **NO alert dialog
should appear** if XSS protections are working correctly.

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

