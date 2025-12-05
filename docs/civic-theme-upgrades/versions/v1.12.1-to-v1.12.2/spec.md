# CivicTheme upgrade spec: v1.12.1 → v1.12.2

**Project**: Destination Drupal project using CivicTheme  
**CivicTheme source version**: `1.12.1`  
**CivicTheme target version**: `1.12.2`  
**Per-version directory**: `docs/civic-theme-upgrades/versions/v1.12.1-to-v1.12.2/`

This document defines the canonical per-version specification for the
CivicTheme `1.12.1 → 1.12.2` upgrade, using the framework defined in
`docs/civic-theme-upgrades/planning.md`. Destination projects SHOULD use
this spec (and its companion `tasks.md` and `playbook.md`) as the primary
instructions for performing this upgrade, adapting only where
project-specific customisations require it.

---

## 1. Environment assumptions and scope

- All upgrade work happens in **non-production** or otherwise safely
  isolated environments (e.g. feature branches, staging sites, local
  environments).
- The project has:
  - A working Drupal code base using CivicTheme `1.12.1`.
  - Composer-based dependency management.
  - Git version control with access to history for custom themes.
- Drupal core requirements remain unchanged from 1.12.1:
  - `core_version_requirement: ^10.2 || ^11` (same as 1.12.1).
  - If the project successfully upgraded to 1.12.1, no additional Drupal
    core changes are required for 1.12.2.
- Standard organisational safeguards apply:
  - Backups and/or database snapshots exist prior to running the upgrade.
  - Deployments are reviewed via merge requests or equivalent.
- **Existing `.gitignore` files MUST NOT be modified.** The assumption is that
  the theme is already operational and its ignored files and folders (such as
  `node_modules/`, `dist/`, vendor directories, and build artefacts) are
  correctly configured. Modifying `.gitignore` can disrupt the existing build
  pipeline, accidentally commit generated files, or break local development
  environments.
- This spec focuses on the **CivicTheme 1.12.1 → 1.12.2** step only. Any
  earlier or later version jumps MUST be documented in separate directories.

Out of scope:

- Production deployment steps.
- The detailed mechanics of upgrading Drupal core itself.

---

## 2. Upstream references for 1.12.2

Authors and AI assistants MUST consult the following upstream sources when
preparing and validating this upgrade:

- **Release notes**: CivicTheme `1.12.2` on drupal.org  
  `https://www.drupal.org/project/civictheme/releases/1.12.2`
- **CivicTheme documentation portal**:  
  `https://docs.civictheme.io/changelog`
- **Upstream code diff** (1.12.1 → 1.12.2):  
  `https://git.drupalcode.org/project/civictheme/-/compare/1.12.1...1.12.2?from_project_id=86817`

When maintaining this spec in a real project, copy or summarise the
relevant sections from these sources to keep this document authoritative
for the `1.12.1 → 1.12.2` upgrade.

---

## 3. High-level upstream changes (1.12.1 → 1.12.2)

CivicTheme 1.12.2 is a **bug-fix release** that addresses important issues
with Twig template rendering and HTML entity handling. This release contains
**breaking changes** for sub-themes that require careful attention.

### 3.1 BREAKING CHANGE – Removal of `|raw` Twig filter

**This is the most significant change in this release and requires sub-theme
updates.**

#### 3.1.1 What changed

The `|raw` Twig filter has been **removed from all components** in the
CivicTheme UI Kit. This filter was previously used to render HTML content
without escaping.

