# MoonShine v4 Skills

Curated knowledge base for [MoonShine v4](https://moonshine-laravel.com/) — the Laravel admin panel framework. Designed for AI coding agents that support context file loading.

**42 files | 18,100+ lines | 8 skill modules** covering setup, resources, fields, components, appearance, frontend, security, and advanced patterns.

## What Is This?

A structured collection of markdown files that give AI coding assistants accurate, up-to-date knowledge of the MoonShine v4 framework. Instead of relying on outdated training data or hallucinating APIs, the agent reads from these curated references on demand.

Works with any AI coding tool that can load markdown files as context. The files are plain markdown — no proprietary format, no vendor lock-in.

> Looking for MoonShine v3? See [moonshine-v3-skills](https://github.com/agarzon/moonshine-v3-skills).

## Skills

| Skill | Description | Files | Lines |
|-------|-------------|:-----:|------:|
| **moonshine-setup-v4** | Installation, configuration, routing, middleware, localization, artisan commands | 3 | 1,441 |
| **moonshine-resources-v4** | ModelResource CRUD, pages, filters, search, query tags, events, buttons | 4 | 1,704 |
| **moonshine-fields-v4** | All field types, relationships, validation, lifecycle, custom fields | 8 | 3,376 |
| **moonshine-components-v4** | FormBuilder, TableBuilder, layout components, overlays, display components | 5 | 2,304 |
| **moonshine-appearance-v4** | Layouts, menus, colors, icons, assets, dark mode, branding | 6 | 2,787 |
| **moonshine-frontend-v4** | API mode, JWT auth, SDUI, Alpine.js events, async UI, reactive fields | 4 | 1,939 |
| **moonshine-security-v4** | Authentication, authorization, policies, 2FA, socialite, IP restrictions | 2 | 1,016 |
| **moonshine-advanced-v4** | Controllers, handlers, routes, type casts, testing, recipes, packages | 9 | 3,551 |

## Usage

Clone the repo and point your AI coding tool to it as a skill:

```bash
git clone https://github.com/agarzon/moonshine-v4-skills.git .claude/skills/moonshine-v4
```

`.claude/settings.json`:

```json
{
  "skills": [
    ".claude/skills/moonshine-v4"
  ]
}
```

The root `SKILL.md` acts as a **router** that automatically directs the agent to load the relevant sub-skill(s) based on the task.

> **Individual registration:** If you prefer loading specific modules only, register sub-skills individually (e.g., `.claude/skills/moonshine-v4/moonshine-fields-v4`).

## How It Works

The root `SKILL.md` gives the agent a brief MoonShine v4 overview and a routing table that maps task keywords to the appropriate sub-skill(s). The agent then reads 1-3 relevant sub-skills on demand, keeping context usage efficient.

```
SKILL.md                                # Router — single entry point (~60 lines)
moonshine-fields-v4/
  SKILL.md                              # Sub-skill entry point (~230-500 lines)
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

moonshine-setup-v4/
  SKILL.md
  references/
    config-options.md
    config-auth-localization.md

moonshine-resources-v4/
  SKILL.md
  references/
    crud-pages.md
    filters-search-querytags.md
    events-buttons.md

moonshine-fields-v4/
  SKILL.md
  references/
    basic-fields.md
    selection-fields.md
    file-fields.md
    relationship-fields.md
    field-display-attributes.md
    field-values-lifecycle.md
    field-interactive.md

moonshine-components-v4/
  SKILL.md
  references/
    form-table-builders.md
    layout-components.md
    overlay-components.md
    display-components.md

moonshine-appearance-v4/
  SKILL.md
  references/
    layouts.md
    menu-system.md
    colors.md
    icons.md
    assets-branding.md

moonshine-frontend-v4/
  SKILL.md
  references/
    api-jwt.md
    js-events-alpine.md
    sdui.md

moonshine-security-v4/
  SKILL.md
  references/
    auth-extensions.md

moonshine-advanced-v4/
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

## Key v3 to v4 Changes

- **Page-centric architecture**: IndexPage, FormPage, DetailPage as first-class citizens
- **Namespace moves**: Forms/JsonResponse from `MoonShine\Laravel` to `MoonShine\Crud`, Enums to `MoonShine\Support`
- **New features**: `#[AsyncMethod]` attribute, query tags, response modifiers, AsyncCallback, OpenAPI generator
- **UI changes**: PurplePalette default, StackFields replaced by Fieldset, CompactLayout removed
- **Config**: MenuItem parameter swap ($filler first), `authorizationRules()` closure signature updated

## Contributing

Contributions welcome. When editing:

- Keep SKILL.md files under 500 lines (these are always loaded into context)
- Keep reference files under 600 lines (split if larger)
- Preserve all code examples — they should be copy-paste ready

## License

MIT
