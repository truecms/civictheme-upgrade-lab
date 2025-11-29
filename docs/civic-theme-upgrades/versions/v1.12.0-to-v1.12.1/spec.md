# CivicTheme upgrade spec: v1.12.0 → v1.12.1

**Project**: Destination Drupal project using CivicTheme  
**CivicTheme source version**: `1.12.0`  
**CivicTheme target version**: `1.12.1`  
**Per-version directory**: `docs/civic-theme-upgrades/versions/v1.12.0-to-v1.12.1/`

This document defines the canonical per-version specification for the
CivicTheme `1.12.0 → 1.12.1` upgrade, using the framework defined in
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
  - A working Drupal code base using CivicTheme `1.12.0`.
  - Composer-based dependency management.
  - Git version control with access to history for custom themes.
- Drupal core requirements remain unchanged from 1.12.0:
  - `core_version_requirement: ^10.2 || ^11` (same as 1.12.0).
  - If the project successfully upgraded to 1.12.0, no additional Drupal
    core changes are required for 1.12.1.
- Standard organisational safeguards apply:
  - Backups and/or database snapshots exist prior to running the upgrade.
  - Deployments are reviewed via merge requests or equivalent.
- This spec focuses on the **CivicTheme 1.12.0 → 1.12.1** step only. Any
  earlier or later version jumps MUST be documented in separate directories.

Out of scope:

- Production deployment steps.
- The detailed mechanics of upgrading Drupal core itself.

---

## 2. Upstream references for 1.12.1

Authors and AI assistants MUST consult the following upstream sources when
preparing and validating this upgrade:

- **Release notes**: CivicTheme `1.12.1` on drupal.org  
  `https://www.drupal.org/project/civictheme/releases/1.12.1`
- **CivicTheme documentation portal**:  
  `https://docs.civictheme.io/changelog`
- **Upstream code diff** (1.12.0 → 1.12.1):  
  `https://git.drupalcode.org/project/civictheme/-/compare/1.12.0...1.12.1?from_project_id=86817`

When maintaining this spec in a real project, copy or summarise the
relevant sections from these sources to keep this document authoritative
for the `1.12.0 → 1.12.1` upgrade.

---

## 3. High-level upstream changes (1.12.0 → 1.12.1)

CivicTheme 1.12.1 is a **bug-fix release** that addresses issues discovered
after the 1.12.0 release. This is a low-risk upgrade with no new features
or breaking changes.

### 3.1 Bug fixes – WYSIWYG preprocessing

The primary fix in this release addresses a regression introduced in 1.12.0
where the email-to-URL conversion fix was causing unintended filtering of
content in WYSIWYG fields.

#### 3.1.1 Problem addressed

The 1.12.0 release included a fix to prevent email addresses from being
incorrectly converted to anchor tags. However, this fix caused any
disallowed tag in `civictheme_rich_text` to be filtered out during
preprocessing.

This was problematic because the `_civictheme_process__html_content`
function expands and renders HTML in order to:

- Preprocess and add CivicTheme link classes
- Convert URLs to links (and add CivicTheme link styling)

The unintended consequence was that **embedded media was being removed**,
including:

- `<iframe>` elements
- `<figure>` elements
- Other media embeds

#### 3.1.2 Solution implemented

The fix removes the email-to-anchor-tag conversion from CivicTheme's
preprocessing entirely. Instead, URL-to-link conversion (including emails)
is now delegated to Drupal's text format filters.

**Key change**: If you want email addresses to be converted to `mailto:`
links, you need to configure your text format to enable the "Convert URLs
into links" filter. The `civictheme_rich_text` format does this by default.

**Files changed**:

| File | Change |
|------|--------|
| `includes/link.inc` | Removed `_civictheme_process_html_content_links_emails()` function; updated `_civictheme_process_html_content_urls_to_links()` to use text format filters |
| `includes/process.inc` | Added `$format` parameter to `_civictheme_process__html_content()` |
| `includes/wysiwyg.inc` | Pass text format to `_civictheme_process__html_content()` |
| `includes/node.inc` | Updated alert node preprocessing to use text format |
| `includes/utilities.inc` | Removed `components.link.email` opt-out flag |

