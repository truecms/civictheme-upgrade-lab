# AI Agent Instructions: CivicTheme Upgrades

This file provides instructions for AI coding agents (Claude Code, Cursor, Copilot, etc.) working on CivicTheme upgrades in Drupal projects.

## Quick Start: Where to Begin

**Start here**: Read this file, then proceed to `spec.md` in the relevant version directory.

```text
docs/civic-theme-upgrades/
├── AGENTS.md              ← You are here (AI instructions)
├── README.md              ← Human-readable entry point
├── planning.md            ← Framework governance (reference only)
├── customisations.md      ← Project customisation register (CRITICAL)
└── versions/
    └── v<FROM>-to-v<TO>/
        ├── spec.md        ← START HERE for each upgrade step
        ├── tasks.md       ← Checklist derived from spec
        └── playbook.md    ← Ordered execution steps
```

---

## Framework Overview

### What Is This?

The CivicTheme Upgrade Assistant is a **documentation-first framework** for planning and executing CivicTheme version upgrades in Drupal projects. It provides structured guidance through:

- **Specifications** (`spec.md`): What needs to change and why
- **Task lists** (`tasks.md`): Concrete, tickable items
- **Playbooks** (`playbook.md`): Ordered execution steps

### Core Principles

1. **Sequential upgrades**: One CivicTheme release per upgrade step. Never skip versions.
2. **Exact version constraints**: Use `drupal/civictheme:1.12.0`, never `^1.12` or `~1.12`.
3. **Non-production environments**: All work in feature branches, dev, or staging.
4. **Preserve `.gitignore`**: Never modify existing `.gitignore` files.
5. **Customisation tracking**: Maintain stable IDs (C001, C002, etc.) in the register.

---

## Version Verification Commands

Run these commands **before** and **after** every upgrade step to confirm success.

### Before Starting

```bash
# 1. Current CivicTheme version
composer show drupal/civictheme | grep -E "^versions"

# 2. Drupal core version (1.11+ requires ^10.2 || ^11)
drush status --field=drupal-version

# 3. Locate sub-theme
ls -la web/themes/custom/

# 4. Check for pending database updates
drush updb --no
```

**Record these values** before proceeding. The upgrade is only successful if the target version matches after completion.

### After Completing

```bash
# 1. Verify exact target version installed
composer show drupal/civictheme | grep -E "^versions"

# 2. Clear caches and run updates
drush cr && drush updb && drush cim -y

# 3. Rebuild frontend assets
npm run build  # or: ahoy fe

# 4. Check for Twig errors
drush cr 2>&1 | grep -i "twig\|template"
```

**Success criteria**: The version output matches the exact target version (e.g., `1.12.0`), caches clear without errors, and the site renders correctly.

---

## Upgrade Sequence

Follow this sequence for every CivicTheme upgrade:

### Phase 1: Discovery

1. **Verify current state** using the commands above
2. **Read the customisation register** at `customisations.md`
3. **Locate the version-specific directory**: `versions/v<FROM>-to-v<TO>/`
4. **Read `spec.md`** to understand upstream changes and risks

### Phase 2: Planning

1. **Review `tasks.md`** for the complete checklist
2. **Cross-reference customisations** with upstream breaking changes
3. **Identify HIGH-risk items** requiring manual developer review
4. **Ensure prerequisites** are met (e.g., Drupal core version)

### Phase 3: Execution

1. **Create a feature branch** (never work on main/master)
2. **Follow `playbook.md`** step-by-step
3. **Pause at stop conditions** and request developer input
4. **Run verification commands** after each major step

### Phase 4: Validation

1. **Run full verification commands** (see above)
2. **Test critical paths**: home page, navigation, search, forms
3. **Test customised components** referenced in the register
4. **Update the customisation register** if any items changed

---

## File Responsibilities

| File | Purpose | When to Read | When to Update |
|------|---------|--------------|----------------|
| `customisations.md` | Project-specific overrides register | Every upgrade | After discovery, after changes |
| `spec.md` | What & why for this version step | Before planning | Rarely (framework maintainers only) |
| `tasks.md` | Checklist of work items | During planning | Mark items complete during work |
| `playbook.md` | How to execute the upgrade | During execution | Add lessons learned after completion |

---

## Stop Conditions

**Halt and request developer input** when:

| Condition | Reason | Action |
|-----------|--------|--------|
| Version mismatch | Current version doesn't match expected "from" version | Verify correct upgrade path |
| Drupal core < 10.2 | CivicTheme 1.11+ requires ^10.2 \|\| ^11 | Upgrade Drupal core first |
| `{% extends %}` patterns found | Not supported in SDC (1.11+) | Requires refactoring decision |
| Build failures | `npm run build` exits non-zero | Debug before proceeding |
| Twig rendering errors | Templates fail after cache clear | Investigate syntax issues |
| Missing customisation register | `customisations.md` is template-only | Populate register first |

---

## Critical Breaking Changes Reference

### CivicTheme 1.11.0+ (SDC Migration)

**Twig include syntax**:
```twig
{# OLD (pre-1.11) #}
{% include '@atoms/paragraph/paragraph.twig' %}
{% include '@molecules/logo/logo.twig' %}

{# NEW (1.11+) #}
{% include 'civictheme:paragraph' %}
{% include 'civictheme:logo' %}
```

**Block naming**:
```twig
{# OLD #}
{% block content_slot %}

{# NEW #}
{% block content_block %}
```

**Template extension** (`{% extends %}` + `{{ parent() }}`):
- **Not supported** for CivicTheme components in 1.11+
- Options: Override completely (copy full template) or remove override

---

## Customisation Register Format

The register at `customisations.md` MUST use this format:

```markdown
- [ ] C001 [HIGH] Custom header override - templates/civictheme-header.html.twig
- [ ] C002 [MEDIUM] Event listing styles - scss/components/_event-card.scss
- [ ] C003 [LOW] Footer logo swap - templates/civictheme-footer.html.twig
```

**Impact levels**:
- **HIGH**: Extended templates, structural changes (likely broken by upgrades)
- **MEDIUM**: Style overrides, custom components (may need adjustment)
- **LOW**: Minor tweaks, configuration (usually safe)

---

## AI Agent Behaviour Guidelines

### DO

- Read `spec.md` thoroughly before proposing changes
- Verify versions before and after every operation
- Reference customisation IDs (C001, C002) in discussions
- Pause at stop conditions and explain the situation
- Keep the customisation register current
- Use exact version constraints in Composer commands

### DO NOT

- Skip versions (e.g., jump from 1.10.0 directly to 1.12.0)
- Modify `.gitignore` files
- Execute playbook steps without confirming the environment is non-production
- Assume customisations are current—always verify against the codebase
- Use caret (`^`) or tilde (`~`) version constraints
- Proceed past stop conditions without developer confirmation

---

## External Resources

- CivicTheme documentation: https://docs.civictheme.io
- CivicTheme releases: https://www.drupal.org/project/civictheme/releases
- Upgrade tools: https://github.com/civictheme/upgrade-tools

---

## Next Step

**If you're starting an upgrade now**:

1. Run the "Before Starting" verification commands above
2. Navigate to `versions/v<FROM>-to-v<TO>/spec.md` for your target version
3. Follow the upgrade sequence outlined in this document

If the version directory doesn't exist, create it following the pattern in `planning.md` and use an existing version directory as a template.
