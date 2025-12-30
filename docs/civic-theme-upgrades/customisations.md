# CivicTheme customisation register (template)

This file illustrates the canonical path and structure for the CivicTheme
customisation register:

- `docs/civic-theme-upgrades/customisations.md`

Destination Drupal projects that adopt this framework MUST maintain their
own version of this file, capturing project-specific CivicTheme
sub-themes and customisations.

Use this template as a starting point and replace the sample entry with
real customisations in each destination project.

- [ ] C001 Example sub-theme overrides (Twig, SCSS, JS) – replace with
      real items in destination projects.

---

## Recording parent-theme modifications

If direct modifications have been made to the upstream CivicTheme parent
theme (`web/themes/contrib/civictheme/`), these **MUST** be recorded here
as **HIGH** risk entries. Parent-theme modifications will be **overwritten**
during any upgrade and require special handling.

### How to detect parent-theme modifications

During pre-flight checks, compare the installed `web/themes/contrib/civictheme/`
against a pristine copy of the version shown in `composer.lock`. Any differences
indicate local modifications.

### Recording format for parent-theme modifications

Use this format for each modification found:

```markdown
- [ ] C0XX [HIGH] Parent theme modification - web/themes/contrib/civictheme/<path>
      Locked version: <version from composer.lock>
      Change: <brief description of what was modified>
      WARNING: Will be overwritten on upgrade – requires migration strategy
```

**Example entries**:

```markdown
- [ ] C010 [HIGH] Parent theme modification - web/themes/contrib/civictheme/templates/block/civictheme-banner.html.twig
      Locked version: 1.11.0
      Change: Added custom CTA button below banner title
      WARNING: Will be overwritten on upgrade – requires migration strategy

- [ ] C011 [HIGH] Parent theme modification - web/themes/contrib/civictheme/components/02-molecules/navigation/navigation.twig
      Locked version: 1.11.0
      Change: Modified mobile menu breakpoint logic
      WARNING: Will be overwritten on upgrade – requires migration strategy
```

### Handling parent-theme modifications during upgrades

When parent-theme modifications are detected:

1. **STOP** the upgrade process and notify the developer
2. The developer must decide how to handle each modification:
   - **Migrate to sub-theme**: Move the customisation to the sub-theme (preferred)
   - **Re-apply after upgrade**: Accept that the change will be lost and re-apply
     manually after upgrading
   - **Create a patch**: Generate a patch file that can be re-applied post-upgrade
   - **Abandon**: If the customisation is no longer needed, remove the register entry
3. Document the decision in the `Change:` field
4. Only proceed with the upgrade after all parent-theme modifications have a
   documented resolution strategy

---

## Recording Composer patches for CivicTheme

Projects using `cweagans/composer-patches` (or similar) may apply patches to
`drupal/civictheme`. These patches are **legitimate customisations** that:

- Cause the installed theme to differ from upstream (expected behaviour)
- Must be tracked in this register so they are not forgotten during upgrades
- May need updating or removal when upgrading to versions that include the fix

### How to detect Composer patches

Check `composer.json` for:

```bash
# Inline patches
grep -A50 '"patches"' composer.json | grep -A10 '"drupal/civictheme"'

# External patches file reference
grep '"patches-file"' composer.json
```

If `cweagans/composer-patches` is in `composer.lock`, the project uses Composer patches.

### Recording format for Composer patches

Use this format for each patch applied to CivicTheme:

```markdown
- [ ] C0XX [MEDIUM] Composer patch - <patch description>
      Locked version: <CivicTheme version this patch applies to>
      Patch source: <URL or local file path>
      Rationale: <why this patch is needed>
      Removal criteria: <when this patch can be removed, e.g., "fixed in 1.13.0">
```

**Example entries**:

```markdown
- [ ] C020 [MEDIUM] Composer patch - Fix mobile navigation accessibility
      Locked version: 1.11.0, 1.12.0
      Patch source: https://www.drupal.org/files/issues/2024-01-15/civictheme-nav-a11y-3412345-12.patch
      Rationale: Fixes WCAG 2.1 AA compliance issue with mobile menu focus trap
      Removal criteria: Fixed in CivicTheme 1.13.0 per issue #3412345

- [ ] C021 [MEDIUM] Composer patch - Custom banner height override
      Locked version: 1.12.0
      Patch source: patches/civictheme-banner-height.patch
      Rationale: Client requirement for taller hero banners on landing pages
      Removal criteria: Never (permanent customisation, re-apply on each upgrade)
```

### Handling Composer patches during upgrades

When upgrading CivicTheme:

1. **Review each patch** in the register against the target version's release notes
2. **Test if patch still applies** – patches may fail on new versions
3. **Check if patch is still needed** – the fix may be included upstream
4. **Update the register**:
   - Remove patches that are now included upstream
   - Update `Locked version` for patches that still apply
   - Note any patches that failed and need rework
5. **Re-run pre-flight comparison** after updating patches to verify no other modifications exist

### Impact on parent-theme modification detection

When comparing the installed theme against a pristine copy, **apply the same
Composer patches to the pristine copy** before diffing. This ensures only
*manual* modifications (not patch-based changes) trigger the "parent theme
modified" stop condition.

---

### Upgrade note: CivicTheme 1.11+ split CSS bundles

If a sub-theme renders bespoke Twig markup that uses CivicTheme classnames
without consuming the SDC components directly (e.g., custom event pages or
cards), the split CSS in 1.11+ will not load automatically. Mitigation:

- Create sub-theme libraries that point to the compiled component CSS files
  (e.g., `components_combined/05-pages/<component>/<component>.css` and
  `components_combined/02-molecules/<component>/<component>.css`).
- Attach those libraries in the relevant Twig templates (`attach_library()`)
  or rebuild a site-level CSS bundle that imports those components.
- Keep this noted in the project’s customisation register so future upgrades
  preserve the attaches/bundle and avoid unstyled pages.

*Example*: When listing events with custom Twig, badge/availability tags
(`ct-event__details-tag*`) are styled by the event page CSS
(`components_combined/05-pages/event/event.css`) **and** the event-card
molecule CSS (`components_combined/02-molecules/event-card/event-card.css`),
which provides the flex row that keeps tags aligned beside the date. Attach
both CSS files (via a sub-theme library) to the events view row so tags keep
their colors, pill styling, and horizontal placement after the 1.11+ split
bundles.

### Upgrade note: Custom filter/banner components using CivicTheme lists

If a project ships bespoke filter or banner components (e.g., day/workshop
filters) that sit on top of CivicTheme list markup (`ct-list`,
`ct-list--with-background`), ensure their libraries deliver **all** required
CSS. After the 1.11 split bundles, JS may load without the supporting styles,
leaving backgrounds, spacing, and buttons unstyled.

- Package the compiled CSS for the custom component **and** any CivicTheme
  dependencies (for example, `components_combined/03-organisms/list/list.css`
  when using list backgrounds) in the same library as the JS.
- Attach that library wherever the component renders (`attach_library()` in
  Twig or preprocess `#attached`). Record the dependency here so future
  upgrades can restore the attach if it is dropped.

### Upgrade note: Website feedback webform styling

If a project exposes a “Was this page helpful?” (or similar) webform block
that uses CivicTheme’s website-feedback component markup, ensure the library
also delivers the component CSS. From CivicTheme 1.11 onward, the split CSS
bundles may omit these styles unless explicitly attached. Recommended steps:

- Add the compiled CSS to the same library as the component JS, e.g.
  `components_combined/03-organisms/website-feedback/website-feedback.css`.
- Attach that library in preprocess or Twig (e.g., webform preprocess for the
  `website_feedback` form) so both CSS and JS load wherever the block renders.
- Record this dependency in the project’s customisation register to keep the
  attach in place during future upgrades or refactors.
