# Planning notes: CivicTheme v1.12.1 → v1.12.2

This scratchpad exists for quick notes while running the upgrade in this
directory. The authoritative instructions remain in `spec.md`, `tasks.md`, and
`playbook.md`.

## Pre-upgrade capture: custom library attachments (T405)

Use this table to record custom sub-theme libraries and where they are
attached before making changes. It supports T405 and later restoration in
T415.

| Library | Files referenced | Where attached (Twig/preprocess) | Notes |
|---------|------------------|-----------------------------------|-------|
|         |                  |                                   |       |

## Pre-upgrade capture: `|raw` filter occurrences (T401)

Document all `|raw` filter usages found in your sub-theme templates.

| File | Line | Variable/Expression | Action taken |
|------|------|---------------------|--------------|
|      |      |                     |              |

## Pre-upgrade capture: preprocessing functions (T402)

Document preprocessing functions that return HTML strings and need Markup
object conversion.

| File | Function | Variable | Current pattern | Updated pattern |
|------|----------|----------|-----------------|-----------------|
|      |          |          |                 |                 |

## Pre-upgrade capture: overridden components (T403)

Document CivicTheme components that have been overridden in your sub-theme.

| Component | Sub-theme path | Has `\|raw`? | Needs update? | Notes |
|-----------|----------------|--------------|---------------|-------|
|           |                |              |               |       |

## Component comparison notes

Use this section to document differences between your overridden templates
and the upstream 1.12.2 versions.

### Component: [name]

**Differences found**:
-

**Action taken**:
-

## Other scratch notes

-

