# MoonShine v3 Skills

Curated knowledge base for [MoonShine v3](https://moonshine-laravel.com/) — the Laravel admin panel framework. Designed for AI coding agents that support context file loading.

**41 files | 14,400+ lines | 8 skill modules** covering setup, resources, fields, components, appearance, frontend, security, and advanced patterns.

## What Is This?

A structured collection of markdown files that give AI coding assistants accurate, up-to-date knowledge of the MoonShine v3 framework. Instead of relying on outdated training data or hallucinating APIs, the agent reads from these curated references on demand.

Works with any AI coding tool that can load markdown files as context. The files are plain markdown — no proprietary format, no vendor lock-in.

> Looking for MoonShine v4? See [moonshine-v4-skills](https://github.com/agarzon/moonshine-v4-skills).

## Skills

| Skill | Description | Files | Lines |
|-------|-------------|:-----:|------:|
| **moonshine-setup-v3** | Installation, configuration, routing, middleware, localization, artisan commands | 3 | 1,016 |
| **moonshine-resources-v3** | ModelResource CRUD, pages, filters, search, events, buttons, import/export | 4 | 1,802 |
| **moonshine-fields-v3** | All field types, relationships, validation, lifecycle, custom fields | 8 | 2,459 |
| **moonshine-components-v3** | FormBuilder, TableBuilder, layout components, overlays, display components | 5 | 2,509 |
| **moonshine-appearance-v3** | Layouts, menus, colors, icons, assets, dark mode, branding | 6 | 2,249 |
| **moonshine-frontend-v3** | API mode, JWT auth, SDUI, Alpine.js events, async UI, reactive fields | 4 | 1,323 |
| **moonshine-security-v3** | Authentication, authorization, policies, 2FA, socialite, IP restrictions | 2 | 933 |
| **moonshine-advanced-v3** | Controllers, handlers, routes, type casts, testing, recipes, packages | 9 | 2,162 |

## Usage

Clone the repo and point your AI coding tool to it as a skill:

```bash
git clone https://github.com/agarzon/moonshine-v3-skills.git .claude/skills/moonshine-v3
```

`.claude/settings.json`:

```json
{
  "skills": [
    ".claude/skills/moonshine-v3"
  ]
}
```

The root `SKILL.md` acts as a **router** that automatically directs the agent to load the relevant sub-skill(s) based on the task.

> **Individual registration:** If you prefer loading specific modules only, register sub-skills individually (e.g., `.claude/skills/moonshine-v3/moonshine-fields-v3`).

## How It Works

The root `SKILL.md` gives the agent a brief MoonShine v3 overview and a routing table that maps task keywords to the appropriate sub-skill(s). The agent then reads 1-3 relevant sub-skills on demand, keeping context usage efficient.

```
SKILL.md                                # Router — single entry point (~60 lines)
moonshine-fields-v3/
  SKILL.md                              # Sub-skill entry point (~250-490 lines)
  references/
    basic-fields.md                     # Detailed API reference
    selection-fields.md                 # (loaded on demand for deeper context)
    ...
```

- **SKILL.md** at the root is the router — load this first.
- **Sub-skill SKILL.md** files are self-contained entry points with code examples and cross-references.
- **references/** contains detailed API references. Load these when deeper context is needed.
- SKILL.md files are kept under 500 lines; reference files under 600 lines each.

## File Structure

```
SKILL.md                                # Router

moonshine-setup-v3/
  SKILL.md
  references/
    config-options.md
    config-auth-localization.md

moonshine-resources-v3/
  SKILL.md
  references/
    crud-pages.md
    filters-search.md
    events-buttons.md

moonshine-fields-v3/
  SKILL.md
  references/
    basic-fields.md
    selection-fields.md
    file-fields.md
    relationship-fields.md
    field-display-attributes.md
    field-values-lifecycle.md
    field-interactive.md

moonshine-components-v3/
  SKILL.md
  references/
    form-table-builders.md
    layout-components.md
    overlay-components.md
    display-components.md

moonshine-appearance-v3/
  SKILL.md
  references/
    layouts.md
    menu-system.md
    colors.md
    icons.md
    assets-branding.md

moonshine-frontend-v3/
  SKILL.md
  references/
    api-jwt.md
    sdui.md
    js-events-alpine.md

moonshine-security-v3/
  SKILL.md
  references/
    auth-extensions.md

moonshine-advanced-v3/
  SKILL.md
  references/
    controllers-routes.md
    handlers.md
    typecasts-packages.md
    testing.md
    recipes-dashboard.md
    recipes-resources.md
    recipes-forms-tables.md
    recipes-ui-other.md
```

## Contributing

Contributions welcome. When editing:

- Keep SKILL.md files under 500 lines (these are always loaded into context)
- Keep reference files under 600 lines (split if larger)
- Preserve all code examples — they should be copy-paste ready

## License

MIT
