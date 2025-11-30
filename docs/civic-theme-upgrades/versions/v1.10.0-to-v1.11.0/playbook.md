# CivicTheme upgrade playbook: v1.10.0 → v1.11.0

**Directory**: `docs/civic-theme-upgrades/versions/v1.10.0-to-v1.11.0/`  
**Related documents**: `spec.md`, `tasks.md`, `docs/civic-theme-upgrades/customisations.md`

This playbook turns the tasks in `tasks.md` into an ordered runbook for a
non-production upgrade of CivicTheme from `1.10.0` to `1.11.0`.

---

## 1. Environment checks

**Related tasks**: T100, T120

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
git checkout -b feature/civictheme-1.11-upgrade
```

### 1.6 Verify Drupal core version

```bash
# Check Drupal core version (use ahoy/docker as needed)
drush status --field=drupal-version

# Or via composer
composer show drupal/core | grep versions
```

**STOP CONDITION**: If Drupal core is `<10.2`, this upgrade is blocked.
CivicTheme 1.11.0 requires `core_version_requirement: ^10.2 || ^11`.
Plan a separate Drupal core upgrade first.

### 1.7 Verify current CivicTheme version

```bash
composer show drupal/civictheme | grep versions
# Expected: 1.10.0
```

### 1.8 Discover available front-end commands

Before making changes, identify how front-end tooling (npm) is run in this
project. In Docker environments, npm commands typically run inside a
container.

```bash
# Check for ahoy front-end commands
if [ -f ".ahoy.yml" ]; then
  echo "=== Ahoy front-end commands ==="
  grep -E "^\s*(fe|front|npm|build|storybook)" .ahoy.yml | head -20
  
  # Look for 'fe' command specifically and check what it does
  echo "=== Checking 'fe' command details ==="
  grep -A 10 "^\s*fe:" .ahoy.yml
fi

# Common front-end command patterns to look for:
# - ahoy fe               (Front-end build - equivalent to npm run build from theme directory)
# - ahoy build            (Direct build command, if available)
# - ahoy storybook        (Start Storybook, if available)
```

**Document discovered front-end commands**:

| Command | Description | Usage |
|---------|-------------|-------|
| `ahoy fe` | Front-end build (equivalent to `npm run build` from theme directory) | Can be invoked from project root |
| `ahoy build` | Direct build | If available |
| `ahoy storybook` | Storybook | If available |

**Key insight**: `ahoy fe` is equivalent to running `npm run build` from the theme directory, but can be invoked from the project root. Use this as the primary command for rebuilding front-end assets.

### 1.9 Discover available test commands

Identify what testing capabilities exist in the project. This establishes
the baseline for regression testing.

```bash
# Check for ahoy test commands
if [ -f ".ahoy.yml" ]; then
  echo "=== Ahoy test commands ==="
  grep -E "^\s*test" .ahoy.yml || grep -i "test" .ahoy.yml | head -20
fi

# Check for composer test scripts
echo "=== Composer test scripts ==="
grep -A 30 '"scripts"' composer.json | grep -i "test" || echo "No test scripts found"

# Common test command patterns to look for:
# - ahoy test-bdd         (Behat/BDD tests)
# - ahoy test-unit        (PHPUnit tests)
# - ahoy test-playwright  (E2E browser tests)
# - ahoy test-kernel      (Kernel tests)
# - ahoy lint             (Code linting)
# - composer test
# - composer test-behat
# - composer test-phpunit
```

**Document discovered test commands** for use in validation:

| Command | Description | Status |
|---------|-------------|--------|
| `ahoy test-???` | [Describe] | Available/Not available |
| `composer test-???` | [Describe] | Available/Not available |

### 1.10 Run tests BEFORE upgrade (baseline)

**Critical**: Establish that tests pass before making any changes. If tests
fail before the upgrade, fix them first or document known failures.

```bash
# Run all available tests (adjust commands based on 1.8 discovery)
# Example using ahoy:
ahoy test-unit
ahoy test-bdd
ahoy test-playwright

# Example using composer:
composer test

