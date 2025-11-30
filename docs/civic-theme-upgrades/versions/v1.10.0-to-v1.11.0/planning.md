# Planning notes: CivicTheme v1.10.0 → v1.11.0

**Directory**: `docs/civic-theme-upgrades/versions/v1.10.0-to-v1.11.0/`  
**Related documents**: `spec.md` (what/why), `tasks.md` (checklist), `playbook.md` (how)

This file is an optional planning scratchpad for developers and AI coding
assistants. It is intended to capture early thoughts and prompts **before**
or while refining `spec.md`, `tasks.md` and `playbook.md`. The canonical
source of truth for this upgrade remains the spec/tasks/playbook trio.

---

## 1. Key upstream changes summary

CivicTheme 1.11.0 is a **major architectural release** that migrates all
components to Single Directory Components (SDC). This is not a typical
minor version bump – it requires significant sub-theme updates.

### Critical breaking changes

| Change | Impact | Priority |
|--------|--------|----------|
| SDC migration | All component includes must use new syntax | HIGH |
| Twig namespace change | `@atoms/...` → `civictheme:...` | HIGH |
| Component extension removed | `{% extends %}` no longer works | HIGH |
| Block name suffix change | `_slot` → `_block` | HIGH |
| Asset file restructure | CSS/JS file names changed | HIGH |
| Drupal core requirement | Now requires `^10.2 \|\| ^11` | MEDIUM |
| Build tooling updates | `build.js`, `package.json` changes | MEDIUM |

### Upstream references consulted

- Release notes: https://www.drupal.org/project/civictheme/releases/1.11.0
- Documentation: https://docs.civictheme.io/changelog/civictheme-release-notes/civictheme-1.11.0
- Git diff: https://git.drupalcode.org/project/civictheme/-/compare/1.10.0...1.11.0
- SDC Update Tool: https://github.com/civictheme/upgrade-tools/tree/main/sdc-update

---

## 2. Planning focus

### Environment assumptions

- [ ] Working in non-production environment (local/dev/staging).
- [ ] Database and file backups exist.
- [ ] Dedicated feature branch created.
- [ ] Drupal core is `>=10.2` (prerequisite).
- [ ] (Optional) Anthropic API key available **only if** you plan to use the
  CivicTheme upgrade-tools helper for Storybook story conversion (T116a). See
  Section 3 for configuration instructions; otherwise you may proceed with the
  manual build tooling path.

### Recommended task order

1. **Discovery** (audit scope of changes):
   - Verify Drupal core compatibility.
   - Audit all sub-theme templates for breaking patterns.
   - Document customisation impact.
   - Capture current custom library attachments (libraries.yml entries and
     Twig/`#attached` usages) in the table below before touching code.

2. **High-priority changes** (will cause immediate breakage):
   - Update Composer dependencies.
   - Update Twig include syntax (all templates).
   - Refactor extended templates (complete overrides or remove).
   - Update block names (`_slot` → `_block`).

3. **Medium-priority changes** (required for full functionality):
   - Update library overrides in `info.yml`.
   - Update `libraries.yml` asset references.
   - Update build tooling (`build.js`, `package.json`).

4. **Validation** (before merge):
   - Clear caches, run updates.
   - Test all pages and components.
   - Run Storybook build (if used).
   - Document results.

### Project-specific customisations to review

List customisation IDs from the register that are likely impacted:

- `C???` – [Add customisation description]
- `C???` – [Add customisation description]

*(Update this list after running the discovery audit in Section 2 of
the playbook.)*

---

## 3. Open questions / risks

### ⛔ STOP CONDITION – Anthropic API Key

The CivicTheme upgrade-tools (storybook-v8-update / sdc-update) require an
`ANTHROPIC_API_KEY` to convert Storybook stories via the Claude API.

**AI assistants MUST stop and request developer confirmation before proceeding
with the upgrade-tools.**

**Important:** These upgrade-tools are an **optional helper only**. If the
developer prefers not to configure an Anthropic API key, simply skip the tools
and use the manual build tooling path (T116b). The overall CivicTheme upgrade
can continue without an Anthropic key.

**Configuration options (choose one):**

1. **Preferred – Destination project `.env` file:**

   ```bash
   # In the destination project root directory
   echo 'ANTHROPIC_API_KEY=sk-ant-...' >> .env
   ```

   Ensure `.env` is listed in `.gitignore`.

2. **Alternative – Shell environment:**

   ```bash
   # Add to ~/.bashrc or ~/.zshrc
   export ANTHROPIC_API_KEY=sk-ant-...
   ```

   Then run `source ~/.bashrc` (or `source ~/.zshrc`).

**Developer confirmation checklist:**

- [ ] `ANTHROPIC_API_KEY` added to `.env` file (preferred)
- [ ] `ANTHROPIC_API_KEY` exported in shell / shell config (alternative)

**⚠️ Security reminder:** Remove the key after upgrade completion:
- Delete the line from `.env`, or
- Run `unset ANTHROPIC_API_KEY` in your shell.

---

### Pre-upgrade capture: custom library attachments

| Library | Files referenced | Where attached (Twig/preprocess) | Notes |
|---------|------------------|-----------------------------------|-------|
| (fill)  |                  |                                   |       |

Use this table during T103e to list every custom sub-theme library and its
attach points so they can be restored after the upgrade if overrides are
lost.

### Resolved

- [x] **Q: What are the exact file name changes?**
  - A: `dist/civictheme.css` → `civictheme.base.css`, `civictheme.theme.css`,
    `civictheme.variables.css`. `dist/civictheme.js` → `civictheme.drupal.base.js`.

- [x] **Q: What is the new Twig include syntax?**
  - A: `{% include 'civictheme:<component>' %}` instead of
    `{% include '@atoms/<component>/<component>.twig' %}`.

- [x] **Q: Can we still extend CivicTheme components?**
  - A: No. SDC does not support partial extension. Must override completely
    or use upstream unchanged.

### Unresolved (project-specific)

- [ ] **Q: Does this project have any extended templates?**
  - A: Run T102b audit to determine.

- [ ] **Q: Does the project use Storybook?**
  - A: If yes, build tooling update is required.

- [ ] **Q: Are there custom components that need SDC metadata?**
  - A: Consider running SDC Update Tool or manual `.component.yml` creation.

---

## 4. Estimation notes

Based on upstream complexity:

| Phase | Estimated Effort |
|-------|------------------|
| Discovery (audit) | 1-2 hours |
| Twig syntax updates | 2-4 hours (depends on template count) |
| Extended template refactoring | 2-8 hours (depends on complexity) |
| Library/build updates | 1-2 hours |
| Validation | 2-4 hours |
| Documentation | 1-2 hours |
| **Total** | **9-22 hours** |

*(Adjust based on project size and customisation complexity.)*

---

## 5. Decision log

Record key decisions made during planning:

| Date | Decision | Rationale |
|------|----------|-----------|
| YYYY-MM-DD | [Decision made] | [Why this approach was chosen] |

---

Once tasks are stable, ensure they are recorded in `tasks.md` with clear IDs
and that `playbook.md` is updated to reflect the agreed execution order.