**Issue**: [#3554125: Removing raw from all components](https://www.drupal.org/project/civictheme/issues/3554125)

#### 3.1.2 Why this change was made

The `|raw` filter bypasses Twig's auto-escaping security mechanism, which
protects against XSS (Cross-Site Scripting) vulnerabilities. Removing it
improves the security posture of CivicTheme components.

#### 3.1.3 Impact on sub-themes

**If your sub-theme overrides CivicTheme components or uses custom
preprocessing, you MUST update your code to:**

1. **Update Twig templates**: Remove `|raw` filters and ensure content is
   passed as renderable arrays or Markup objects.

2. **Update preprocessing**: Change preprocessing functions to return
   `\Drupal\Component\Render\MarkupInterface` objects (e.g. using
   `Markup::create()`) instead of raw HTML strings for content that should
   not be escaped.

3. **Review all custom components**: Check for any `|raw` usage in your
   sub-theme's Twig templates.

**Example change in preprocessing**:

```php
// Before (1.12.1)
$variables['title'] = '<span class="custom">' . $title . '</span>';

// After (1.12.2)
use Drupal\Component\Render\MarkupInterface;
use Drupal\Core\Render\Markup;

$variables['title'] = Markup::create('<span class="custom">' . Html::escape($title) . '</span>');
```

**Example change in Twig**:

```twig
{# Before (1.12.1) #}
{{ content|raw }}

{# After (1.12.2) #}
{{ content }}
```

#### 3.1.4 Files changed

The following files in CivicTheme core were updated to support the raw filter
removal:

| File | Change |
|------|--------|
| `civictheme.post_update.php` | Added hook for sub-theme updates |
| `civictheme_create_subtheme.php` | Added feature to remove Drupal packaging info from starter kit |
| Multiple component `.twig` files | Removed `\|raw` filter usage |
| Multiple preprocessing files | Updated to return Markup objects |

### 3.2 Bug fixes – HTML entity rendering

Several bug fixes address issues where HTML entities (such as `&` rendering
as `&amp;`) were being double-escaped in component output.

#### 3.2.1 Promo card title ampersand fix

**Issue**: [#3558421: When I enter a title on promo card with &, it is outputted with &amp;](https://www.drupal.org/project/civictheme/issues/3558421)

**Problem**: Titles containing ampersands (`&`) were being rendered as
`&amp;` in the HTML output, displaying incorrectly to users.

**Solution**: Fixed rendering of HTML entities in string fields. This affects
multiple components including:

- Promo cards
- Buttons
- Menu items
- Breadcrumbs

#### 3.2.2 Components affected

| Component | Fix applied |
|-----------|-------------|
| Promo card | Title field HTML entity fix |
| Button | Title ampersand rendering fix |
| Menu items | Removed HTML entity double-escaping |
| Breadcrumbs | Added Markup objects for proper rendering |
| Title field | Added Markup objects for proper rendering |

### 3.3 Starter kit improvements

#### 3.3.1 Drupal packaging information removal

**Issue**: [#3554770: Drupal Admin UI not recognizing 1.12 update](https://www.drupal.org/project/civictheme/issues/3554770)

A new feature was added to the starter kit creation script that automatically
removes Drupal packaging information from the generated sub-theme's `.info.yml`
file. This prevents conflicts with Drupal's update status system.

**Impact**: Only relevant when creating new sub-themes. No action required for
existing sub-themes.

### 3.4 NPM package update

The `@civictheme/sdc` package has been updated from 1.12.1 to 1.12.2.

**Impact**: If you rebuild your sub-theme's front-end assets, you will
receive the updated package. This version includes the Twig template changes
that remove `|raw` filters.

### 3.5 Summary of changes

| Area | Change | Risk | Sub-theme action |
|------|--------|------|------------------|
| Twig `\|raw` removal | Removed from all components | **MEDIUM** | **REQUIRED**: Update overridden templates |
| HTML entity rendering | Fixed double-escaping | LOW | Review if you have custom preprocessing |
| Button title | Fixed ampersand rendering | LOW | None (upstream fix) |
| Menu items | Fixed entity escaping | LOW | None (upstream fix) |
| Breadcrumbs | Added Markup objects | LOW | Review if overridden |
| Starter kit | Added packaging info removal | N/A | None (new sub-themes only) |
| NPM package | Updated to 1.12.2 | LOW | Rebuild assets |

---

## 4. Customisation inventory view (from register)

This spec assumes the project maintains a canonical customisation register
at:

- `docs/civic-theme-upgrades/customisations.md`

For this release, focus on entries that intersect with:

- **Overridden Twig templates**: Any template that uses `|raw` filter.
- **Custom preprocessing**: Functions that pass HTML strings to templates.
- **Component overrides**: Any overridden CivicTheme components.
- **Promo card customisations**: If using custom title handling.
- **Button customisations**: If using custom title preprocessing.
- **Menu customisations**: If using custom menu item processing.
- **Breadcrumb customisations**: If overriding breadcrumb rendering.

Each relevant customisation SHOULD be cross-referenced by ID (e.g. `C5`,
`C7`) in the tasks and playbook.

---

## 5. Risk assessment and focus areas

Based on upstream changes documented in Section 3, this is a **medium-risk**
upgrade due to the breaking changes around `|raw` filter removal.

### 5.1 MEDIUM RISK – Sub-theme template updates required

- **Twig `|raw` filter removal**:
  - This is a **breaking change** that requires sub-theme updates.
  - **Action REQUIRED**: Audit all sub-theme Twig templates for `|raw` usage.
  - **Action REQUIRED**: Update preprocessing to use Markup objects.

- **HTML entity rendering fixes**:
  - These are bug fixes that should improve display.
  - **Action**: Verify that content with special characters (e.g. `&`, `<`,
    `>`) renders correctly after upgrade.

### 5.2 LOW RISK – Standard upgrade steps

- **NPM package update**:
  - Standard asset rebuild required.
  - **Action**: Run front-end build after upgrade.

- **Starter kit changes**:
  - Only affects new sub-theme creation.
  - **Action**: None required for existing projects.

### 5.3 Project-specific risk areas

Projects SHOULD document their own risk areas here by referencing IDs from
the customisation register:

- `C001` – [Placeholder: list impacted customisation and why]

Review the customisation register at
`docs/civic-theme-upgrades/customisations.md` and annotate each entry
with its risk level for this upgrade.

---

## 6. Upgrade strategy overview

This upgrade requires careful attention to sub-theme templates due to the
removal of the `|raw` Twig filter.

### 6.1 Discovery phase

1. **Verify prerequisites**:
   - Confirm current CivicTheme version is `1.12.1`.
   - Confirm Drupal core version meets `^10.2 || ^11` requirement.
   - Ensure Git working tree is clean with a dedicated feature branch.

2. **Audit sub-theme for `|raw` usage**:
   - Search all Twig templates in your sub-theme for `|raw` filter usage.
   - Document each occurrence for update in the change phase.

3. **Audit preprocessing for raw HTML strings**:
   - Review preprocessing functions that pass HTML to templates.
   - Document functions that need Markup object conversion.

### 6.2 Change phase

1. **Update Drupal dependencies**:
   - Run `composer require drupal/civictheme:1.12.2` to install exactly
     version 1.12.2. **Do NOT use caret (^) or tilde (~) constraints**;
     sequential upgrades require pinning to the precise target version.
   - Verify `composer.lock` is regenerated.

2. **Update sub-theme Twig templates**:
   - Remove `|raw` filters from overridden templates.
   - Ensure content is passed as renderable arrays or Markup objects.

3. **Update preprocessing functions**:
   - Convert raw HTML string returns to Markup objects.
   - Use `\Drupal\Core\Render\Markup::create()` for trusted HTML.
   - Use `\Drupal\Component\Utility\Html::escape()` for user input.

4. **Rebuild sub-theme assets**:
   - Run front-end build to pick up the updated `@civictheme/sdc` package.

### 6.3 Validation phase

1. **Clear caches and run updates**:
   - `drush cr` (or equivalent).
   - `drush updb` – run any database updates.
   - `drush cim` if configuration changes are pending.

2. **Test content rendering**:
   - Verify pages with special characters (`&`, `<`, `>`) render correctly.
   - Check promo cards, buttons, menu items, and breadcrumbs.

3. **Test overridden components**:
   - Verify all overridden templates render content correctly.
   - Check that HTML content is not double-escaped or unescaped.

4. **Update documentation**:
   - Mark completed tasks in `tasks.md`.
   - Record any lessons learned in `playbook.md`.

The detailed tasks and execution steps are defined in `tasks.md` and
`playbook.md` for this directory.

---

## 7. Guidance for AI coding assistants

When assisting with this upgrade, AI models SHOULD:

- Start with:
  - `docs/civic-theme-upgrades/customisations.md`
  - This file: `docs/civic-theme-upgrades/versions/v1.12.1-to-v1.12.2/spec.md`
  - The corresponding `tasks.md` and `playbook.md`.
- Understand that this is a **bug-fix release with breaking changes**
  requiring sub-theme updates.
- Use the upstream links in Section 2 to understand the context.
- Treat the customisation register as the source of truth for what must be
  preserved or adapted.
- Prefer:
  - Systematically auditing sub-theme templates for `|raw` usage.
  - Using `Markup::create()` for trusted HTML content.
  - Avoiding destructive operations and any commands that act directly on
    production environments.
- Keep this spec and the customisation register up to date by:
  - Adding notes about which customisations were impacted or retired.
  - Recording any new customisations introduced as part of the upgrade.

---

## 8. References

This section provides links to official upstream resources for the CivicTheme `1.12.1 → 1.12.2` upgrade.

### Release information

- **CivicTheme 1.12.1 release notes** (source version):  
  https://www.drupal.org/project/civictheme/releases/1.12.1
- **CivicTheme 1.12.2 release notes** (target version):  
  https://www.drupal.org/project/civictheme/releases/1.12.2
- **CivicTheme documentation portal**:  
  https://docs.civictheme.io/changelog

### Code comparison

- **Git diff** (1.12.1 → 1.12.2):  
  https://git.drupalcode.org/project/civictheme/-/compare/1.12.1...1.12.2?from_project_id=86817

### Related issues

- **#3554125**: Removing raw from all components  
  https://www.drupal.org/project/civictheme/issues/3554125
- **#3558421**: Promo card ampersand rendering fix  
  https://www.drupal.org/project/civictheme/issues/3558421
- **#3554770**: Drupal Admin UI update recognition  
  https://www.drupal.org/project/civictheme/issues/3554770

### Upgrade tools and resources

- **CivicTheme Upgrade Tools** (optional helper scripts):  
  https://github.com/civictheme/upgrade-tools/blob/main/README.md
- **CivicTheme Drupal.org project page**:  
  https://www.drupal.org/project/civictheme
- **CivicTheme main website**:  
  https://civictheme.io

### Security advisories

- **Security advisories** (if applicable):  
  Check https://www.drupal.org/security/contrib for any security advisories related to CivicTheme 1.12.1 or 1.12.2

---

## 9. Developer review checklist

Before considering the 1.12.1 → 1.12.2 upgrade complete, a developer
SHOULD confirm:

### Upgrade review

- [ ] CivicTheme version is 1.12.2 (`composer show drupal/civictheme`).
- [ ] Database updates have been run (`drush updb`).
- [ ] Sub-theme assets have been rebuilt.

### Template review (CRITICAL for this release)

- [ ] All `|raw` filter usages in sub-theme templates have been removed.
- [ ] Preprocessing functions return Markup objects for HTML content.
- [ ] Overridden components render content without double-escaping.

### Functional review

- [ ] The project builds successfully and basic smoke tests (cache clear,
      config import, entity updates) complete without errors.
- [ ] Promo card titles with special characters render correctly.
- [ ] Button titles with special characters render correctly.
- [ ] Menu items with special characters render correctly.
- [ ] Breadcrumbs render correctly.
- [ ] All overridden templates display content as expected.

### Customisation register

- [ ] The customisation register has been reviewed for any entries that
      may have been affected by this upgrade.
- [ ] Any templates using `|raw` have been updated and documented.

### Documentation

- [ ] Any regressions or follow-up tasks are captured in `tasks.md` or
      in a project issue tracker.
- [ ] This spec, `tasks.md` and `playbook.md` accurately reflect what was
      performed, so that future upgrades can rely on them as history.

