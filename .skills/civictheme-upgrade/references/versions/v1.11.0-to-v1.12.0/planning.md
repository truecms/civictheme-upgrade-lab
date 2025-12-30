# Planning notes: CivicTheme v1.11.0 → v1.12.0

**Directory**: `docs/civic-theme-upgrades/versions/v1.11.0-to-v1.12.0/`  
**Related documents**: `spec.md` (what/why), `tasks.md` (checklist), `playbook.md` (how)

This file is an optional planning scratchpad for developers and AI coding
assistants. It is intended to capture early thoughts and prompts **before**
or while refining `spec.md`, `tasks.md` and `playbook.md`. The canonical
source of truth for this upgrade remains the spec/tasks/playbook trio.

---

## 1. Key upstream changes summary

CivicTheme 1.12.0 is a **security release** with incremental feature
additions. Unlike the 1.10.0 → 1.11.0 upgrade (which involved major SDC
migration), this is a more straightforward upgrade focused on patching
vulnerabilities.

### Security fixes (CRITICAL)

| Advisory | Type | Severity |
|----------|------|----------|
| SA-CONTRIB-2025-112 | Information disclosure | Moderately critical |
| SA-CONTRIB-2025-113 | Cross-site Scripting (XSS) | Moderately critical |

**Urgency**: All sites on CivicTheme 1.11.0 or earlier should upgrade to
1.12.0 to remediate security risks. Sub-themes may also carry these risks
if they override affected components.

#### Key security changes in 1.12.0

1. **Twig template XSS fixes**:
   - `heading.twig`: Removed `|raw` from `{{ content }}`
   - `button.twig`: Removed `|raw` from `{{ text }}`

2. **CivicTheme API updates for access control**:
   - `civictheme_get_field_referenced_entities()` now requires `$build` arg
   - `civictheme_get_field_referenced_entity()` now requires `$build` arg
   - `civictheme_get_field_value()` now requires `build:` named parameter
   - These functions now check entity access and manage cache metadata

3. **XSS filtering in preprocess**:
   - Title resolver results filtered with `Xss::filter()` + `strip_tags()`
   - Link text from `getText()` filtered with `Xss::filter()`
   - Menu item titles filtered with `Xss::filter()`

4. **iframe paragraph security**:
   - `field_c_p_attributes` removed to prevent attribute injection

5. **Permission updates**:
   - Content Author/Approver should not edit Icons media type (SVG XSS risk)

### New features

| Feature | Description | Priority |
|---------|-------------|----------|
| Multi-line header | Full-width nav with logo above | Optional |
| Message organism | 4-style message component | Optional |
| Inline filter | Keyword in search results | Automatic |
| Fast fact card | Simple fact display card | Optional |
| GovCMS automation | Setup script for GovCMS | GovCMS only |

### SDC continued improvement

- CSS versions of SCSS variables added.
- CSS variables for typography use.
- SDC validation and error handling improvements.

### Bug fixes

- Enhanced menu theming system.
- Archived webforms no longer break site.
- Search form element issues resolved.
- Email addresses not incorrectly converted to links.
- Logo paths with spaces handled correctly.
- Webform striptags error fixed.
- `menu_level_modifier_class` scope fixed.
- Attachment file extension reset fixed.
- Mobile menu visibility fixed.

### Upstream references consulted

- Release notes: https://docs.civictheme.io/changelog
- Security advisories: SA-CONTRIB-2025-112, SA-CONTRIB-2025-113
- Git diff: https://git.drupalcode.org/project/civictheme/-/compare/1.11.0...1.12.0

---

## 2. Planning focus

### Environment assumptions

- [ ] Working in non-production environment (local/dev/staging).
- [ ] Database and file backups exist.
- [ ] Dedicated feature branch created.
- [ ] Drupal core meets `^10.2 || ^11` (unchanged from 1.11.0).

### Recommended task order

1. **Discovery** (audit scope of changes):
   - Review security advisories.
   - Audit sub-theme for security-relevant overrides.
   - Document customisation impact.
   - Capture current custom library attachments (libraries.yml entries and
     Twig/`#attached` usages) in the table below before making any changes.

2. **High-priority changes** (security):
   - Update Composer dependencies.
   - Security review of sub-theme overrides.
   - Apply any necessary fixes to sub-theme templates.

3. **Medium-priority changes** (bug fixes and improvements):
   - Update menu customisations if affected.
   - Update attachment overrides if affected.
   - Consider CSS variable migration (optional).

4. **Validation** (before merge):
   - Clear caches, run updates.
   - Test all pages and components.
   - Verify security fixes are effective.
   - Document results.

### Project-specific customisations to review

List customisation IDs from the register that are likely impacted:

- `C???` – [Add customisation description]
- `C???` – [Add customisation description]

*(Update this list after running the discovery audit in Section 2 of
the playbook.)*

---

## 3. Open questions / risks

### Pre-upgrade capture: custom library attachments