# Example using docker compose directly:
docker compose exec cli ./vendor/bin/phpunit
docker compose exec cli ./vendor/bin/behat
```

Record test results:

- [ ] Unit tests: PASS / FAIL / N/A
- [ ] BDD tests: PASS / FAIL / N/A  
- [ ] E2E tests: PASS / FAIL / N/A
- [ ] Other tests: PASS / FAIL / N/A

**STOP CONDITION**: If critical tests fail before the upgrade, resolve those
issues first. Do not proceed with upgrade on a broken test baseline.

### 1.11 Review upgrade documentation

- [ ] Read `spec.md` Section 3 (High-level upstream changes).
- [ ] Read `spec.md` Section 5 (Risk assessment).
- [ ] Understand the SDC migration implications.

---

## 2. Discovery

**Related tasks**: T100, T101, T102, T103, T104

### 2.1 Refresh customisation register (T101)

Open `docs/civic-theme-upgrades/customisations.md` and ensure all
CivicTheme-related customisations are documented with stable IDs.

### 2.2 Audit sub-theme templates for breaking patterns (T102)

Run these commands in the sub-theme directory to identify files requiring
changes:

```bash
SUBTHEME_PATH=/path/to/your/subtheme

# T102a: Find old include patterns (need SDC syntax update)
echo "=== Old @atoms includes ==="
grep -rn "include '@atoms/" $SUBTHEME_PATH/templates/
echo "=== Old @molecules includes ==="
grep -rn "include '@molecules/" $SUBTHEME_PATH/templates/
echo "=== Old @organisms includes ==="
grep -rn "include '@organisms/" $SUBTHEME_PATH/templates/
echo "=== Old @base includes ==="
grep -rn "include '@base/" $SUBTHEME_PATH/templates/

# T102b: Find extended templates (HIGH RISK - not supported in SDC)
echo "=== Extended templates (CRITICAL) ==="
grep -rn "{% extends " $SUBTHEME_PATH/templates/
grep -rn "{{ parent() }}" $SUBTHEME_PATH/templates/

