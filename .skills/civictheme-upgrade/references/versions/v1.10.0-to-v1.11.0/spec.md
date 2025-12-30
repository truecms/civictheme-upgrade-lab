# CivicTheme upgrade spec: v1.10.0 → v1.11.0

**Project**: Destination Drupal project using CivicTheme  
**CivicTheme source version**: `1.10.0`  
**CivicTheme target version**: `1.11.0`  
**Per-version directory**: `docs/civic-theme-upgrades/versions/v1.10.0-to-v1.11.0/`

This document defines the canonical per-version specification for the
CivicTheme `1.10.0 → 1.11.0` upgrade, using the framework defined in
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
  - A working Drupal code base using CivicTheme `1.10.0`.
  - Composer-based dependency management.
  - Git version control with access to history for custom themes.
- Drupal core is already on a version compatible with CivicTheme 1.11.0:
  - Upstream `civictheme.info.yml` changes the requirement to  
    `core_version_requirement: ^10.2 || ^11`.
  - If the project is still on Drupal `<10.2`, plan and execute the
    Drupal core upgrade as a **separate** (but prerequisite) piece of
    work before applying this CivicTheme step.
- Standard organisational safeguards apply:
  - Backups and/or database snapshots exist prior to running the upgrade.
  - Deployments are reviewed via merge requests or equivalent.
- **Existing `.gitignore` files MUST NOT be modified.** The assumption is that
  the theme is already operational and its ignored files and folders (such as
  `node_modules/`, `dist/`, vendor directories, and build artefacts) are
  correctly configured. Modifying `.gitignore` can disrupt the existing build
  pipeline, accidentally commit generated files, or break local development
  environments.
- This spec focuses on the **CivicTheme 1.10.0 → 1.11.0** step only. Any
  earlier or later version jumps MUST be documented in separate directories.

Out of scope:

- Production deployment steps.
- The detailed mechanics of upgrading Drupal core itself (these MAY be
  referenced where required by CivicTheme but are not the primary focus).

---

## 2. Upstream references for 1.11.0

Authors and AI assistants MUST consult the following upstream sources when
preparing and validating this upgrade:

- **Release notes**: CivicTheme `1.11.0` on drupal.org  
  `https://www.drupal.org/project/civictheme/releases/1.11.0`
- **CivicTheme documentation portal (including 1.11.0 release notes)**:  
  `https://docs.civictheme.io/`
- **Single Directory Components (SDC) guidance** for CivicTheme and
  Drupal core 10.2+/10.3+ (see relevant sections under
  `https://docs.civictheme.io`).
- **CivicTheme Upgrade Tools – SDC update script** (for optional,
  automated assistance converting sub-theme components to SDC):
  - Repository: `https://github.com/civictheme/upgrade-tools`
  - Tool: `sdc-update` (CivicTheme SDC Update Tool).
  - High-level usage (see its README for full details):
    - This helper is **strictly optional** – a manual build tooling update
      path is always available and does **not** require an Anthropic API key.
      If you do not want to configure a key, skip this helper and follow the
      manual steps in Section 6.2 instead.
    - When used, it requires Node.js 22+, a CivicTheme sub-theme already
      updated to CivicTheme 1.10.x, and an Anthropic API key.
      - **⛔ STOP CONDITION for AI assistants:** The tool requires an
        `ANTHROPIC_API_KEY`. Before executing, AI assistants MUST stop and
        request developer confirmation that the key is available:
        - **Preferred:** Add `ANTHROPIC_API_KEY=sk-ant-...` to the destination
          project's `.env` file.
        - **Alternative:** Export the key in the shell session or add to
          `~/.bashrc` / `~/.zshrc`.
        - Only proceed with this optional helper after the developer confirms
          the key is configured; if they decline or no key is available, do
          **not** run the tool and instead use the manual path.
        - **Critical:** Remind developer to remove/unset the key after upgrade
          completes (see T119 in `tasks.md` for removal steps and T129 for
          verification).
    - Run in a throwaway working copy or feature branch only (it
      updates `package.json`, Storybook config, `build.js` and
      component directories).
    - Typical flow: clone the repo, run `npm install`, configure a
      `.env` file (sub-theme path and, when using AI-powered story
      conversion, an Anthropic API key and optional model) and execute
      `npm run update-components`, then review changes and `.logs/`
      output.
- **Upstream code diff** (1.10.0 → 1.11.0):  
  `https://git.drupalcode.org/project/civictheme/-/compare/1.10.0...1.11.0?from_project_id=86817`

