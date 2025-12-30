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

## Pre-flight: Baseline Normalisation

Before starting any upgrade, ensure the project has **exact version constraints** and **no modifications to the upstream parent theme**. This step prevents unexpected version changes during upgrades and captures any direct edits to `web/themes/contrib/civictheme`.

### Step 1: Extract versions from Composer files

```bash
# Get the INSTALLED version from composer.lock (authoritative)
composer show drupal/civictheme --locked | grep -E "^versions"

# Get the DECLARED constraint from composer.json
grep -A2 '"drupal/civictheme"' composer.json
```

**Record both values.** The `composer.lock` version is the actual installed version; the `composer.json` constraint determines what Composer will allow during updates.

### Step 2: Classify the constraint

| Constraint Type                | Examples                                     | Action Required         |
|--------------------------------|----------------------------------------------|-------------------------|
| **Exact** (allowed)            | `1.12.0`, `1.12.0-rc1`                       | No normalisation needed |
| **Non-exact** (must normalise) | `^1.12`, `~1.12`, `>=1.12`, `*`, `dev-main`  | Normalise after Step 3  |

### Step 3: Check for parent-theme modifications

**CRITICAL**: Before normalising `composer.json`, check whether the upstream CivicTheme parent theme has been directly modified. Upgrades will **overwrite** any such changes.

Since `web/themes/contrib/` is typically git-ignored (managed by Composer), use the pristine comparison method:

```bash
# 1. Get the exact installed version from composer.lock
CIVICTHEME_VERSION=$(composer show drupal/civictheme --locked --format=json | grep -o '"version": "[^"]*"' | head -1 | cut -d'"' -f4)
echo "Installed version: $CIVICTHEME_VERSION"

# 2. Copy the currently installed theme to a temp location
mkdir -p /tmp/civictheme-compare
cp -r web/themes/contrib/civictheme /tmp/civictheme-compare/installed

# 3. Create a pristine copy using Composer in an isolated directory
mkdir -p /tmp/civictheme-compare/pristine-project
cd /tmp/civictheme-compare/pristine-project
composer init --no-interaction --name="temp/civictheme-check"
composer require drupal/civictheme:$CIVICTHEME_VERSION --no-interaction
cp -r vendor/drupal/civictheme /tmp/civictheme-compare/pristine

# 4. Return to project root
cd -

# 5. Compare using checksums (excluding node_modules, dist, and other generated files)
echo "=== Comparing installed vs pristine (excluding generated files) ==="
diff -rq \
  --exclude='node_modules' \
  --exclude='dist' \
  --exclude='.npm' \
  --exclude='package-lock.json' \
  --exclude='storybook-static' \
  /tmp/civictheme-compare/pristine \
  /tmp/civictheme-compare/installed

# 6. Alternative: Generate MD5 checksums for key files
echo "=== MD5 checksum comparison for source files ==="
cd /tmp/civictheme-compare
find pristine -type f \( -name "*.twig" -o -name "*.php" -o -name "*.yml" -o -name "*.js" -o -name "*.scss" \) \
  ! -path "*/node_modules/*" ! -path "*/dist/*" -exec md5sum {} \; | sort > pristine-checksums.txt
find installed -type f \( -name "*.twig" -o -name "*.php" -o -name "*.yml" -o -name "*.js" -o -name "*.scss" \) \
  ! -path "*/node_modules/*" ! -path "*/dist/*" -exec md5sum {} \; | sort > installed-checksums.txt
# Compare (adjust paths for comparison)
sed 's|pristine/|theme/|g' pristine-checksums.txt > pristine-normalised.txt
sed 's|installed/|theme/|g' installed-checksums.txt > installed-normalised.txt
diff pristine-normalised.txt installed-normalised.txt
cd -
```

**If the diff/checksum shows differences**: The parent theme has been modified locally.

**If no differences**: The parent theme is unmodified and safe to proceed.

### Step 4: Handle findings

**If parent-theme modifications are found**:

