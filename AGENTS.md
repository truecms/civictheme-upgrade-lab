# civictheme-upgrade-assistant Development Guidelines

Auto-generated from all feature plans. Last updated: 2025-11-29

## Active Technologies
- Git-tracked files only (no databases or external storage) (001-rely-contents-docs)

- Bash 5.x (scaffolding scripts), Markdown documentation; applies to downstream Drupal projects using CivicTheme + Git, local file system, CivicTheme upstream documentation, Drupal + composer tooling in destination projects (001-rely-contents-docs)

## Project Structure

```text
.specify/                    # Templates, scripts, and constitution for AI tooling
  memory/
    constitution.md          # Framework governance and principles
  scripts/
    bash/                    # Scaffolding shell scripts
  templates/                 # Document templates (spec, plan, tasks, etc.)
docs/
  civic-theme-upgrades/
    customisations.md        # Canonical customisation register (template)
    README.md                # Entry point for upgrade documentation
    versions/
      vX.Y.Z-to-vA.B.C/      # Per-version upgrade directories
        spec.md              # What & why (planning, analysis)
        tasks.md             # Checklist of work
        playbook.md          # How (ordered runbook)
  planning.md                # Global upgrade documentation framework
specs/
  NNN-feature-name/          # Feature specifications (internal planning)
```

## Code Style

- Markdown: Follow CommonMark conventions with Australian English spelling
- Shell scripts: Bash 5.x compatible, shellcheck compliant

<!-- MANUAL ADDITIONS START -->

## CivicTheme Upgrade Assistant Framework

- This repository defines a **documentation-first CivicTheme upgrade assistant** used by both humans and AI coding assistants.
- Destination projects MUST keep a canonical customisation register at
  `docs/civic-theme-upgrades/customisations.md`.
- Each CivicTheme version step MUST live under
  `docs/civic-theme-upgrades/versions/vX.Y.Z-to-vA.B.C/` and contain:
  - `spec.md` (planning & analysis),
  - `tasks.md` (checklist of work),
  - `playbook.md` (ordered runbook).
- Upgrades are **sequential and single-release**: one upstream CivicTheme
  release per `vX.Y.Z-to-vA.B.C` directory; do not combine multiple
  version jumps into a single plan.
- AI assistants SHOULD:
  - Refresh the customisation register on every run.
  - Use the per-version `spec.md` + `tasks.md` + `playbook.md` trio as the
    primary guide for upgrades, validating outcomes via checklists and
    repository inspection rather than relying on automated tests.


<!-- MANUAL ADDITIONS END -->