# T102c: Find old block names
echo "=== Old block names (_slot suffix) ==="
grep -rn "_slot %}" $SUBTHEME_PATH/templates/
```

**Record the output**. Each file listed requires modification.

### 2.3 Audit library overrides and build tooling (T103)

```bash
# T103a: Check info.yml for old library overrides
echo "=== Library overrides in info.yml ==="
grep -A 20 "libraries-override:" $SUBTHEME_PATH/*.info.yml

# T103b: Check libraries.yml for old file references
echo "=== Libraries.yml references ==="
grep -E "civictheme\.(css|js)" $SUBTHEME_PATH/*.libraries.yml

# T103c: Check for build.js existence
if [ -f "$SUBTHEME_PATH/build.js" ]; then
  echo "build.js exists - will need update"
fi

# T103d: Check package.json for SDC dependency
echo "=== Package.json dependencies ==="
grep "@civictheme" $SUBTHEME_PATH/package.json 2>/dev/null || echo "No @civictheme deps found"

# T103e: Capture custom library attachments (record in planning.md)
echo "=== Custom attach_library / #attached usage ==="
grep -rn "attach_library" $SUBTHEME_PATH/templates/ $SUBTHEME_PATH | head -100
grep -rn "#attached" $SUBTHEME_PATH | head -100
echo "=== Custom libraries defined ==="
grep -E "^[A-Za-z0-9_.-]+:" $SUBTHEME_PATH/*.libraries.yml | head -50

# Record the library name, file path, and attach location in
# docs/civic-theme-upgrades/versions/v1.10.0-to-v1.11.0/planning.md so they
# can be restored after the upgrade.
```

### 2.4 Use CivicTheme upgrade-tools (optional helper) (T116a)

The CivicTheme upgrade-tools repository includes `storybook-v8-update`/`sdc-update`
helpers that can refresh build tooling (Vite/Storybook v8) and convert
Storybook stories to the controls API. These steps are intentionally generic so
they can be reused across projects.

**Prerequisites**

- Node.js ≥ 22
- **Anthropic API key** (required only if you choose to use this helper;
  see STOP CONDITION below)
- Clean working copy or feature branch (the tool rewrites `package.json`,
  `.storybook`, `build.js`, component stories)

---

#### ⛔ STOP CONDITION – Anthropic API Key Required

**AI assistants MUST stop here and request developer action** before proceeding
with the upgrade-tools. The Storybook story conversion script requires an
`ANTHROPIC_API_KEY` to call the Anthropic Claude API.

These upgrade-tools are **optional**. If the developer does not wish to
configure an Anthropic API key, skip Section 2.4 entirely and rely on the
manual build tooling updates in Section 3.8 (Option B). The upgrade can
proceed without the key.

**Action required from developer:**

1. **Preferred method** – Add the key to the destination project's `.env` file
   (create the file if it does not exist):

   ```bash
   # In the destination project root directory
   echo 'ANTHROPIC_API_KEY=sk-ant-...' >> .env
   ```

2. **Alternative method** – Export the key in your shell session or shell
   configuration file (`~/.bashrc`, `~/.zshrc`):

   ```bash
   export ANTHROPIC_API_KEY=sk-ant-...
   ```

   If added to a shell configuration file, run `source ~/.bashrc` (or
   `source ~/.zshrc`) to load the new value.

**⚠️ Security reminder:** The Anthropic API key is a secret credential.
- Do NOT commit it to version control.
- Remove or unset the key after the upgrade is complete.

---

**Developer confirmation required:**

> Please confirm how the `ANTHROPIC_API_KEY` has been added:
>
> - [ ] Added to `.env` file in the destination project (preferred)
> - [ ] Exported in shell session / shell configuration file (alternative)
>
> Once confirmed, the AI assistant may proceed with the upgrade-tools
> execution. If neither option is selected (no key configured), do **not**
> run the upgrade-tools; continue with the manual build tooling steps instead.

---

**Execution (non-interactive recommended)**

```bash
# 1) Clone tools (or reuse an existing checkout)
git clone https://github.com/civictheme/upgrade-tools.git /tmp/civictheme-upgrade-tools
cd /tmp/civictheme-upgrade-tools/storybook-v8-update

# 2) Install dependencies
npm install

# 3) Update build + Storybook configs for your sub-theme
SUBTHEME_DIRECTORY=/absolute/path/to/your/subtheme \
  ./scripts/update-build-and-storybook.sh

# 4) (Optional) Convert Storybook stories using AI
SUBTHEME_DIRECTORY=/absolute/path/to/your/subtheme \
ANTHROPIC_API_KEY=$ANTHROPIC_API_KEY \
node -e "import('./scripts/convert-subtheme-storybook.mjs').then(m => m.default())"
```

**Observed issues and workarounds**

- *Model not found / unsupported*: Edit `scripts/convert-subtheme-storybook.mjs`
  and set `model` to an available Anthropic model (e.g.
  `claude-sonnet-4-5-20250929`).
- *Sass embedded binary missing*: Install optional platform-specific
  `sass-embedded-*` packages or temporarily switch `build.js` to use `sass`
  instead of `sass-embedded` and rerun the build.
- *CivicTheme path detection wrong*: After the tool runs, ensure
  `build-config.json` points to the contrib Civictheme directory (e.g.
  `../contrib/civictheme`). If overwritten, restore the correct path before
  building.
- *Interactive menu crashes*: Use the direct script invocation above instead of
  the interactive `npm run update-storybook` prompt.
- *Story diffs*: Manually review converted stories—long/complex stories may
  need hand edits.

### 2.5 Document findings (T104)

Update the customisation register with:
- List of templates requiring include syntax updates.
- List of templates with extends/parent() patterns (HIGH PRIORITY).
- List of templates with old block names.
- Library override changes required.
- Build tooling changes required.

Assign impact levels:
- **HIGH**: Extended templates, include syntax, library overrides.
- **MEDIUM**: Block name changes, build tooling.
- **LOW**: Configuration review.

---

## 3. Apply changes

**Related tasks**: T110–T118

### 3.1 Upgrade CivicTheme via Composer (T110)

```bash
cd /path/to/drupal/project

# Update CivicTheme to exact target version
# IMPORTANT: Use exact version constraint (no ^ or ~) to ensure
# sequential upgrades install precisely the intended release.
composer require drupal/civictheme:1.11.0

# Verify the update
composer show drupal/civictheme | grep versions
# Expected: 1.11.0 (exact version, not 1.11.x)

# Commit changes
git add composer.json composer.lock
git commit -m "feat: Updated CivicTheme from 1.10.0 to 1.11.0"
```

### 3.2 Update Twig include syntax (T111) – HIGH PRIORITY

For each file identified in T102a, update the include statements.

**Common replacements** (adapt paths to your sub-theme structure):

```bash
SUBTHEME_PATH=/path/to/your/subtheme

# Preview changes first (dry run)
grep -rl "include '@atoms/" $SUBTHEME_PATH/templates/ | while read file; do
  echo "Would update: $file"
done

# Example sed commands for common patterns:
# Button
sed -i '' "s/@atoms\/button\/button\.twig/civictheme:button/g" $SUBTHEME_PATH/templates/**/*.twig
# Paragraph
sed -i '' "s/@atoms\/paragraph\/paragraph\.twig/civictheme:paragraph/g" $SUBTHEME_PATH/templates/**/*.twig
# Link
sed -i '' "s/@atoms\/link\/link\.twig/civictheme:link/g" $SUBTHEME_PATH/templates/**/*.twig
# Heading
sed -i '' "s/@atoms\/heading\/heading\.twig/civictheme:heading/g" $SUBTHEME_PATH/templates/**/*.twig
# Icon
sed -i '' "s/@base\/icon\/icon\.twig/civictheme:icon/g" $SUBTHEME_PATH/templates/**/*.twig
# Logo
sed -i '' "s/@molecules\/logo\/logo\.twig/civictheme:logo/g" $SUBTHEME_PATH/templates/**/*.twig
# Item list
sed -i '' "s/@base\/item-list\/item-list\.twig/civictheme:item-list/g" $SUBTHEME_PATH/templates/**/*.twig
```

**Manual review required**: Open each updated file and verify the changes
are correct. The props passed to components should remain unchanged.

### 3.3 Refactor extended templates (T112) – HIGH PRIORITY

For each file identified in T102b with `{% extends %}` or `{{ parent() }}`:

**Option A – Override completely** (recommended for significant customisations):

1. Find the upstream template in CivicTheme 1.11.0:
   ```bash
   # Example: find button template
   find /path/to/themes/contrib/civictheme/templates -name "button.twig"
   ```

2. Copy the entire upstream template to your sub-theme:
   ```bash
   cp /path/to/civictheme/templates/component.twig \
      $SUBTHEME_PATH/templates/component.twig
   ```

3. Apply your customisations directly to the copied template.

4. Remove any `{% extends %}` or `{{ parent() }}` patterns.

**Option B – Remove override** (for minor customisations):

If the upstream defaults are acceptable, simply delete the sub-theme
template override.

### 3.4 Update Twig block names (T113)

```bash
SUBTHEME_PATH=/path/to/your/subtheme

