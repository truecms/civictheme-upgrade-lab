# CivicTheme upgrade spec: v1.10.0 → v1.11.0

**Project**: Destination Drupal project using CivicTheme  
**CivicTheme source version**: `1.10.0`  
**CivicTheme target version**: `1.11.0`  
**Per-version directory**: `docs/civic-theme-upgrades/versions/v1.10.0-to-v1.11.0/`

This document defines the canonical per-version specification for the
CivicTheme `1.10.0 → 1.11.0` upgrade, using the framework defined in
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
    - Requires Node.js 22+, a CivicTheme sub-theme already updated to
      CivicTheme 1.10.x, and an Anthropic API key.
    - Run in a throwaway working copy or feature branch only (it
      updates `package.json`, Storybook config, `build.js` and
      component directories).
    - Typical flow: clone the repo, run `npm install`, configure a
      `.env` file (sub-theme path, Anthropic API key, optional model)
      and execute `npm run update-components`, then review changes and
      `.logs/` output.
- **Upstream code diff** (1.10.0 → 1.11.0):  
  `https://git.drupalcode.org/project/civictheme/-/compare/1.10.0...1.11.0?from_project_id=86817`

When maintaining this spec in a real project, copy or summarise the
relevant sections from these sources to keep this document authoritative
for the `1.10.0 → 1.11.0` upgrade.

---

## 3. High-level upstream changes (1.10.0 → 1.11.0)

From the release notes and diff, CivicTheme 1.11.0 introduces, among other
changes:

- **Component architecture & SDC adoption**:
  - Core CivicTheme components are migrated towards **Single Directory
    Components (SDC)** to align with Drupal 10.2+/10.3+.
  - New `.component.yml` definitions and per-component `.css` files are
    added across the component tree.
  - A new `civictheme_table_of_contents` theme hook and corresponding
    component are introduced.
- **Theming and build assets**:
  - Global CSS and JS outputs are restructured:
    - `dist/civictheme.css` is split into
      `dist/civictheme.base.css`, `dist/civictheme.theme.css`
      and `dist/civictheme.variables.css`.
    - `dist/civictheme.js` is replaced by
      `dist/civictheme.drupal.base.js` (plus a Storybook base bundle).
  - The `civictheme.libraries.yml` `global` library is updated to attach
    the new CSS/JS files; linting/build tooling (`build.js`,
    `build-config.json`) is extended for SDC and new bundles.
- **Theme bootstrap & behaviour**:
  - `civictheme.theme` now integrates with Drupal’s SDC plugin manager
    to attach the `civictheme:alert` component library for authenticated
    users, with appropriate error handling.
  - API examples for `hook_civictheme_automated_list_view_info_alter()`
    switch from a generic `'event'` content type to the canonical
    `civictheme_event` machine name.
- **Starter kit and Storybook**:
  - The starter kit’s `build.js`, Storybook config and asset pipeline are
    updated to match the new SDC-aware build.

For a real project, refine this section with concrete notes pulled from
the release notes and diff, including any *breaking* or *deprecated*
patterns relevant to your current usage (for example, references to
`dist/civictheme.css` in sub-themes or custom build scripts).

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

Based on upstream changes and the customisation inventory, typical risk
areas for this upgrade include:

- **Template & SDC compatibility**:
  - Overridden Twig templates may no longer match upstream structure
    after SDC-related refactors.
  - Custom components that start using `.component.yml` definitions may
    require directory structure, naming or annotation updates to remain
    discoverable by Drupal’s SDC system.
- **SCSS/JS and asset pipeline compatibility**:
  - Changes in markup or CSS class names could break custom styling or
    scripts.
  - Sub-themes or build scripts that reference `dist/civictheme.css` or
    `dist/civictheme.js` directly must be updated to the new
    split bundles and file names.
- **Configuration dependencies**:
  - View modes, blocks or Layout Builder sections that rely on CivicTheme
    components (including alerts, table-of-contents and form/field
    components) may need review.
- **Drupal core and module compatibility**:
  - Sites still on Drupal `<10.2` cannot adopt CivicTheme 1.11.0 without
    a core upgrade; SDC features may depend on core 10.2+/10.3+ and the
    appropriate core modules being enabled.
- **Regression risk on key pages**:
  - Landing pages, search, news or other high-traffic patterns that make
    heavy use of CivicTheme components (especially lists, cards, forms
    and navigation) carry higher regression risk.

Projects SHOULD explicitly list their own high-risk areas here (for
example, “news landing pages using custom teaser card overrides
`C004`/`C005`”).

---

## 6. Upgrade strategy overview

The intended strategy for this upgrade is:

1. **Discovery**
   - Confirm current CivicTheme version is `1.10.0`.
   - Confirm Drupal core is on a version compatible with
     `core_version_requirement: ^10.2 || ^11`; if not, record the
     dependency and plan a separate core upgrade.
   - Refresh the customisation register and identify entries impacted by
     SDC and component changes, including any custom build tooling that
     references old asset names.
   - Review the 1.11.0 release notes, CivicTheme docs and git diff,
     annotating which upstream changes touch local customisations.
2. **Change**
   - Apply the Composer update for CivicTheme (and any tightly coupled
     dependencies) to move from `1.10.0` to `1.11.0`, ensuring both
     `composer.json` and `composer.lock` are updated together.
   - Reconcile overridden Twig templates, SCSS and JS with upstream
     changes, updating or retiring overrides as needed.
   - Update sub-theme libraries and build scripts where they reference
     old global asset file names (`dist/civictheme.css`,
     `dist/civictheme.js`) so they align with the new split bundles.
   - Evaluate and, where appropriate, run the CivicTheme SDC Update Tool
     against custom components in the sub-theme:
     - Treat this as an **optional, high-impact helper** – always run
       it in a feature branch or isolated working copy.
     - Ensure the sub-theme meets the tool’s prerequisites (CivicTheme
       1.10.x, Node.js 22+, Anthropic API key configured via `.env`).
     - Use the tool to generate or update SDC schemas and
       `.component.yml`/component `.css` files, then manually review the
       diff, logs and resulting site/Storybook output before commit.
   - Adjust configuration where component APIs or behaviour changed (for
     example automated lists, alerts, table-of-contents, navigation).
3. **Validation**
   - Run through targeted checks on key pages and components, including
     SDC-backed components and forms.
   - Confirm Storybook/design-system builds (where used) still succeed
     and reflect the updated components.
   - Record observations, regressions and follow-ups in the per-version
     docs and (where applicable) the customisation register.

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

## 8. Human review checklist

Before considering the 1.10.0 → 1.11.0 upgrade complete, a human reviewer
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