1. **Copy the modified theme** for safekeeping:

   ```bash
   cp -r web/themes/contrib/civictheme /tmp/civictheme-backup-$(date +%Y%m%d)
   ```

2. **Record each modification** in `docs/civic-theme-upgrades/customisations.md` as a **HIGH** risk entry:

   ```markdown
   - [ ] C0XX [HIGH] Parent theme modification - web/themes/contrib/civictheme/<path>
         Locked version: 1.11.0
         Change: <brief description>
         WARNING: Will be overwritten on upgrade
   ```

3. **STOP** and request developer decision. Do not proceed with the upgrade until the developer decides how to handle these modifications.

**If NO parent-theme modifications are found** and `composer.json` is non-exact:

1. **Normalise `composer.json`** to pin the exact installed version:

   ```bash
   composer require drupal/civictheme:<VERSION_FROM_LOCK> --no-update
   ```

2. **Verify no version change** occurred:

   ```bash
   composer show drupal/civictheme --locked | grep -E "^versions"
   # Must match the original installed version
   ```

---

## Upgrade Sequence

Follow this sequence for every CivicTheme upgrade:

### Phase 0: Pre-flight (One-Time per Project)

1. **Run baseline normalisation** (see "Pre-flight: Baseline Normalisation" above)
2. **Ensure `composer.json` pins an exact version** matching `composer.lock`
3. **Confirm no parent-theme modifications** (or record them and stop)

### Phase 1: Discovery

1. **Verify current state** using the version verification commands above
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

| File                | Purpose                              | When to Read     | When to Update                       |
|---------------------|--------------------------------------|------------------|--------------------------------------|
| `customisations.md` | Project-specific overrides register  | Every upgrade    | After discovery, after changes       |
| `spec.md`           | What & why for this version step     | Before planning  | Rarely (framework maintainers only)  |
| `tasks.md`          | Checklist of work items              | During planning  | Mark items complete during work      |
| `playbook.md`       | How to execute the upgrade           | During execution | Add lessons learned after completion |

---

## Stop Conditions

**Halt and request developer input** when:

| Condition                      | Reason                                                 | Action                                             |
|--------------------------------|--------------------------------------------------------|----------------------------------------------------|
| Version mismatch               | Current version doesn't match expected "from" version  | Verify correct upgrade path                        |
| Drupal core < 10.2             | CivicTheme 1.11+ requires ^10.2 \|\| ^11               | Upgrade Drupal core first                          |
| `{% extends %}` patterns found | Not supported in SDC (1.11+)                           | Requires refactoring decision                      |
| Build failures                 | `npm run build` exits non-zero                         | Debug before proceeding                            |
| Twig rendering errors          | Templates fail after cache clear                       | Investigate syntax issues                          |
| Missing customisation register | `customisations.md` is template-only                   | Populate register first                            |
| Parent theme modified          | `web/themes/contrib/civictheme` differs from pristine  | Record in register + stop for developer decision   |
| Dev/non-release version        | `composer.lock` shows `dev-*` or non-semver version    | Cannot map to version docs; clarify target version |

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
- Use `composer.lock` as the authoritative source for currently installed versions
- Normalise non-exact `composer.json` constraints before starting upgrades
- Check for parent-theme modifications before any upgrade step
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
- Proceed if parent-theme modifications are detected without developer approval
- Trust `composer.json` constraints alone—always verify against `composer.lock`

---

## External Resources

- CivicTheme documentation: <https://docs.civictheme.io>
- CivicTheme releases: <https://www.drupal.org/project/civictheme/releases>
- Upgrade tools: <https://github.com/civictheme/upgrade-tools>

---

## Next Step

**If you're starting an upgrade now**:

1. Run the "Before Starting" verification commands above
2. Navigate to `versions/v<FROM>-to-v<TO>/spec.md` for your target version
3. Follow the upgrade sequence outlined in this document

If the version directory doesn't exist, create it following the pattern in `planning.md` and use an existing version directory as a template.