**Sub-theme impact**: If your sub-theme calls `_civictheme_process__html_content()`
directly, you MAY need to update calls to pass the text format as the third
argument. However, this is optional – the function remains backwards compatible.

### 3.2 Post-update hook for iframe attributes field

A new post-update hook has been added to automatically remove the
`field_c_p_attributes` field from the `civictheme_iframe` paragraph type.

**Background**: In 1.12.0, this field was identified as a security risk (XSS
injection via attributes) and the recommendation was to remove it manually.
The 1.12.1 release automates this removal.

**File changed**:

| File | Change |
|------|--------|
| `civictheme.post_update.php` | Added `civictheme_post_update_remove_civictheme_iframe_field_c_p_attributes()` |

**Impact**: When you run `drush updb` after upgrading to 1.12.1, this
post-update hook will automatically:

1. Remove `field.field.paragraph.civictheme_iframe.field_c_p_attributes`
2. Remove `field.storage.paragraph.field_c_p_attributes`

If you already manually removed these fields as part of your 1.12.0 upgrade,
the hook will simply do nothing (it checks if the config exists first).

### 3.3 GovCMS installation script fix

A minor fix to the GovCMS installation script ensures it creates the correct
theme directory path: `/app/web/themes/custom/civictheme`.

**Impact**: Only relevant for GovCMS projects. No changes required for
standard Drupal projects.

### 3.4 NPM package update

The `@civictheme/sdc` package has been updated from 1.12.0 to 1.12.1.

**Impact**: If you rebuild your sub-theme's front-end assets, you will
receive the updated package. This is a minor version bump with no breaking
changes.

### 3.5 Summary of changes

| Area | Change | Risk |
|------|--------|------|
| WYSIWYG preprocessing | Fixed filtering of embedded media | LOW |
| Email link conversion | Removed; delegated to text format | LOW |
| iframe attributes field | Automated removal via post-update hook | LOW |
| GovCMS script | Fixed directory path | N/A (GovCMS only) |
| NPM package | Updated to 1.12.1 | LOW |

---

## 4. Customisation inventory view (from register)

This spec assumes the project maintains a canonical customisation register
at:

- `docs/civic-theme-upgrades/customisations.md`

For this release, focus on entries that intersect with:

- **WYSIWYG field preprocessing**: If your sub-theme has custom WYSIWYG
  preprocessing or calls `_civictheme_process__html_content()` directly.
- **Email link handling**: If your sub-theme has custom email-to-link
  conversion logic.
- **iframe paragraph customisations**: If you have custom handling for
  iframe paragraphs (though the post-update hook should handle cleanup
  automatically).

Each relevant customisation SHOULD be cross-referenced by ID (e.g. `C5`,
`C7`) in the tasks and playbook.

---

## 5. Risk assessment and focus areas

Based on upstream changes documented in Section 3, this is a **low-risk**
upgrade.

### 5.1 LOW RISK – Standard upgrade path

- **WYSIWYG preprocessing fix**:
  - This is a bug fix that restores expected behaviour.
  - Embedded media (iframes, figures) that were being stripped in 1.12.0
    will now render correctly.
  - **Action**: No action required unless you implemented a workaround for
    the 1.12.0 issue (in which case, you can remove the workaround).

- **Email link conversion change**:
  - Email addresses are no longer automatically converted to `mailto:` links
    by CivicTheme preprocessing.
  - The `civictheme_rich_text` text format includes the "Convert URLs into
    links" filter by default, which handles this.
  - **Action**: If you have a custom text format that should convert emails
    to links, ensure the "Convert URLs into links" filter is enabled.

- **iframe attributes field removal**:
  - The post-update hook automates what was a manual step in 1.12.0.
  - **Action**: Run `drush updb` after upgrading. If you already removed
    the field manually, no action needed.