When maintaining this spec in a real project, copy or summarise the
relevant sections from these sources to keep this document authoritative
for the `1.10.0 → 1.11.0` upgrade.

---

## 3. High-level upstream changes (1.10.0 → 1.11.0)

From the release notes (drupal.org), CivicTheme documentation portal
(docs.civictheme.io) and the 1.10.0 → 1.11.0 git diff, CivicTheme 1.11.0
introduces the following changes:

### 3.1 Single Directory Components (SDC) migration – BREAKING

This is the **major architectural change** in 1.11.0. All CivicTheme
components are now SDC-based, affecting how components are included and
extended in sub-themes.

- **Component extension model changes**:
  - **Removed**: Direct extension of CivicTheme base components in
    sub-themes is no longer supported.
  - **Supported inheritance types**:
    - ✅ Custom components in sub-theme (new components that do not exist
      in CivicTheme).
    - ✅ Override CivicTheme components completely (replace entire
      component in sub-theme).
    - ❌ Extend CivicTheme components partially (e.g. adding to a parent
      template via `{{ parent() }}`) – **no longer supported**.
  - Sub-themes that extend upstream CivicTheme components must be
    refactored to either:
    - Override the component completely, or
    - Remove the extension and use the upstream component unchanged.

- **New `.component.yml` definitions**: Every CivicTheme component now
  has a `.component.yml` file defining its schema, props and metadata.
  Example props schema from `button.component.yml`:
  - `theme` (light/dark), `kind`, `type`, `size`, `icon`, `text`, `url`,
    `is_external`, `is_disabled`, `modifier_class`, etc.

- **Per-component CSS files**: Each component directory now includes a
  compiled `.css` file (e.g. `button.css`, `chip.css`).

### 3.2 Twig include syntax – BREAKING

The Twig namespace syntax for including CivicTheme components has changed.

| Before (1.10.0) | After (1.11.0) |
|-----------------|----------------|
| `{% include '@atoms/paragraph/paragraph.twig' %}` | `{% include 'civictheme:paragraph' %}` |
| `{% include '@molecules/logo/logo.twig' %}` | `{% include 'civictheme:logo' %}` |
| `{% include '@base/icon/icon.twig' %}` | `{% include 'civictheme:icon' %}` |
| `{% include '@organisms/header/header.twig' %}` | `{% include 'civictheme:header' %}` |

All Twig templates (sub-theme overrides and custom templates) that include
CivicTheme components via the old `@atoms`, `@molecules`, `@organisms` or
`@base` namespaces MUST be updated to use the new `civictheme:<component>`
SDC syntax.

### 3.3 Twig block naming convention – BREAKING

Twig block names in component templates have been renamed from the
`<name>_slot` pattern to `<name>_block`:

| Before (1.10.0) | After (1.11.0) |
|-----------------|----------------|
| `{% block content_slot %}` | `{% block content_block %}` |
| `{% block content_top_slot %}` | `{% block content_top_block %}` |
| `{% block content_bottom_slot %}` | `{% block content_bottom_block %}` |
| `{% block sidebar_top_left_slot %}` | `{% block sidebar_top_left_block %}` |
| `{% block sidebar_bottom_right_slot %}` | `{% block sidebar_bottom_right_block %}` |
| `{% block rows_slot %}` | `{% block rows_block %}` |
| `{% block footer_slot %}` | `{% block footer_block %}` |
| `{% block hidden_slot %}` | `{% block hidden_block %}` |

Sub-theme templates that override blocks using the `_slot` suffix MUST
update to use the `_block` suffix.

### 3.4 Asset restructuring – BREAKING

Global CSS and JS outputs are restructured with new file names:

**CSS changes** (in `civictheme.libraries.yml`):

| Before (1.10.0) | After (1.11.0) |
|-----------------|----------------|
| `dist/civictheme.css` | Split into three files: |
| | `dist/civictheme.base.css` |
| | `dist/civictheme.theme.css` |
| | `dist/civictheme.variables.css` |

**JS changes**:

| Before (1.10.0) | After (1.11.0) |
|-----------------|----------------|
| `dist/civictheme.js` | `dist/civictheme.drupal.base.js` |

**Sub-theme library overrides** in `<subtheme>.info.yml` must be updated:

```yaml
# Before (1.10.0)
libraries-override:
  civictheme/global:
    css:
      theme:
        dist/civictheme.css: dist/styles.css
    js:
      dist/civictheme.js: dist/scripts.js

# After (1.11.0)
libraries-override:
  civictheme/global:
    css:
      theme:
        dist/civictheme.base.css: dist/styles.base.css
        dist/civictheme.theme.css: dist/styles.theme.css
        dist/civictheme.variables.css: dist/styles.variables.css
    js:
      dist/civictheme.drupal.base.js: dist/scripts.drupal.base.js
```

### 3.5 New theme hook and component

- Added `civictheme_table_of_contents` theme hook with template at
  `templates/misc/table-of-contents.html.twig`.
- Variables: `component_theme`, `title`, `position`, `links`,
  `anchor_selector`, `scope_selector`, `content`, `attributes`,
  `modifier_class`.

### 3.6 SDC plugin manager integration

`civictheme.theme` now integrates with Drupal's SDC plugin manager to
attach the `civictheme:alert` component library for authenticated users:

```php
$component_plugin_manager = \Drupal::service('plugin.manager.sdc');
try {
  $alert = $component_plugin_manager->find('civictheme:alert');
  $attachments['#attached']['library'][] = $alert->getLibraryName();
}
catch (ComponentNotFoundException $exception) {
  \Drupal::logger('civictheme')->error('Unable to find alert component');
}
```

### 3.7 Starter kit and build tooling updates

- **New files**:
  - `.storybook/sdc-plugin.js` (Storybook SDC integration).
  - `build-config.json` extended with new SDC build targets.
- **Updated files**:
  - `build.js` – major refactor with SDC-aware compilation, new output
    targets (`sdc_base`, `sdc_components`, `styles_theme`), automatic
    CivicTheme directory detection.
  - `package.json` – updated dependencies including `@civictheme/sdc`.
  - `.storybook/preview.js` – updated for SDC.
  - `vite.config.js` – extended for SDC builds.
- **Sub-theme `info.yml`**:
  - Added `enforce_prop_schemas: true` for SDC prop validation.

### 3.8 API and hook changes

- `hook_civictheme_automated_list_view_info_alter()` example in
  `civictheme.api.php` now uses `'civictheme_event'` instead of generic
  `'event'` as the content type machine name.

### 3.9 Bug fixes and improvements

- **[#3518669]** Search displays results in correct relevance order (was
  showing least relevant first).
