---
name: civictheme-upgrade
description: Plan and execute CivicTheme upgrades in Drupal projects. Use when working with Drupal sites using CivicTheme that need version upgrades (e.g., 1.10→1.11, 1.11→1.12). Handles SDC migration, Twig syntax updates, build tooling changes, and customisation preservation. Triggers include "upgrade civictheme", "civictheme migration", "update civictheme version", or any CivicTheme-related Drupal theme upgrade work.
---

# CivicTheme Upgrade Skill

Assists with planning and executing CivicTheme upgrades in Drupal projects using a documentation-first, sequential upgrade approach.

## Core Principles

1. **Sequential upgrades**: One CivicTheme release per upgrade step. Never skip versions.
2. **Exact version constraints**: Use `drupal/civictheme:1.12.0` not `^1.12`.
3. **Non-production first**: All work in feature branches, dev, or staging environments.
4. **Preserve `.gitignore`**: Never modify existing `.gitignore` files.
5. **Customisation register**: Track all theme customisations with stable IDs (C001, C002, etc.).

## Workflow Overview

```
1. Discovery → 2. Planning → 3. Changes → 4. Validation
```

### Step 1: Discovery

Identify current state and customisations:

```bash
# Current CivicTheme version
composer show drupal/civictheme | grep versions

# Drupal core version (1.11+ requires ^10.2 || ^11)
drush status --field=drupal-version

# Find sub-theme location
ls -la web/themes/custom/

# Audit Twig templates for breaking patterns
grep -rn "include '@atoms/" <subtheme>/templates/
grep -rn "{% extends " <subtheme>/templates/
grep -rn "_slot %}" <subtheme>/templates/
```

### Step 2: Planning

Read version-specific documentation:

```
references/versions/v<FROM>-to-v<TO>/
├── spec.md      # What & why (upstream changes, risks)
├── tasks.md     # Checklist (tickable items)
└── playbook.md  # How (ordered runbook)
```

Cross-reference with `references/customisations.md` to identify impacted customisations.

### Step 3: Changes

Apply upgrade in order:

1. **Composer update**: `composer require drupal/civictheme:<VERSION>`
2. **Twig syntax**: Update include patterns
3. **Block names**: Rename `_slot` → `_block`
4. **Library overrides**: Update file references in `<subtheme>.info.yml`
5. **Build tooling**: Update `package.json`, `build.js`, Storybook config

### Step 4: Validation

```bash
drush cr && drush updb && drush cim
composer show drupal/civictheme | grep versions  # Verify exact version
npm run build  # or ahoy fe
```

Test: home page, navigation, search, forms, custom components.

## Version-Specific References

| Upgrade Path | Key Changes | Reference |
|--------------|-------------|-----------|
| 1.10.0 → 1.11.0 | SDC migration, Twig namespace changes | `references/versions/v1.10.0-to-v1.11.0/` |
| 1.11.0 → 1.12.0 | Security fixes, SDC refinements | `references/versions/v1.11.0-to-v1.12.0/` |
| 1.12.0 → 1.12.1 | Patch release | `references/versions/v1.12.0-to-v1.12.1/` |
| 1.12.1 → 1.12.2 | Patch release | `references/versions/v1.12.1-to-v1.12.2/` |

**Before starting any upgrade**, read the relevant `spec.md` and `tasks.md` for that version step.

## Critical Breaking Changes (1.11.0+)

### Twig Include Syntax

```twig
{# OLD (pre-1.11) #}
{% include '@atoms/paragraph/paragraph.twig' %}
{% include '@molecules/logo/logo.twig' %}

{# NEW (1.11+) #}
{% include 'civictheme:paragraph' %}
{% include 'civictheme:logo' %}
```

### Block Naming

```twig
{# OLD #}
{% block content_slot %}

{# NEW #}
{% block content_block %}
```

### Template Extension

**Not supported in 1.11+**: `{% extends %}` and `{{ parent() }}` for CivicTheme components.

Options:
- **Override completely**: Copy full upstream template, apply customisations inline
- **Remove override**: Use upstream component unchanged

### Library Overrides (`<subtheme>.info.yml`)

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

## Customisation Register

Maintain at `docs/civic-theme-upgrades/customisations.md` with:

```markdown
- [ ] C001 [HIGH] Custom header override - templates/civictheme-header.html.twig
- [ ] C002 [MEDIUM] Event listing styles - scss/components/_event-card.scss  
- [ ] C003 [LOW] Footer logo swap - templates/civictheme-footer.html.twig
```

Impact levels:
- **HIGH**: Extended templates, structural changes
- **MEDIUM**: Style overrides, custom components
- **LOW**: Minor tweaks, configuration

## Stop Conditions

Halt and seek developer input when:

1. CivicTheme version doesn't match expected "from" version
2. Drupal core < 10.2 (for 1.11+ upgrades)
3. `{% extends %}` patterns found (requires refactoring decision)
4. Build failures after tooling updates
5. Test regressions detected

## Additional References

- `references/planning.md` - Global framework and governance
- `references/customisations.md` - Customisation register template
- `references/versions/*/spec.md` - Version-specific specifications
- `references/versions/*/tasks.md` - Version-specific task checklists
- `references/versions/*/playbook.md` - Version-specific runbooks

## External Links

- CivicTheme docs: https://docs.civictheme.io
- CivicTheme releases: https://www.drupal.org/project/civictheme/releases
- Upgrade tools: https://github.com/civictheme/upgrade-tools