# Replace _slot with _block in all templates
sed -i '' "s/content_slot/content_block/g" $SUBTHEME_PATH/templates/**/*.twig
sed -i '' "s/content_top_slot/content_top_block/g" $SUBTHEME_PATH/templates/**/*.twig
sed -i '' "s/content_bottom_slot/content_bottom_block/g" $SUBTHEME_PATH/templates/**/*.twig
sed -i '' "s/sidebar_top_left_slot/sidebar_top_left_block/g" $SUBTHEME_PATH/templates/**/*.twig
sed -i '' "s/sidebar_top_right_slot/sidebar_top_right_block/g" $SUBTHEME_PATH/templates/**/*.twig
sed -i '' "s/sidebar_bottom_left_slot/sidebar_bottom_left_block/g" $SUBTHEME_PATH/templates/**/*.twig
sed -i '' "s/sidebar_bottom_right_slot/sidebar_bottom_right_block/g" $SUBTHEME_PATH/templates/**/*.twig
sed -i '' "s/rows_slot/rows_block/g" $SUBTHEME_PATH/templates/**/*.twig
sed -i '' "s/footer_slot/footer_block/g" $SUBTHEME_PATH/templates/**/*.twig
sed -i '' "s/hidden_slot/hidden_block/g" $SUBTHEME_PATH/templates/**/*.twig

# Verify no _slot patterns remain
grep -rn "_slot %}" $SUBTHEME_PATH/templates/
```

### 3.5 Update library overrides (T114)

Edit `<subtheme>.info.yml` and update the `libraries-override` section:

```yaml
# BEFORE (1.10.0)
libraries-override:
  civictheme/global:
    css:
      theme:
        dist/civictheme.css: dist/styles.css
    js:
      dist/civictheme.js: dist/scripts.js

# AFTER (1.11.0)
libraries-override:
  civictheme/global:
    css:
      theme:
        dist/civictheme.base.css: dist/styles.base.css
        dist/civictheme.theme.css: dist/styles.theme.css
        dist/civictheme.variables.css: dist/styles.variables.css
    js:
      dist/civictheme.drupal.base.js: dist/scripts.drupal.base.js

  civictheme/css-variables:
    css:
      theme:
        dist/civictheme.variables.css: dist/styles.variables.css
```

### 3.6 Update libraries.yml (T115)

Edit `<subtheme>.libraries.yml`:

```yaml
global:
  css:
    theme:
      dist/styles.base.css: {}
      dist/styles.theme.css: {}
      dist/styles.variables.css: {}
  js:
    dist/scripts.drupal.base.js: {}
  dependencies:
    - core/drupal
    - core/once
    - core/drupalSettings