- **[#3522101]** Fixed exception when filter label not set in automated
  list.
- **[#3522098]** Added CSS class passing from view display to template for
  Automated List.
- **[#3522109]** Removed legacy IE10 font.
- **[#3520948]** Removed Shortcut from theme dependency.
- **[#3520413]** Fixed lint standards.
- **[#3500869]** Fixed `setSyncing()` on null if Drupal view is removed and
  post_update hook runs.

### 3.10 Summary of breaking changes requiring sub-theme action

| Area | Impact | Action Required |
|------|--------|-----------------|
| Component extension | Cannot extend CivicTheme components | Override completely or use upstream |
| Twig includes | Old namespace syntax broken | Update to `civictheme:<component>` |
| Twig blocks | `_slot` suffix no longer exists | Update to `_block` suffix |
| Asset files | Old CSS/JS file names removed | Update library overrides |
| Build tooling | Old build scripts incompatible | Update `build.js`, `package.json` |

---

## 4. Customisation inventory view (from register)

This spec assumes the project maintains a canonical customisation register
at:

- `docs/civic-theme-upgrades/customisations.md`

For illustration, the example register in this repo uses placeholder
entries such as:

- `C001` – Example sub-theme overrides (Twig, SCSS, JS).

In a real project, replace the placeholder with your actual inventory.
For 1.10.0 → 1.11.0, focus on entries that intersect with:

- Components that are being converted to SDC, or that now have
  `.component.yml` definitions and/or per-component `.css` bundles.
- Twig templates that override upstream CivicTheme templates, especially
  components under `components/00-base`, `01-atoms`, `02-molecules` and
  `03-organisms`.
- SCSS/JS that depends on specific component markup or classes, or on
  global assets previously exposed as `dist/civictheme.css` or
  `dist/civictheme.js`.
- Configuration overrides (e.g. view modes, blocks, menus, Layout
  Builder configuration) tied to CivicTheme components, including
  navigation, alerts, table-of-contents and field/paragraph displays.

Each relevant customisation SHOULD be cross-referenced by ID (e.g. `C5`,
`C7`) in the tasks and playbook.

---

## 5. Risk assessment and focus areas

Based on upstream changes documented in Section 3 and the customisation
inventory, risk areas for this upgrade are categorised by severity.

### 5.1 HIGH RISK – Requires immediate action

- **Extended CivicTheme components (CRITICAL)**:
  - Sub-themes that use `{% extends %}` or `{{ parent() }}` on CivicTheme
    templates will **break**. The SDC architecture does not support
    partial extension.
  - **Action**: Identify all extended templates; refactor to either:
    - Override completely (copy entire upstream template to sub-theme), or
    - Remove override and accept upstream defaults.
  - Check for Twig templates with patterns like:
    - `{% extends '@atoms/...' %}`
    - `{% extends '@molecules/...' %}`
    - `{{ parent() }}` calls in CivicTheme overrides.

- **Twig include statements**:
  - All `{% include '@atoms/...' %}`, `{% include '@molecules/...' %}`,
    `{% include '@organisms/...' %}` and `{% include '@base/...' %}`
    patterns must be updated to `{% include 'civictheme:...' %}`.
  - **Impact**: Templates will fail to render or produce errors.
  - **Scope**: Every sub-theme Twig file that includes CivicTheme
    components.

- **Twig block names (`_slot` → `_block`)**:
  - Templates overriding blocks with `_slot` suffix will silently fail
    (block content ignored).
  - **Action**: Search and replace all `_slot` block references with
    `_block`.

- **Library override file names**:
  - Sub-themes with `libraries-override` in `<subtheme>.info.yml`
    referencing old asset names (`dist/civictheme.css`,
    `dist/civictheme.js`) will fail to load styles/scripts.
  - **Action**: Update to new split file names per Section 3.4.

### 5.2 MEDIUM RISK – Requires review and possible action

- **Drupal core compatibility**:
  - CivicTheme 1.11.0 requires `core_version_requirement: ^10.2 || ^11`.
  - Sites on Drupal 10.0 or 10.1 **cannot** upgrade without first
    upgrading Drupal core.
  - **Action**: Verify Drupal core version; if <10.2, plan core upgrade
    as a prerequisite.

- **Build tooling (`build.js`, `package.json`)**:
  - Sub-themes derived from the starter kit will have incompatible build
    scripts.
  - **Action**: Use the CivicTheme SDC Update Tool or manually update
    build configuration.
- **Component completeness (missing Twig template)**:
  - After SDC conversion, a custom component directory that keeps only a
    `.component.yml` without a matching `.twig` will trigger
    `InvalidComponentException: Unable to find the Twig template for the
    component`.
  - **Action**: Ensure each component has a Twig template; copy the upstream
    component (e.g. Table of Contents) into the sub-theme or remove the stale
    `.component.yml` before clearing caches.

- **Custom components without SDC metadata**:
  - Custom sub-theme components may need `.component.yml` files to be
    properly discovered by Drupal's SDC system.
  - **Action**: Consider running the SDC Update Tool or manually creating
    component schemas.

- **Storybook configuration**:
  - Sub-themes using Storybook require updated `.storybook/` config files,
    including the new `sdc-plugin.js`.

### 5.3 LOW RISK – Review recommended

- **Configuration dependencies**:
  - Views, blocks and Layout Builder sections using CivicTheme components
    should be tested for correct rendering.
  - Pay attention to: alerts, table-of-contents, automated lists, cards.

- **API hook implementations**:
  - If the project implements `hook_civictheme_automated_list_view_info_alter()`,
    verify content type machine names (example changed from `'event'` to
    `'civictheme_event'`).

- **Search result ordering**:
  - Search was previously returning least relevant results first; this is
    now fixed. Verify search behaviour meets expectations.

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
Given the significance of the SDC migration, this upgrade requires more
thorough preparation than typical minor version bumps.

### 6.1 Discovery phase

1. **Verify prerequisites**:
   - Confirm current CivicTheme version is `1.10.0`.
   - Confirm Drupal core version is `>=10.2` (CivicTheme 1.11.0 requires
     `^10.2 || ^11`). If Drupal is <10.2, **stop** and plan a separate
     core upgrade first.
   - Ensure Git working tree is clean with a dedicated feature branch.
   - **Note**: If using Docker-based local development, commands should
     run inside the CLI container via `ahoy` or `docker compose exec cli`.

2. **Discover and run tests (baseline)**:
   - Check `.ahoy.yml` for test commands: `grep -i "test" .ahoy.yml`.
   - Check `composer.json` for test scripts: look for `test-*` scripts.
   - Common test commands: `ahoy test-bdd`, `ahoy test-unit`,
     `ahoy test-playwright`, `composer test`.
   - Run all available tests and record results as baseline.
   - **Stop condition**: If critical tests fail before upgrade, fix them
     first. Do not upgrade on a broken baseline.

2. **Audit sub-theme Twig templates**:
   - Search for all `{% include '@atoms/` patterns.
   - Search for all `{% include '@molecules/` patterns.
   - Search for all `{% include '@organisms/` patterns.
   - Search for all `{% include '@base/` patterns.
   - Search for all `{% extends ` patterns in CivicTheme overrides.
   - Search for all `{{ parent() }}` calls in CivicTheme overrides.
   - Search for all `_slot %}` block patterns.
   - Record count and file locations in the customisation register.

3. **Audit library overrides**:
   - Check `<subtheme>.info.yml` for `libraries-override` referencing:
     - `dist/civictheme.css`
     - `dist/civictheme.js`
   - Check `<subtheme>.libraries.yml` for similar references.

4. **Refresh customisation register**:
   - Update `docs/civic-theme-upgrades/customisations.md` with findings.
   - Mark each customisation with impact level (HIGH/MEDIUM/LOW) based
     on Section 5 risk assessment.

### 6.2 Change phase

1. **Update Drupal dependencies**:
   - Run `composer require drupal/civictheme:1.11.0` to install exactly
     version 1.11.0. **Do NOT use caret (^) or tilde (~) constraints**;
     sequential upgrades require pinning to the precise target version.
   - Ensure `composer.lock` is regenerated.
   - Run `composer install` to verify lock file consistency.

2. **Update Twig include statements** (HIGH PRIORITY):
   - Replace `{% include '@atoms/<component>/<component>.twig'` with
     `{% include 'civictheme:<component>'`.
   - Replace `{% include '@molecules/<component>/<component>.twig'` with
     `{% include 'civictheme:<component>'`.
   - Replace `{% include '@organisms/<component>/<component>.twig'` with
     `{% include 'civictheme:<component>'`.
   - Replace `{% include '@base/<component>/<component>.twig'` with
     `{% include 'civictheme:<component>'`.

3. **Refactor extended components** (HIGH PRIORITY):
   - For each template using `{% extends %}` or `{{ parent() }}`:
     - **Option A**: Override completely by copying the full upstream
       template to sub-theme and applying customisations inline.
     - **Option B**: Remove the override entirely if upstream defaults
       are acceptable.

4. **Update Twig block names**:
   - Replace all `_slot %}` with `_block %}` in sub-theme templates.
   - Common replacements:
     - `content_slot` → `content_block`
     - `content_top_slot` → `content_top_block`
     - `content_bottom_slot` → `content_bottom_block`
     - `rows_slot` → `rows_block`
     - `footer_slot` → `footer_block`
     - `hidden_slot` → `hidden_block`

5. **Update library overrides** in `<subtheme>.info.yml`:
   ```yaml
   libraries-override:
     civictheme/global:
       css:
         theme:
           dist/civictheme.base.css: dist/styles.base.css
           dist/civictheme.theme.css: dist/styles.theme.css
           dist/civictheme.variables.css: dist/styles.variables.css
       js:
         dist/civictheme.drupal.base.js: dist/scripts.drupal.base.js
   ```

6. **Update build tooling** (if using starter kit build system):
   - **Option A – Use SDC Update Tool** (recommended):
     - **⛔ STOP CONDITION:** AI assistants MUST stop and request developer
       confirmation that `ANTHROPIC_API_KEY` is available before proceeding.
       - **Preferred:** Add to the destination project's `.env` file.
       - **Alternative:** Export in shell or add to `~/.bashrc` / `~/.zshrc`.
       - Only proceed with this optional helper after the developer confirms.
       - If the developer does **not** want to provide a key or no key is
         available, skip Option A entirely and use Option B (manual update)
         instead – the CivicTheme upgrade remains valid without the helper.
       - **Critical:** After upgrade completes, remove the key (see T119) and
         verify removal (see T129) to prevent accidental commit.
     - Clone `https://github.com/civictheme/upgrade-tools`.
     - Install dependencies: `npm install`.
     - Configure `.env` with sub-theme path and Anthropic API key (only when
       using the helper).
     - Run `npm run update-components`.
     - Review changes in `.logs/` and apply selectively.
     - **Post-completion:** Remove the key using T119 and verify using T129.
   - **Option B – Manual update**:
     - Update `package.json` dependencies to match 1.11.0 starter kit.
     - Copy new `build.js` and `build-config.json` from starter kit.
     - Copy `.storybook/sdc-plugin.js` and update `.storybook/preview.js`.

7. **Add SDC enforcement** (optional but recommended):
   - Add `enforce_prop_schemas: true` to `<subtheme>.info.yml`.

### 6.3 Validation phase

1. **Clear caches and run updates**:
   - `drush cr` (or equivalent).
   - `drush updb` if database updates are pending.
   - `drush cim` if configuration changes are pending.

2. **Check for SDC-related errors**:
   - Review Drupal watchdog/logs for:
     - `ComponentNotFoundException` errors.
     - Missing library errors for `civictheme:alert` or other components.
     - Twig rendering errors.

3. **Test key pages and components**:
   - Home page.
   - Navigation (header, footer, mobile menu).
   - Search results (verify relevance ordering fix).
   - Content pages (articles, landing pages).
   - Components: alerts, table-of-contents, cards, forms.
   - Automated lists (verify CSS class passing).

4. **Test Storybook** (if used):
   - Run `npm run storybook`.
   - Verify all components render correctly.
   - Check for build errors in console.

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
  - This file: `docs/civic-theme-upgrades/versions/v1.10.0-to-v1.11.0/spec.md`
  - The corresponding `tasks.md` and `playbook.md`.
- Use the upstream links in Section 2 to understand the intent of 1.11.0.
- Treat the customisation register as the source of truth for what must be
  preserved or adapted.
- Prefer:
  - Proposing changes as diffs that keep customisations aligned with
    upstream CivicTheme.
  - Avoiding destructive operations and any commands that act directly on
    production environments.
- Keep this spec and the customisation register up to date by:
  - Adding notes about which customisations were impacted or retired.
  - Recording any new customisations introduced as part of the upgrade.

---

## 8. References

This section provides links to official upstream resources for the CivicTheme `1.10.0 → 1.11.0` upgrade.

### Release information

- **CivicTheme 1.10.0 release notes** (source version):  
  https://www.drupal.org/project/civictheme/releases/1.10.0
- **CivicTheme 1.11.0 release notes** (target version):  
  https://www.drupal.org/project/civictheme/releases/1.11.0
- **CivicTheme documentation portal**:  
  https://docs.civictheme.io/changelog

### Code comparison

- **Git diff** (1.10.0 → 1.11.0):  
  https://git.drupalcode.org/project/civictheme/-/compare/1.10.0...1.11.0?from_project_id=86817

### Upgrade tools and resources

- **CivicTheme Upgrade Tools** (SDC update script and other helpers):  
  https://github.com/civictheme/upgrade-tools/blob/main/README.md
- **CivicTheme Drupal.org project page**:  
  https://www.drupal.org/project/civictheme
- **CivicTheme main website**:  
  https://civictheme.io

### Single Directory Components (SDC) resources

- **Drupal SDC documentation**:  
  https://www.drupal.org/docs/core-modules-and-themes/core-modules/sdc-module
- **CivicTheme SDC guidance**:  
  See relevant sections under https://docs.civictheme.io

### Security advisories

- **Security advisories index**:  
  https://www.drupal.org/security/contrib

---

## 9. Developer review checklist

Before considering the 1.10.0 → 1.11.0 upgrade complete, a developer
SHOULD confirm:

- [ ] The customisation register has been refreshed and all known
      CivicTheme-related customisations are listed with stable IDs.
- [ ] All customisations identified as affected by 1.11.0 have been
      reviewed and either updated, replaced or explicitly retired.
- [ ] Drupal core meets the `^10.2 || ^11` requirement and any core /
      module changes required for SDC are documented (even if implemented
      under a separate upgrade step).
- [ ] The project builds successfully and basic smoke tests (cache clear,
      config import, entity updates) complete without errors.
- [ ] Key pages and components that rely on CivicTheme (home, search,
      article/landing pages, navigation, alerts, table-of-contents,
      forms) have been manually reviewed.
- [ ] Any regressions or follow-up tasks are captured in
      `tasks.md` or in a project issue tracker.
- [ ] This spec, `tasks.md` and `playbook.md` accurately reflect what was
      performed, so that future upgrades can rely on them as history.