| Library | Files referenced | Where attached (Twig/preprocess) | Notes |
|---------|------------------|-----------------------------------|-------|
| (fill)  |                  |                                   |       |

Use this table during T204c to list every custom sub-theme library and its
attach points so they can be restored after the upgrade if overrides are
lost.

### Resolved

- [x] **Q: What security vulnerabilities are addressed?**
  - A: Information disclosure (SA-CONTRIB-2025-112) and Cross-site
    Scripting (SA-CONTRIB-2025-113). Both are moderately critical.

- [x] **Q: Do sub-themes need security review?**
  - A: Yes. Sub-themes that override CivicTheme templates may carry the
    same XSS and information disclosure risks. Review any overrides that
    handle user input or display user-generated content.

- [x] **Q: Is this upgrade as complex as 1.10.0 → 1.11.0?**
  - A: No. The 1.11.0 release involved major SDC migration. The 1.12.0
    release is primarily security fixes and incremental improvements.
    Upgrade effort should be significantly lower.

- [x] **Q: Are there any breaking changes?**
  - A: No breaking changes documented. The upgrade is additive with bug
    fixes. However, sub-themes with menu, attachment, or content display
    overrides should be tested.

### Unresolved (project-specific)

- [ ] **Q: Does this project have templates that handle user input?**
  - A: Run T203a audit to determine.

- [ ] **Q: Does the project override menu templates?**
  - A: Run T203b audit to determine.

- [ ] **Q: Does the project override attachment components?**
  - A: Run T203c audit to determine.

- [ ] **Q: Should we migrate from SCSS to CSS variables?**
  - A: Evaluate based on project needs. This can be deferred if time is
    constrained – the security fixes are the priority.

---

## 4. Estimation notes

Based on upstream complexity and security review requirements:

| Phase | Estimated Effort |
|-------|------------------|
| Discovery (audit Twig + preprocess) | 1-2 hours |
| Remove `\|raw` from Twig templates | 0.5-1 hour |
| Update CivicTheme API calls ($variables) | 1-2 hours |
| Add XSS filtering to preprocess | 1-2 hours |
| Composer update + config changes | 0.5 hours |
| Menu/attachment updates | 0.5-1 hour (if applicable) |
| XSS testing | 1-2 hours |
| Validation | 1-2 hours |
| Documentation | 0.5-1 hour |
| **Total** | **7-13 hours** |

*(Adjust based on number of sub-theme overrides and preprocess functions.)*

**Key factors affecting effort**:

- Number of Twig template overrides using `|raw` filter
- Number of preprocess functions calling CivicTheme API
- Number of preprocess functions outputting titles/labels
- Presence of custom menu preprocessing
- Whether iframe paragraph attributes field is used

**Comparison with 1.10.0 → 1.11.0**: That upgrade was estimated at
9-22 hours due to the SDC migration. This upgrade is similar in scope
due to the security review requirements, but more focused on specific
patterns rather than wholesale architecture changes.

---

## 5. Decision log

Record key decisions made during planning:

| Date | Decision | Rationale |
|------|----------|-----------|
| YYYY-MM-DD | Prioritise security review over new feature adoption | Security fixes are the primary driver for this upgrade |
| YYYY-MM-DD | Defer CSS variable migration to follow-up task | Focus on security, keep upgrade scope manageable |
| YYYY-MM-DD | [Decision made] | [Why this approach was chosen] |

---

## 6. Comparison: 1.10.0→1.11.0 vs 1.11.0→1.12.0

| Aspect | 1.10.0 → 1.11.0 | 1.11.0 → 1.12.0 |
|--------|-----------------|-----------------|
| Primary focus | SDC migration (architectural) | Security fixes (XSS + info disclosure) |
| Breaking changes | Yes (major) | No (but API deprecations) |
| Twig syntax changes | Yes (all includes) | Yes (remove `\|raw` from overrides) |
| Block name changes | Yes (_slot → _block) | No |
| Asset file changes | Yes (CSS/JS split) | No |
| Build tooling changes | Yes (significant) | No |
| API changes | No | Yes (functions require $build arg) |
| Preprocess changes | No | Yes (XSS filtering required) |
| Sub-theme template updates | Required for all | Only if using `\|raw` on user data |
| Sub-theme preprocess updates | No | Yes (API calls + XSS filtering) |
| Estimated effort | 9-22 hours | 7-13 hours |
| Urgency | Medium (feature release) | **High (security release)** |

### Key differences in upgrade approach

| 1.10.0 → 1.11.0 | 1.11.0 → 1.12.0 |
|-----------------|-----------------|
| Focus on Twig include syntax | Focus on security patterns |
| Replace all `@atoms/...` includes | Audit and remove `\|raw` filter |
| Refactor extended templates | Update CivicTheme API calls |
| Update library overrides | Add XSS filtering to preprocess |
| Update build tooling | Run XSS tests |

---

Once tasks are stable, ensure they are recorded in `tasks.md` with clear IDs
and that `playbook.md` is updated to reflect the agreed execution order.