css-variables:
  css:
    theme:
      dist/styles.variables.css: { preprocess: false, weight: 10 }
```

### 3.7 Restore custom library attachments (T115a)

Use the table you wrote in `planning.md` during T103e to re-attach any
custom libraries that were previously loading bespoke CSS/JS:

- Re-add the libraries to `<subtheme>.libraries.yml` if they were removed or
  renamed.
- Restore `attach_library()` calls (Twig) or `#attached` entries
  (preprocess/hooks) where they originally loaded.
- Clear caches and spot-check pages that rely on those libraries (events,
  workshop filters, feedback blocks, etc.).

### 3.8 Update build tooling (T116)

**Option A – Use SDC Update Tool** (recommended):

---

#### ⛔ STOP CONDITION – Anthropic API Key Required

**AI assistants MUST stop here if the `ANTHROPIC_API_KEY` has not been
configured.** This tool requires the key to convert Storybook stories via the
Anthropic Claude API.

If you have not yet configured the key (or skipped Section 2.4), request
developer confirmation now:

**Note:** Option A is an **optional helper**. If the developer prefers not to
configure an Anthropic API key, skip Option A and proceed directly to
**Option B – Manual update** below. The CivicTheme upgrade can continue
without the key.

**Action required from developer:**

1. **Preferred method** – Add to the destination project's `.env` file:

   ```bash
   echo 'ANTHROPIC_API_KEY=sk-ant-...' >> .env
   ```

2. **Alternative method** – Export in your shell session or add to
   `~/.bashrc` / `~/.zshrc`:

   ```bash
   export ANTHROPIC_API_KEY=sk-ant-...
   ```

**Developer confirmation required before proceeding:**

> - [ ] `ANTHROPIC_API_KEY` is available (in `.env` or exported in shell)
>
> Once confirmed, the AI assistant may proceed.

---

```bash
# Clone the upgrade tools repo
cd /tmp
git clone https://github.com/civictheme/upgrade-tools.git
cd upgrade-tools/sdc-update

# Install dependencies
npm install

# Create .env file for the tool (copy key from project .env or shell)
cat > .env << EOF
SUBTHEME_PATH=/absolute/path/to/your/subtheme
ANTHROPIC_API_KEY=$ANTHROPIC_API_KEY
EOF

# Run the update
npm run update-components

# Review the output
ls -la .logs/

# Review changes in sub-theme
cd /path/to/your/subtheme
git diff
```

**Post-completion reminder:** See Section 3.11 for API key removal steps
(T119). Do not proceed to validation until the key has been removed and
verified (T129).

**Option B – Manual update**:

1. Download CivicTheme 1.11.0 starter kit files:
   ```bash
   # Copy build files from CivicTheme starter kit
   CIVICTHEME=/path/to/themes/contrib/civictheme
   STARTER=$CIVICTHEME/civictheme_starter_kit
   SUBTHEME=/path/to/your/subtheme

   cp $STARTER/build.js $SUBTHEME/
   cp $STARTER/build-config.json $SUBTHEME/ 2>/dev/null || true
   cp $STARTER/.storybook/sdc-plugin.js $SUBTHEME/.storybook/
   ```

2. Update `package.json` to include `@civictheme/sdc`:
   ```bash
   cd $SUBTHEME
   npm install @civictheme/sdc@^1.11 --save-dev
   ```

3. Update `.storybook/preview.js` to match starter kit.

### 3.9 Add SDC prop schema enforcement (T117) – Optional

Add to `<subtheme>.info.yml`:

```yaml
enforce_prop_schemas: true
```

This enables validation of component props against schemas.

### 3.10 Rebuild sub-theme assets

```bash
# RECOMMENDED: Using ahoy fe (equivalent to npm run build from theme directory)
ahoy fe

# Alternative: Native (non-Docker environments)
cd $SUBTHEME_PATH
npm install
npm run build  # or npm run dist

# Alternative: Using docker compose directly
docker compose exec cli bash -c "cd web/themes/custom/yourtheme && npm install && npm run build"

# Verify output files exist
ls -la $SUBTHEME_PATH/dist/
# Expected: styles.base.css, styles.theme.css, styles.variables.css,
#           scripts.drupal.base.js
```

### 3.10 Commit changes

```bash
git add -A
git status  # Review all changes

# Commit in logical chunks if preferred
git commit -m "feat: Updated sub-theme for CivicTheme 1.11 SDC migration"
```

