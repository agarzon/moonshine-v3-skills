---
name: moonshine-v3
description: Use when working on any MoonShine v3 admin panel task — setup, resources, fields, components, appearance, frontend, security, or advanced patterns. This is the single entry point that routes to specialized sub-skills.
---

# MoonShine v3 — Skill Router

MoonShine v3 is a Laravel admin panel framework for building back-office applications. It provides:

- **ModelResource** — CRUD scaffolding with pages, filters, search, and actions
- **Fields** — 30+ field types including relationships, files, JSON, and custom fields
- **Components** — FormBuilder, TableBuilder, layout primitives, modals, metrics
- **Layouts & Theming** — Menu system, colors, icons, dark mode, custom assets
- **Frontend** — API mode with JWT, SDUI (Server-Driven UI), Alpine.js integration
- **Security** — Authentication, authorization, policies, 2FA, socialite

## How to Use This Skill

Based on what the task involves, read **1–3 relevant sub-skills** from the table below. Each sub-skill is a self-contained `SKILL.md` with code examples and references to deeper documentation.

## Routing Table

| Task involves... | Read this sub-skill |
|---|---|
| Installation, configuration, bootstrapping, routing, middleware, localization, artisan commands, project structure, IDE setup, replacing default pages/forms | `moonshine-setup-v3/SKILL.md` |
| ModelResource, CRUD operations, tables, forms, detail pages, filters, search, pagination, events lifecycle, buttons, query modification, import/export, metrics, modal CRUD, redirects, active actions | `moonshine-resources-v3/SKILL.md` |
| Field types, relationship fields (BelongsTo, HasMany, etc.), field validation, field lifecycle, apply logic, field modes (default/preview/raw), creating custom fields | `moonshine-fields-v3/SKILL.md` |
| FormBuilder, TableBuilder, ActionButton, layout components, grid/columns, modals, tabs, cards, metrics, overlays, display components, JS events, component combinations | `moonshine-components-v3/SKILL.md` |
| Layouts, custom layouts, menus, colors, icons, assets, custom pages, dark mode, branding, Blade templates, admin panel design | `moonshine-appearance-v3/SKILL.md` |
| API mode/backend, JWT authentication for API, SDUI (Server-Driven UI), Alpine.js events, JavaScript helpers, reactive fields, async UI updates, fragment-based partial updates, custom Alpine.js components | `moonshine-frontend-v3/SKILL.md` |
| Authentication, authorization, login, guards, custom user models, Socialite, 2FA, JWT, role-based access, Laravel policies, auth pipelines, middleware, IP restrictions | `moonshine-security-v3/SKILL.md` |
| Custom controllers, handlers, custom routes, type casts, notifications, toasts, testing, package development, CrudResource (non-Eloquent data), MoonShineJsonResponse, recipe patterns | `moonshine-advanced-v3/SKILL.md` |

## Instructions

1. **Identify** which areas the task touches (usually 1–3 from the table above)
2. **Read** the corresponding `SKILL.md` file(s) using the Read tool — resolve paths relative to this file's directory
3. **Follow** the patterns and examples in those sub-skills
4. If deeper detail is needed, each sub-skill cross-references files in its `references/` folder

## Quick Reference: Common Task Mappings

**"Create a new resource"** → setup + resources + fields

**"Add a custom page"** → resources + components

**"Build a dashboard"** → components + appearance + advanced (recipes)

**"Set up auth/permissions"** → security + setup

**"Add API endpoints"** → frontend + resources

**"Customize the theme"** → appearance

**"Add a custom field"** → fields + components

**"Write tests"** → advanced
