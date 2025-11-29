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
- Standard organisational safeguards apply:
  - Backups and/or database snapshots exist prior to running the upgrade.
  - Deployments are reviewed via merge requests or equivalent.
- This spec focuses on the **CivicTheme 1.10.0 → 1.11.0** step only. Any
  earlier or later version jumps MUST be documented in separate directories.

Out of scope:

- Production deployment steps.
- Non-CivicTheme-related module or core Drupal upgrades (these MAY be
  referenced if tightly coupled but are not the primary focus).

---

## 2. Upstream references for 1.11.0

Authors and AI assistants MUST consult the following upstream sources when
preparing and validating this upgrade:

- **Release notes**: CivicTheme `1.11.0` on drupal.org  
  `https://www.drupal.org/project/civictheme/releases/1.11.0`
- **CivicTheme documentation portal**:  
  `https://docs.civictheme.io/`
- **Upstream code diff** (1.10.0 → 1.11.0):  
  `https://git.drupalcode.org/project/civictheme/-/compare/1.10.0...1.11.0?from_project_id=86817`

When maintaining this spec in a real project, copy or summarise the
relevant sections from these sources to keep this document authoritative
for the `1.10.0 → 1.11.0` upgrade.

---

## 3. High-level upstream changes (1.10.0 → 1.11.0)

From the release notes and diff, CivicTheme 1.11.0 introduces, among other
changes:

- **Component architecture**:
  - Migration of components to Single Directory Components (SDC) and
    support for SDC in Drupal 10.3+.
  - Adjustments to component structure and template discovery.
- **Design system and components**:
  - Updates to certain Twig templates and component behaviours.
  - Potential new or adjusted configuration options for components.
- **Theming and assets**:
  - Possible changes to SCSS, JavaScript and asset loading patterns to
    align with SDC and updated components.
- **Miscellaneous fixes and improvements**:
  - Bug fixes, accessibility tweaks and documentation updates.

For a real project, refine this section with concrete notes pulled from
the release notes and diff, including any *breaking* or *deprecated*
patterns relevant to your current usage.

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

- Components that are being converted to SDC.
- Twig templates that override upstream CivicTheme templates.
- SCSS/JS that depends on specific component markup or classes.
- Configuration overrides (e.g. view modes, blocks, menus) tied to
  CivicTheme components.

Each relevant customisation SHOULD be cross-referenced by ID (e.g. `C5`,
`C7`) in the tasks and playbook.

---

## 5. Risk assessment and focus areas

Based on upstream changes and the customisation inventory, typical risk
areas for this upgrade include:

- **Template compatibility**:
  - Overridden Twig templates may no longer match upstream structure after
    SDC changes.
  - Custom SDC components may require directory structure or annotation
    updates.
- **SCSS/JS compatibility**:
  - Changes in markup or CSS class names could break custom styling or
    scripts.
- **Configuration dependencies**:
  - View modes, blocks or layouts that rely on specific CivicTheme
    components may need review.
- **Regression risk on key pages**:
  - Landing pages, search, news or other high-traffic patterns that make
    heavy use of CivicTheme components.

Projects SHOULD explicitly list their own high-risk areas here (for
example, “news landing pages using custom teaser card overrides
`C004`/`C005`”).

---

## 6. Upgrade strategy overview

The intended strategy for this upgrade is:

1. **Discovery**
   - Confirm current CivicTheme version is `1.10.0`.
   - Refresh the customisation register and identify entries impacted by
     SDC and component changes.
   - Review the 1.11.0 release notes and diff, annotating which upstream
     changes touch local customisations.
2. **Change**
   - Apply the composer update for CivicTheme (and any tightly coupled
     dependencies) to move from `1.10.0` to `1.11.0`.
   - Reconcile overridden Twig templates, SCSS and JS with upstream
     changes, updating or retiring overrides as needed.
   - Adjust configuration where component APIs or behaviour changed.
3. **Validation**
   - Run through targeted checks on key pages and components.
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
- [ ] The project builds successfully and basic smoke tests (cache clear,
      config import, entity updates) complete without errors.
- [ ] Key pages and components that rely on CivicTheme (home, search,
      article/landing pages, navigation) have been manually reviewed.
- [ ] Any regressions or follow-up tasks are captured in
      `tasks.md` or in a project issue tracker.
- [ ] This spec, `tasks.md` and `playbook.md` accurately reflect what was
      performed, so that future upgrades can rely on them as history.
