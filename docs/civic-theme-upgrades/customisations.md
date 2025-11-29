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

### Upgrade note: Workshop filter component styling

If a project provides a custom "workshop filter" banner (e.g., on
`/events/supervisory-groups-workshops`) that relies on bespoke classes
like `sts-workshop-filter*`, ensure the corresponding compiled CSS is
loaded. In 1.11+ only the JS may be attached, leaving the dark background,
layout spacing, spinner, and reset button unstyled. Mitigation:

- Add the compiled CSS to the same library as the JS, e.g.
  `components_combined/02-molecules/workshop-filter/workshop-filter.css`.
- Confirm the library is attached where the filter renders (page
  attachments alter hook or Twig). Keep this noted so future upgrades
  preserve the CSS attach and avoid the unstyled filter regression.

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
