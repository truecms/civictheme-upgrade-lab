# CivicTheme upgrade spec: v1.11.0 → v1.12.0

**Project**: Destination Drupal project using CivicTheme  
**CivicTheme source version**: `1.11.0`  
**CivicTheme target version**: `1.12.0`  
**Per-version directory**: `docs/civic-theme-upgrades/versions/v1.11.0-to-v1.12.0/`

This document defines the canonical per-version specification for the
CivicTheme `1.11.0 → 1.12.0` upgrade, using the framework defined in
`specs/001-rely-contents-docs/spec.md`. Destination projects SHOULD use
this spec (and its companion `tasks.md` and `playbook.md`) as the primary
instructions for performing this upgrade, adapting only where
project-specific customisations require it.

---

## 1. Environment assumptions and scope

- All upgrade work happens in **non-production** or otherwise safely
  isolated environments (e.g. feature branches, staging sites, local
  environments).
- The project has:
  - A working Drupal code base using CivicTheme `1.11.0`.
  - Composer-based dependency management.
  - Git version control with access to history for custom themes.
- Drupal core requirements remain unchanged from 1.11.0:
  - `core_version_requirement: ^10.2 || ^11` (same as 1.11.0).
  - If the project successfully upgraded to 1.11.0, no additional Drupal
    core changes are required for 1.12.0.
- Standard organisational safeguards apply:
  - Backups and/or database snapshots exist prior to running the upgrade.
  - Deployments are reviewed via merge requests or equivalent.
- This spec focuses on the **CivicTheme 1.11.0 → 1.12.0** step only. Any
  earlier or later version jumps MUST be documented in separate directories.

Out of scope:

- Production deployment steps.
- The detailed mechanics of upgrading Drupal core itself.

---

## 2. Upstream references for 1.12.0

Authors and AI assistants MUST consult the following upstream sources when
preparing and validating this upgrade:

- **Release notes**: CivicTheme `1.12.0` on drupal.org  
  `https://www.drupal.org/project/civictheme/releases/1.12.0`
- **CivicTheme documentation portal (including 1.12.0 release notes)**:  
  `https://docs.civictheme.io/changelog`
- **Security advisories**:
  - SA-CONTRIB-2025-112 (Information disclosure – Moderately critical)
  - SA-CONTRIB-2025-113 (Cross-site Scripting – Moderately critical)
- **Manual security mitigation instructions** (if unable to update immediately):  
  `https://docs.civictheme.io/` – Security update - 1.12.0 section
- **Upstream code diff** (1.11.0 → 1.12.0):  
  `https://git.drupalcode.org/project/civictheme/-/compare/1.11.0...1.12.0?from_project_id=86817`

When maintaining this spec in a real project, copy or summarise the
relevant sections from these sources to keep this document authoritative
for the `1.11.0 → 1.12.0` upgrade.

---

## 3. High-level upstream changes (1.11.0 → 1.12.0)

From the release notes (drupal.org), CivicTheme documentation portal
(docs.civictheme.io) and the 1.11.0 → 1.12.0 git diff, CivicTheme 1.12.0
introduces the following changes:

### 3.1 Security fixes – CRITICAL

This release contains fixes for two **moderately critical** security
vulnerabilities. Projects running CivicTheme 1.11.0 or earlier are at risk
and SHOULD upgrade to 1.12.0 as soon as practical.

| Advisory | Severity | Type |
|----------|----------|------|
| SA-CONTRIB-2025-112 | Moderately critical | Information disclosure |
| SA-CONTRIB-2025-113 | Moderately critical | Cross-site Scripting (XSS) |

**Sub-theme risk**: Sub-themes that override CivicTheme templates may also
carry the XSS and information disclosure risks. After upgrading, review
any sub-theme customisations against the manual mitigation instructions
to assess risk.

**Manual mitigation**: If unable to upgrade immediately, consult the
"Security update - 1.12.0" documentation at docs.civictheme.io for manual
remediation steps.

### 3.2 New features

#### 3.2.1 Multi-line header

The header component can now be configured so the primary navigation uses
the full width of the container, with the logo displayed in a container
above. This is helpful for sites that:

