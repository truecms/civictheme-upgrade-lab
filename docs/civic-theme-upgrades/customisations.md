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