### 3.11 Remove Anthropic API key (T119)

**Critical security task**: Remove the `ANTHROPIC_API_KEY` immediately after
completing the upgrade-tools steps to prevent accidental commit of sensitive
credentials.

**If added to `.env` file:**

```bash
# Remove the API key line from .env
sed -i '' '/ANTHROPIC_API_KEY/d' .env

# Verify removal
grep -i "ANTHROPIC_API_KEY" .env
# Expected: No matches (empty output)
```

**If exported in shell:**

```bash
# Remove from current session
unset ANTHROPIC_API_KEY

# If added to shell config, remove the export line
sed -i '' '/ANTHROPIC_API_KEY/d' ~/.bashrc  # or ~/.zshrc
source ~/.bashrc  # or source ~/.zshrc
```

**Verification:** Proceed to Section 4.8 for comprehensive verification
steps (T129).

### 3.12 Known issues and workarounds

- **InvalidComponentException: Unable to find Twig template for component** —
  can occur if a custom component directory contains a `.component.yml` but no
  matching `.twig` after SDC conversion (e.g. custom Table of Contents). Fix by
  adding the missing Twig template (copy upstream component if needed) or
  removing the stale `.component.yml`, then clear caches (`drush cr`).
- **Sass embedded binary missing** — install optional `sass-embedded-<platform>`
  packages or temporarily point `build.js` to `sass` and rerun the build.
- **Story conversion model unavailable** — set `model` in
  `scripts/convert-subtheme-storybook.mjs` to an available Anthropic model
  (e.g. `claude-sonnet-4-5-20250929`).
- **Wrong CivicTheme path in build-config** — ensure `build-config.json`
  `civicthemePath` points to the contrib Civictheme directory after running
  upgrade-tools.

---

## 4. Validate

**Related tasks**: T120–T128

### 4.1 Clear caches and run updates (T120)

```bash
# Clear all Drupal caches (use ahoy/docker as appropriate)
drush cr
# Or: ahoy drush cr
# Or: docker compose exec cli drush cr

# Check for and run database updates
drush updb -y

# Import configuration if pending
drush cim -y

# Check for errors during these operations
echo $?  # Should be 0
```

### 4.2 Check logs for SDC-related errors (T121)

```bash
# View recent watchdog entries
drush watchdog:show --count=50

# Filter for CivicTheme-related errors
drush watchdog:show --filter="civictheme" --count=50
drush watchdog:show --filter="component" --count=50
drush watchdog:show --filter="sdc" --count=50
```

**Expected**: No errors related to CivicTheme or SDC components.

**Common error patterns to watch for**:
- `ComponentNotFoundException`: A component include is using wrong syntax.
- `Unable to find alert component`: SDC plugin manager issue.
- Twig syntax errors: Template syntax problems.

### 4.3 Validate page rendering (T122)

Open the site in a browser and manually check:

- [ ] **Home page** (T122a):
  - Header renders correctly.
  - Footer renders correctly.
  - Navigation works.
  - Content displays properly.

- [ ] **Navigation** (T122b):
  - Primary navigation works.
  - Secondary navigation works.
  - Mobile menu opens and closes.
  - Menu links navigate correctly.

- [ ] **Search** (T122c):
  - Search form displays.
  - Search results return.
  - Results are in correct relevance order (fix in 1.11.0).

- [ ] **Content pages** (T122d):
  - Article pages render.
  - Landing pages render.
  - Page content types render.
  - All paragraphs/components display.

- [ ] **Forms** (T122e):
  - Contact forms display.
  - Form submissions work.
  - Validation messages appear.

### 4.4 Validate specific components (T123)

- [ ] **Alerts** (T123a): Check alert banners render.
- [ ] **Table of contents** (T123b): If used, verify TOC renders.
- [ ] **Cards** (T123c): Check various card types.
- [ ] **Automated lists** (T123d): Verify list rendering.
- [ ] **Buttons** (T123e): Check button styles and states.
- [ ] **Icons** (T123f): Verify icons display.

### 4.5 Validate Storybook (T124) – if used

```bash
# Start Storybook
# RECOMMENDED: Using dedicated ahoy storybook (if available)
ahoy storybook

# Alternative: Native (non-Docker environments)
cd $SUBTHEME_PATH
npm run storybook

# Alternative: Using docker compose directly
docker compose exec cli bash -c "cd web/themes/custom/yourtheme && npm run storybook"

# In browser, check:
# - Build completes without errors
# - Components render in sidebar
# - Individual component stories render
# - SDC components have correct props
```