### 5.2 Project-specific risk areas

Projects SHOULD document their own risk areas here by referencing IDs from
the customisation register:

- `C001` – [Placeholder: list impacted customisation and why]

Review the customisation register at
`docs/civic-theme-upgrades/customisations.md` and annotate each entry
with its risk level for this upgrade.

---

## 6. Upgrade strategy overview

This is a straightforward bug-fix upgrade with minimal complexity.

### 6.1 Discovery phase

1. **Verify prerequisites**:
   - Confirm current CivicTheme version is `1.12.0`.
   - Confirm Drupal core version meets `^10.2 || ^11` requirement.
   - Ensure Git working tree is clean with a dedicated feature branch.

2. **Check for 1.12.0 workarounds**:
   - If you implemented workarounds for the WYSIWYG/embedded media issue,
     identify them for removal.

### 6.2 Change phase

1. **Update Drupal dependencies**:
   - Run `composer require drupal/civictheme:^1.12` (or update constraint
     to `^1.12.1` if pinning).
   - Verify `composer.lock` is regenerated.

2. **Remove workarounds** (if applicable):
   - Remove any workarounds implemented for the 1.12.0 WYSIWYG issue.

3. **Rebuild sub-theme assets** (optional):
   - Run front-end build to pick up the updated `@civictheme/sdc` package.

### 6.3 Validation phase

1. **Clear caches and run updates**:
   - `drush cr` (or equivalent).
   - `drush updb` – this will run the iframe attributes field removal hook.
   - `drush cim` if configuration changes are pending.

2. **Test WYSIWYG content**:
   - Verify pages with embedded media (iframes, figures) render correctly.
   - Verify email addresses in content are handled as expected.

3. **Update documentation**:
   - Mark completed tasks in `tasks.md`.
   - Record any lessons learned in `playbook.md`.

The detailed tasks and execution steps are defined in `tasks.md` and
`playbook.md` for this directory.

---

## 7. Guidance for AI coding assistants

When assisting with this upgrade, AI models SHOULD:

- Start with:
  - `docs/civic-theme-upgrades/customisations.md`
  - This file: `docs/civic-theme-upgrades/versions/v1.12.0-to-v1.12.1/spec.md`
  - The corresponding `tasks.md` and `playbook.md`.
- Understand that this is a **bug-fix release** with no new features or
  breaking changes.
- Use the upstream links in Section 2 to understand the context.
- Treat the customisation register as the source of truth for what must be
  preserved or adapted.
- Prefer:
  - Proposing minimal changes as this is a low-risk upgrade.
  - Avoiding destructive operations and any commands that act directly on
    production environments.
- Keep this spec and the customisation register up to date by:
  - Adding notes about which customisations were impacted or retired.
  - Recording any new customisations introduced as part of the upgrade.

---

## 8. Developer review checklist

Before considering the 1.12.0 → 1.12.1 upgrade complete, a developer
SHOULD confirm:

### Upgrade review

- [ ] CivicTheme version is 1.12.1 (`composer show drupal/civictheme`).
- [ ] Database updates have been run (`drush updb`).
- [ ] The `field_c_p_attributes` field has been removed from iframe
      paragraphs (automatic via post-update hook).

### Functional review

- [ ] The project builds successfully and basic smoke tests (cache clear,
      config import, entity updates) complete without errors.
- [ ] Pages with embedded media (iframes, figures, etc.) render correctly.
- [ ] WYSIWYG content displays as expected.
- [ ] Email addresses in content are handled correctly (either as plain
      text or as links, depending on text format configuration).

### Customisation register

- [ ] The customisation register has been reviewed for any entries that
      may have been affected by this upgrade.
- [ ] Any 1.12.0 workarounds have been identified and removed.

### Documentation

- [ ] Any regressions or follow-up tasks are captured in `tasks.md` or
      in a project issue tracker.
- [ ] This spec, `tasks.md` and `playbook.md` accurately reflect what was
      performed, so that future upgrades can rely on them as history.