- Require more than 4 primary navigation items.
- Have a wide or co-branded logo.

**Impact**: New configuration option only. Existing header configurations
remain unchanged unless explicitly modified.

#### 3.2.2 Message organism

A new component with four styling options to highlight important messages
anywhere within a page.

**Impact**: New component available for use. No changes required to
existing content.

#### 3.2.3 Inline filter (search results uplift)

Search results now display the keyword in the search results message, both
visually and in code for screen reading technology.

**Impact**: Improved UX for search results. Verify search result pages
display correctly after upgrade.

#### 3.2.4 Fast fact card

A new card type that can be used to display simple facts about an
organisation.

**Impact**: New card variant available for use. No changes required to
existing content.

#### 3.2.5 GovCMS automation script

An automation script has been added for GovCMS site setup.

**Impact**: Relevant only for GovCMS projects. No changes required for
other projects.

### 3.3 Accessibility improvements

| Area | Change |
|------|--------|
| Video transcript | Improved video transcript functionality |
| Card design | Card design updates implemented |
| Link atom | Updates to Link atom and content links |
| Search results | Improved search results messaging |

**Impact**: These are improvements to existing components. Verify that
accessible features continue to work correctly, particularly video
transcripts and card interactions.

### 3.4 SDC continued improvement

Building on the SDC migration in 1.11.0, this release continues to improve
Single Directory Components:

- **CSS versions of SCSS variables added**: CSS custom properties are now
  available alongside SCSS variables for theming flexibility.
- **CSS variables for typography use**: Typography now uses CSS variables,
  enabling easier customisation without SCSS compilation.
- **SDC validation and error handling**: Improved validation and error
  handling for Single Directory Components.

**Impact**: These are internal improvements that enhance SDC support. If
your sub-theme uses SCSS variables for theming, consider migrating to the
new CSS custom properties for improved flexibility.

### 3.5 Bug fixes and improvements

| Issue | Fix |
|-------|-----|
| Menu theming | Enhanced menu theming system |
| Archived webforms | Fixed site breaking when viewing nodes that reference archived webforms |
| Search form | Search form element issues resolved |
| Email links | Fixed email addresses being incorrectly converted to links |
| Logo paths | Fixed issues caused by logo paths with spaces |
| Webform messages | Fixed striptags error on webform messages |
| PHP Twig bug | Fixed `menu_level_modifier_class` not in scope |
| Attachment component | Fixed attachment component not resetting file extension from previous items |
| Mobile menu | Fixed mobile menu visibility problems on non-drawer/dropdown links |

**Impact**: These are bug fixes that improve stability. Test any areas of
your site that may have been affected by these issues.

### 3.6 Summary of changes requiring sub-theme review

Unlike the 1.10.0 → 1.11.0 upgrade (which required significant SDC
migration work), the 1.11.0 → 1.12.0 upgrade is **primarily additive**.
However, sub-themes SHOULD review:

| Area | Action Required |
|------|-----------------|
| Security patches | Verify sub-theme overrides don't reintroduce XSS/disclosure vulnerabilities |
| Menu overrides | Review if sub-theme menu customisations need updating for enhanced theming |
| CSS variables | Consider adopting new CSS custom properties if currently using SCSS variables |
| Attachment overrides | Verify attachment component overrides handle file extension reset correctly |

---

## 4. Customisation inventory view (from register)

This spec assumes the project maintains a canonical customisation register
at:

- `docs/civic-theme-upgrades/customisations.md`

For illustration, the example register in this repo uses placeholder
entries such as:

- `C001` – Example sub-theme overrides (Twig, SCSS, JS).

In a real project, replace the placeholder with your actual inventory.
For 1.11.0 → 1.12.0, focus on entries that intersect with:

- **Templates affected by security fixes**: Any overrides to components
  that may expose user data or allow script injection.
- **Menu templates**: Sub-theme menu overrides that may need adjustment
  for the enhanced theming system.
- **Attachment templates**: Overrides to attachment/file components.
- **Header templates**: If planning to adopt the multi-line header feature.
- **Card templates**: If using custom card styling that may conflict with
  the card design updates.
- **Search result templates**: If customising search result display.

