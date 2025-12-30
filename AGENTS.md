# civictheme-upgrade-assistant Development Guidelines

Auto-generated from all feature plans. Last updated: 2025-11-29

## Active Technologies
- Git-tracked files only (no databases or external storage) (001-rely-contents-docs)

- Bash 5.x (scaffolding scripts), Markdown documentation; applies to downstream Drupal projects using CivicTheme + Git, local file system, CivicTheme upstream documentation, Drupal + composer tooling in destination projects (001-rely-contents-docs)

## Project Structure

```text
.skills/                     # AI assistant skills (self-contained instruction sets)
  civictheme-upgrade/
    SKILL.md                 # Skill definition for CivicTheme upgrades
    references/              # Version-specific upgrade documentation
.specify/                    # Templates, scripts, and constitution for AI tooling
  memory/
    constitution.md          # Framework governance and principles
  scripts/
    bash/                    # Scaffolding shell scripts
  templates/                 # Document templates (spec, plan, tasks, etc.)
docs/
  civic-theme-upgrades/
    customisations.md        # Canonical customisation register (template)
    planning.md              # Global upgrade documentation framework
    README.md                # Entry point for upgrade documentation
    versions/
      vX.Y.Z-to-vA.B.C/      # Per-version upgrade directories
        spec.md              # What & why (planning, analysis)
        tasks.md             # Checklist of work
        playbook.md          # How (ordered runbook)
specs/
  NNN-feature-name/          # Feature specifications (internal planning)
```

## Code Style

- Markdown: Follow CommonMark conventions with Australian English spelling
- Shell scripts: Bash 5.x compatible, shellcheck compliant

<!-- MANUAL ADDITIONS START -->

## Skills

This repository includes AI-assistant skills in the `.skills/` directory. These are self-contained instruction sets designed for AI coding assistants (originally developed for Claude Code, but applicable to other AI tools).

| Skill | Description | Location |
|-------|-------------|----------|
| **civictheme-upgrade** | Plan and execute CivicTheme version upgrades in Drupal projects | [`.skills/civictheme-upgrade/SKILL.md`](.skills/civictheme-upgrade/SKILL.md) |

### civictheme-upgrade Skill

The CivicTheme upgrade skill provides structured guidance for:

- **Sequential version upgrades**: One CivicTheme release at a time (e.g., 1.10→1.11, 1.11→1.12)
- **SDC migration**: Handling Single Directory Components introduced in 1.11+
- **Twig syntax updates**: Converting legacy include paths to new SDC namespaces
- **Build tooling changes**: Updating `package.json`, `build.js`, and Storybook configurations
- **Customisation preservation**: Tracking and maintaining site-specific overrides

The skill includes its own `references/` directory with version-specific documentation (`spec.md`, `tasks.md`, `playbook.md`) for each supported upgrade path.

**Usage**: AI assistants should read the skill file when working on CivicTheme upgrades. The skill can be extended with project-specific context (e.g., custom theme location, existing customisations) via the `references/` directory.

## CivicTheme Upgrade Assistant Framework

- This repository defines a **documentation-first CivicTheme upgrade assistant** used by both developers and AI coding assistants.
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
- **Exact version constraints are REQUIRED**: When upgrading CivicTheme via
  Composer, ALWAYS use exact version constraints (e.g. `drupal/civictheme:1.12.0`)
  and NEVER use caret (`^`) or tilde (`~`) constraints. This ensures:
  - The upgrade installs exactly the intended target version.
  - Sequential upgrade documentation remains valid and predictable.
  - Developers can follow playbooks step-by-step without unexpected version skips.
- AI assistants SHOULD:
  - Refresh the customisation register on every run.
  - Use the per-version `spec.md` + `tasks.md` + `playbook.md` trio as the
    primary guide for upgrades, validating outcomes via checklists and
    repository inspection rather than relying on automated tests.


<!-- MANUAL ADDITIONS END -->