### 4.6 Test sub-theme build (T125)

```bash
cd $SUBTHEME_PATH

# Run full build (use docker if npm is in container)
npm run build  # or npm run dist
# Or: docker compose exec cli bash -c "cd web/themes/custom/yourtheme && npm run build"

# Check exit code
echo $?  # Should be 0

# Verify output files
ls -la dist/

# Expected files:
# - styles.base.css
# - styles.theme.css
# - styles.variables.css
# - scripts.drupal.base.js
```

### 4.7 Run tests AFTER upgrade (T125b)

Re-run the same tests discovered in Section 1.9 to verify the upgrade has
not introduced regressions.

```bash
# Run all available tests (use commands discovered in 1.9)
# Example using ahoy:
ahoy test-unit
ahoy test-bdd
ahoy test-playwright

# Example using composer:
composer test

# Example using docker compose directly:
docker compose exec cli ./vendor/bin/phpunit
docker compose exec cli ./vendor/bin/behat
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

### 4.8 Verify Anthropic API key removal (T129)

**Critical verification**: Before committing or merging, ensure no API keys
are present in tracked files or staging area.

```bash
# Check .env file (if exists)
if [ -f ".env" ]; then
  echo "=== Checking .env ==="
  if grep -qi "ANTHROPIC_API_KEY" .env; then
    echo "⚠️  WARNING: ANTHROPIC_API_KEY found in .env - REMOVE IMMEDIATELY"
    exit 1
  fi
  echo "✅ .env file checked - no API key found"
fi

# Check Git staging area
echo "=== Checking Git staging area ==="
if git diff --cached | grep -qi "ANTHROPIC_API_KEY"; then
  echo "⚠️  WARNING: ANTHROPIC_API_KEY found in staged changes - REMOVE IMMEDIATELY"
  exit 1
fi
echo "✅ Staging area checked - no API key found"

# Check Git working directory
echo "=== Checking Git working directory ==="
if git diff | grep -qi "ANTHROPIC_API_KEY"; then
  echo "⚠️  WARNING: ANTHROPIC_API_KEY found in working directory changes - REMOVE IMMEDIATELY"
  exit 1
fi
echo "✅ Working directory checked - no API key found"

echo "✅ API key verification complete - no keys found in tracked files"
```

**Blocker**: If any API keys are found, remove them immediately before
proceeding. Do not commit API keys to version control.

### 4.9 Document validation results

Create a checklist of validation results:

```markdown
## Validation Results – CivicTheme 1.10.0 → 1.11.0

Date: YYYY-MM-DD
Tester: [Name]

### Test Suite Results
| Test Suite | Before Upgrade | After Upgrade | Notes |
|------------|----------------|---------------|-------|
| Unit tests | PASS/FAIL | PASS/FAIL | |
| BDD tests | PASS/FAIL | PASS/FAIL | |
| E2E tests | PASS/FAIL | PASS/FAIL | |

### Page Rendering
- [ ] Home page: PASS/FAIL
- [ ] Navigation: PASS/FAIL
- [ ] Search: PASS/FAIL
- [ ] Content pages: PASS/FAIL
- [ ] Forms: PASS/FAIL

### Components
- [ ] Alerts: PASS/FAIL
- [ ] Cards: PASS/FAIL
- [ ] Buttons: PASS/FAIL
- [ ] Icons: PASS/FAIL

### Build
- [ ] npm run build: PASS/FAIL
- [ ] Storybook: PASS/FAIL

### Issues Found
1. [Description of any issues]

### Notes
[Any additional observations]
```

---

## 5. Capture outcomes and follow-ups

**Related tasks**: T126, T127, T128

### 5.1 Update customisation register (T126)

Edit `docs/civic-theme-upgrades/customisations.md` and update each entry
affected by this upgrade:

```markdown
| ID | Description | Status | Notes |
|----|-------------|--------|-------|
| C001 | Button override | Updated | Changed to SDC include syntax |
| C002 | Extended card template | Refactored | Now complete override |
| C003 | Custom component X | Unchanged | No changes required |
```

### 5.2 Document lessons learned (T127)

Add a "Lessons Learned" section to this file or create a separate note:

```markdown
## Lessons Learned – CivicTheme 1.10.0 → 1.11.0

### What went well
- [List things that worked smoothly]

### Challenges encountered
- [List difficulties and how they were resolved]