Each relevant customisation SHOULD be cross-referenced by ID (e.g. `C5`,
`C7`) in the tasks and playbook.

---

## 5. Risk assessment and focus areas

Based on upstream changes documented in Section 3 and the customisation
inventory, risk areas for this upgrade are categorised by severity.

### 5.1 HIGH RISK – Requires immediate action

- **Security vulnerabilities (CRITICAL)**:
  - The primary driver for this upgrade is the security fixes for
    information disclosure and XSS.
  - **Action**: Upgrade to 1.12.0 as soon as practical. If unable to
    upgrade immediately, apply manual mitigations.
  - **Sub-theme review**: After upgrading, audit any sub-theme overrides
    for components that handle user input or display user-generated
    content to ensure they don't reintroduce the patched vulnerabilities.

### 5.2 MEDIUM RISK – Requires review and possible action

- **Menu customisations**:
  - The enhanced menu theming system may affect sub-themes with menu
    overrides.
  - **Action**: Review menu-related templates and SCSS for compatibility.

- **Attachment component overrides**:
  - Bug fix changes how file extensions are handled.
  - **Action**: If you have attachment component overrides, verify they
    work correctly with the new file extension reset behaviour.

- **CSS/SCSS variable usage**:
  - New CSS custom properties are available. While SCSS variables continue
    to work, this may be a good opportunity to modernise sub-theme styling.
  - **Action**: Evaluate whether to migrate from SCSS variables to CSS
    custom properties for easier runtime customisation.

### 5.3 LOW RISK – Review recommended

- **New components (Message organism, Fast fact card)**:
  - These are new components that don't affect existing functionality.
  - **Action**: Consider adopting if useful for your site's content needs.

- **Multi-line header option**:
  - New configuration option for header layout.
  - **Action**: Consider enabling if your site has navigation/logo
    constraints that would benefit from this layout.

- **Accessibility improvements**:
  - Video transcript, card design, and search messaging improvements.
  - **Action**: Verify existing accessible features continue to work.

- **Bug fixes for webforms, logo paths, email links**:
  - If you experienced any of these issues, verify they are resolved.
  - **Action**: Test affected areas if previously problematic.

### 5.4 Project-specific risk areas

Projects SHOULD document their own high-risk customisations here by
referencing IDs from the customisation register:

- `C001` – [Placeholder: list impacted customisation and why]
- `C002` – [Placeholder: list impacted customisation and why]

Review the customisation register at
`docs/civic-theme-upgrades/customisations.md` and annotate each entry
with its risk level for this upgrade.

---

## 6. Upgrade strategy overview

The intended strategy for this upgrade is structured in three phases.
Compared to 1.10.0 → 1.11.0, this upgrade is significantly simpler as it
does not involve architectural changes like the SDC migration.

### 6.1 Discovery phase

1. **Verify prerequisites**:
   - Confirm current CivicTheme version is `1.11.0` (or 1.11.x).
   - Confirm Drupal core version meets `^10.2 || ^11` requirement.
   - Ensure Git working tree is clean with a dedicated feature branch.
   - **Note**: If using Docker-based local development, commands should
     run inside the CLI container via `ahoy` or `docker compose exec cli`.

2. **Discover and run tests (baseline)**:
   - Check `.ahoy.yml` for test commands: `grep -i "test" .ahoy.yml`.
   - Check `composer.json` for test scripts.
   - Run all available tests and record results as baseline.
   - **Stop condition**: If critical tests fail before upgrade, fix them
     first. Do not upgrade on a broken baseline.

3. **Review security advisory impact**:
   - Read SA-CONTRIB-2025-112 and SA-CONTRIB-2025-113.
   - Identify any sub-theme templates that may need review for similar
     vulnerabilities.

4. **Audit sub-theme for affected areas**:
   - Search for menu-related template overrides.
   - Search for attachment component overrides.
   - Review any templates handling user input or displaying user content.

5. **Refresh customisation register**:
   - Update `docs/civic-theme-upgrades/customisations.md` with findings.
   - Mark each customisation with impact level (HIGH/MEDIUM/LOW) based
     on Section 5 risk assessment.

### 6.2 Change phase

1. **Update Drupal dependencies**:
   - Run `composer require drupal/civictheme:^1.12` (or update
     `composer.json` constraint manually).
   - Ensure `composer.lock` is regenerated.
   - Run `composer install` to verify lock file consistency.

2. **Review sub-theme security** (HIGH PRIORITY):
   - Audit sub-theme template overrides against security advisory
     mitigations.
   - Apply fixes to any sub-theme code that may reintroduce patched
     vulnerabilities.

3. **Update menu customisations** (if applicable):
   - Review menu-related templates for compatibility with enhanced
     theming system.
   - Test menu rendering across desktop and mobile.

4. **Update attachment overrides** (if applicable):
   - Verify attachment component overrides work correctly with new file
     extension handling.

5. **Consider CSS variable migration** (optional):
   - If currently using SCSS variables, consider migrating to CSS custom
     properties for improved flexibility.

6. **Rebuild sub-theme assets**:
   - Run front-end build: `ahoy fe` or `npm run build`.
   - Verify output files are generated correctly.

### 6.3 Validation phase

1. **Clear caches and run updates**:
   - `drush cr` (or equivalent).
   - `drush updb` if database updates are pending.
   - `drush cim` if configuration changes are pending.

2. **Security validation**:
   - Verify that information disclosure and XSS vulnerabilities are
     patched (consult security advisory test cases if available).
   - Review sub-theme overrides for any reintroduced vulnerabilities.

3. **Test key pages and components**:
   - Home page.
   - Navigation (header, footer, mobile menu) – especially if menu
     customisations exist.
   - Search results (verify inline filter messaging).
   - Content pages with attachments.
   - Forms, especially webforms.
   - Video content with transcripts.

4. **Test new features** (optional):
   - Multi-line header configuration (if adopting).
   - Message organism (if adopting).
   - Fast fact card (if adopting).

5. **Update documentation**:
   - Mark completed tasks in `tasks.md`.
   - Update customisation register with outcomes.
   - Record lessons learned in `playbook.md`.

The detailed tasks and execution steps are defined in `tasks.md` and
`playbook.md` for this directory.

---

## 7. Guidance for AI coding assistants

When assisting with this upgrade, AI models SHOULD:

- Start with:
  - `docs/civic-theme-upgrades/customisations.md`
  - This file: `docs/civic-theme-upgrades/versions/v1.11.0-to-v1.12.0/spec.md`
  - The corresponding `tasks.md` and `playbook.md`.
- Understand that this is a **security-focused release** with incremental
  improvements, not an architectural change like 1.11.0.
- Use the upstream links in Section 2 to understand the security context.
- Treat the customisation register as the source of truth for what must be
  preserved or adapted.
- Prefer:
  - Proposing changes as diffs that keep customisations aligned with
    upstream CivicTheme.
  - Avoiding destructive operations and any commands that act directly on
    production environments.
- Prioritise the security aspects of this upgrade – the XSS and
  information disclosure fixes are the primary reason to upgrade.
- Keep this spec and the customisation register up to date by:
  - Adding notes about which customisations were impacted or retired.
  - Recording any new customisations introduced as part of the upgrade.

---

## 8. Human review checklist

Before considering the 1.11.0 → 1.12.0 upgrade complete, a human reviewer
SHOULD confirm:

- [ ] The security advisories SA-CONTRIB-2025-112 and SA-CONTRIB-2025-113
      have been addressed by the upgrade.
- [ ] Sub-theme overrides have been reviewed to ensure they don't
      reintroduce the patched vulnerabilities.
- [ ] The customisation register has been refreshed and all known
      CivicTheme-related customisations are listed with stable IDs.
- [ ] All customisations identified as affected by 1.12.0 have been
      reviewed and either updated, replaced or explicitly retired.
- [ ] The project builds successfully and basic smoke tests (cache clear,
      config import, entity updates) complete without errors.
- [ ] Key pages and components that rely on CivicTheme (home, search,
      navigation, attachments, menus, forms) have been manually reviewed.
- [ ] Any regressions or follow-up tasks are captured in
      `tasks.md` or in a project issue tracker.
- [ ] This spec, `tasks.md` and `playbook.md` accurately reflect what was
      performed, so that future upgrades can rely on them as history.