### Recommendations for future upgrades
- [List advice for future SDC-related upgrades]

### Time spent
- Discovery: X hours
- Changes: X hours
- Validation: X hours
- Documentation: X hours
- Total: X hours
```

### 5.3 Commit all documentation

```bash
cd /path/to/drupal/project

# Add all documentation changes
git add docs/civic-theme-upgrades/

# Commit
git commit -m "docs: Completed CivicTheme 1.10.0 to 1.11.0 upgrade documentation"
```

### 5.4 Prepare for review (T128)

Create a merge request with the following information:

```markdown
## CivicTheme 1.10.0 → 1.11.0 Upgrade

### Summary
This PR upgrades CivicTheme from 1.10.0 to 1.11.0, including migration
to Single Directory Components (SDC).

### Changes Made
- Updated CivicTheme via Composer
- Migrated X templates to SDC include syntax
- Refactored X extended templates to complete overrides
- Updated X block names from _slot to _block suffix
- Updated library overrides for new asset file names
- Updated build tooling for SDC compatibility

### Customisations Impacted
- C001: [Description]
- C002: [Description]

### Testing Performed
- [ ] All pages render correctly
- [ ] Navigation works
- [ ] Search returns correct results
- [ ] Forms function properly
- [ ] Storybook builds successfully

### Known Issues / Follow-ups
- CivicTheme 1.11 splits CSS by component and no longer bundles custom component styles into a single `styles.css`. Any bespoke Twig that renders custom markup (not an SDC component) can lose styling. Fix: add a theme library that points to the generated component CSS (e.g. `components_combined/05-pages/<component>/<component>.css`) and attach it in the Twig template, or restore a project-level bundle that imports those components. Example fix for events: add an `event-page` library pointing to `components_combined/05-pages/event/event.css` and call `attach_library('theme/event-page')` on the event node template.
- Bespoke filter/banner components that layer custom markup over CivicTheme list backgrounds can lose styles after the 1.11 CSS split when their libraries ship JS only. Fix by bundling the compiled custom CSS together with any dependent CivicTheme CSS (for example, the list organism CSS) in the same library and ensuring the page keeps `attach_library()`/`#attached` calls so backgrounds and buttons stay styled.

### Documentation
See `docs/civic-theme-upgrades/versions/v1.10.0-to-v1.11.0/` for full
upgrade documentation.
```

---

## Appendix: Quick reference

### Include syntax mapping

| Before (1.10.0) | After (1.11.0) |
|-----------------|----------------|
| `@atoms/paragraph/paragraph.twig` | `civictheme:paragraph` |
| `@atoms/button/button.twig` | `civictheme:button` |
| `@atoms/link/link.twig` | `civictheme:link` |
| `@atoms/heading/heading.twig` | `civictheme:heading` |
| `@molecules/logo/logo.twig` | `civictheme:logo` |
| `@base/icon/icon.twig` | `civictheme:icon` |
| `@base/item-list/item-list.twig` | `civictheme:item-list` |
| `@base/grid/grid.twig` | `civictheme:grid` |
| `@base/layout/layout.twig` | `civictheme:layout` |
| `@base/menu/menu.twig` | `civictheme:menu` |

### Block name mapping

| Before (1.10.0) | After (1.11.0) |
|-----------------|----------------|
| `content_slot` | `content_block` |
| `content_top_slot` | `content_top_block` |
| `content_bottom_slot` | `content_bottom_block` |
| `sidebar_top_left_slot` | `sidebar_top_left_block` |
| `sidebar_top_right_slot` | `sidebar_top_right_block` |
| `sidebar_bottom_left_slot` | `sidebar_bottom_left_block` |
| `sidebar_bottom_right_slot` | `sidebar_bottom_right_block` |
| `rows_slot` | `rows_block` |
| `footer_slot` | `footer_block` |
| `hidden_slot` | `hidden_block` |

### Asset file mapping

| Before (1.10.0) | After (1.11.0) |
|-----------------|----------------|
| `dist/civictheme.css` | `dist/civictheme.base.css` |
| | `dist/civictheme.theme.css` |
| | `dist/civictheme.variables.css` |
| `dist/civictheme.js` | `dist/civictheme.drupal.base.js` |

---

This completes the runbook for the CivicTheme `1.10.0` → `1.11.0` upgrade.
For questions or issues, consult the upstream documentation at
https://docs.civictheme.io/changelog/civictheme-release-notes/civictheme-1.11.0
