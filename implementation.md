# uSpec Implementation

> **Purpose of this file:** This is an architecture reference for AI agents working in this codebase. It covers how the system connects (skills, MCP, Figma) — not the details of each spec type.
>
> **What belongs here:**
> - System architecture and data flow
> - Template registry pattern and extensibility
> - Cross-cutting utilities (cloning, error handling)
> - Build, test, and documentation setup
> - Reference file pointers
>
> **What does NOT belong here:**
> - Per-spec-type JSON schemas and examples — those live in each skill's instruction file (e.g., `screen-reader/agent-screenreader-instruction.md`)
> - Hardcoded template component keys — those are configured via `@setup-library` and stored in `uspecs.config.json`
>
> **Adding a new spec type:** Update the skills table, template type table, and reference files table below. Document the spec's schema, examples, and template structure in its own instruction file.

## Overview

uSpec generates documentation specifications for UI components. Most skills extract component data via MCP and render annotations directly in Figma using `figma_execute`. The motion skill is an exception — it reads pre-computed data from an After Effects export script rather than inspecting Figma components.

1. **Anatomy** - Numbered markers on a component instance with an attribute table
2. **Property** - Variant axes and boolean toggles with instance previews
3. **Screen Reader Specs** - Accessibility specifications for VoiceOver, TalkBack, and ARIA
4. **Color Annotation** - Design token specifications for component colors
5. **API Overview** - Component property documentation with configuration examples
6. **Structure Specification** - Dimensional properties documentation (spacing, padding, density variants)
7. **Changelog (create)** - Create a new changelog template with initial entries
8. **Changelog (update)** - Append new entries to an existing changelog
9. **Changelog (convert)** - Extract and convert an existing Figma changelog to structured JSON
10. **Motion Specification** - Animation timeline documentation from After Effects export data (pre-computed segments, no raw keyframes)

## Cursor Skills

Agent workflows can be triggered via Cursor skills in `.cursor/skills/`:

| Skill | Trigger Keywords | Purpose |
|-------|------------------|---------|
| `create-anatomy` | anatomy, component anatomy, create anatomy | Numbered markers and attribute table |
| `create-property` | property, properties, create property | Variant axes and boolean toggle exhibits |
| `create-voice` | voice, voiceover, screen reader, talkback, aria | Screen reader spec generation |
| `create-color` | color, color annotation, tokens | Color annotation generation |
| `create-api` | api, props, properties, component api | API overview generation |
| `create-structure` | structure, structure spec, dimensions, spacing, density, sizing | Structure spec generation |
| `create-changelog` | create changelog, new changelog, start changelog | Create new changelog with first entry |
| `update-changelog` | update changelog, add to changelog, changelog entry, log this change | Add entries to existing changelog |
| `convert-changelog` | convert changelog, import changelog, migrate changelog | Convert existing Figma changelog to JSON format |
| `create-motion` | motion, motion spec, animation spec, timeline | Motion specification from AE export (JSON paste, file ref, or Figma destination link) |
| `setup-library` | setup library, configure templates, link templates | Configure uSpec with your Figma template library |

**Usage:** Mention trigger keywords in your prompt (e.g., "Create voice spec for this button") or reference directly with `@create-voice`.

## Figma Console MCP Tools

Skills use the Figma Console MCP to gather component context when a Figma link is provided. The agent combines user-provided input (screenshots, descriptions) with MCP-retrieved data for maximum context.

For the latest tools and usage, see: https://docs.figma-console-mcp.southleft.com/tools

### Connection Verification

Before using MCP tools, verify the connection:
- `figma_get_status` — Confirms Figma Desktop is connected via the Desktop Bridge plugin

### Context Gathering Tools

| Tool | Purpose |
|------|---------|
| `figma_navigate` | Open a Figma URL to start monitoring |
| `figma_take_screenshot` | Capture visual of component and variants |
| `figma_get_file_data` | Get component structure, variant axes, properties |
| `figma_get_component` | Get detailed component metadata |
| `figma_get_component_for_development` | Get component data + visual reference in one call |
| `figma_get_variables` | Get variable collections and token definitions |
| `figma_get_token_values` | Get variable values organized by collection and mode |
| `figma_get_styles` | Get color, text, effect styles |
| `figma_get_design_system_summary` | Get overview of entire design system |
| `figma_search_components` | Find components by name |

### Tool Selection by Spec Type

| Spec Type | Key Tools |
|-----------|-----------|
| Anatomy / Property | `figma_execute` (extraction, template import, rendering), `figma_take_screenshot` (validation) |
| Screen Reader | `figma_take_screenshot`, `figma_get_file_data` (for states/variants), `figma_execute` (template import, rendering) |
| Color Annotation | `figma_get_variables`, `figma_get_token_values`, `figma_get_styles`, `figma_execute` (template import, rendering) |
| API Overview | `figma_get_file_data` (variant axes), `figma_get_component` (properties), `figma_execute` (template import, rendering) |
| Structure Spec | `figma_get_token_values`, `figma_execute` (for measurements, template import, rendering) |
| Motion Spec | `figma_execute` (template import, rendering), `figma_take_screenshot` (validation) |

## Architecture

```
Cursor Agent Skill  ──>  Figma Console MCP  ──>  Figma (via figma_execute)
```

```
Cursor              Figma MCP           Figma
   |                    |                  |
   |-- get context ---->|                  |
   |<-- component data -|                  |
   |                    |                  |
   |-- figma_execute ---|----------------->|
   |  (import template, |                  |-- render annotation
   |   create instances,|                  |-- place markers
   |   fill tables)     |                  |-- build exhibits
```

Most skills extract component data via MCP, then render annotations directly in Figma using `figma_execute`. Each skill imports its documentation template (by component key from `uspecs.config.json`), detaches it, and fills text fields, clones sections, and builds tables programmatically. The motion skill is different: its data comes from an After Effects export script (`motion/export-timeline.jsx`) that pre-computes segments, easing values, formatted labels, and `composition.durationMs`. Raw keyframes are stripped from the output — the JSON contains only segments. The agent passes segment data and `pxPerMs` to the Figma code, which computes bar positions at render time.

The anatomy and property skills share a single template (`anatomyOverview`); anatomy clones and fills its sections first, then property re-uses the same detached frame to build its own chapters. The voice, color, API, structure, changelog, and motion skills each have their own template and render independently.

### Template Keys

Template component keys are stored in `uspecs.config.json` and configured via `@setup-library`. Skills read the key for their template type and import it via `figma.importComponentByKeyAsync`:

| Config key | Template |
|------------|----------|
| `screenReader` | Screen reader spec |
| `colorAnnotation` | Color annotation |
| `anatomyOverview` | Anatomy annotation template |
| `apiOverview` | API overview |
| `structureSpec` | Structure specification |
| `propertyOverview` | Property overview |
| `changelog` | Changelog (create new) |
| `motionSpec` | Motion specification |

## Components

### Skills

All skills render directly in Figma via `figma_execute`, following a shared pattern:

1. **Extract** — Gather component data via MCP tools and AI reasoning (motion skill reads pre-computed data from AE export JSON instead)
2. **Import template** — `figma.importComponentByKeyAsync` with the skill's template key (from `uspecs.config.json`), create instance, detach
3. **Fill header** — Set component name, description, and header text
4. **Build content** — Clone template sections, fill text fields, build tables, create component instances where needed
5. **Validate** — `figma_take_screenshot` to verify output

**Template keys:** All template keys are stored in `uspecs.config.json` under the `templateKeys` object and configured via `@setup-library`. Each skill has its own template key.

**Variant matching (Anatomy/Property):** When creating component instances for a specific property value, the skill first attempts an exact match across all variant axes. If no exact match exists, it falls back to the best partial match.

**Marker positioning (Anatomy):** After placing the component instance in the artwork, the skill re-reads actual child positions from the instance using `absoluteTransform` rather than relying on extraction-time positions.

**Property normalization (Property):** Before rendering, `create-property` runs a 4-step pipeline to normalize raw Figma property definitions — resolving variant options, boolean toggles, instance-swap slots, and nested component properties into a uniform list of property axes.

**Clone visibility:** All cloned sections explicitly set `visible = true` after cloning, since template sources are hidden.

| Skill | Template | Sections Generated |
|-------|----------|-------------------|
| `create-anatomy` | Anatomy | Component structure with numbered markers and attribute table |
| `create-property` | Property | One chapter per variant axis (with instance previews) and per boolean toggle |
| `create-voice` | Screen reader | Focus order, per-state platform sections (VoiceOver, TalkBack, ARIA) with property tables |
| `create-color` | Color annotation | Per-variant sections with element-to-token mapping tables (Strategy A for ≤6 variants; Strategy B consolidates states into columns for >6) |
| `create-api` | API overview | Main property table, sub-component tables, configuration examples |
| `create-structure` | Structure spec | Per-section dimensional tables with dynamic columns for size/density variants |
| `create-changelog` | Changelog | Date entries with change items, bullet-formatted descriptions |
| `update-changelog` | Changelog (existing frame) | New date entry cloned and inserted at top of existing changelog |
| `convert-changelog` | — (reads existing frame) | Extracts an existing Figma changelog into structured JSON |
| `create-motion` | Motion spec | Timeline bars with easing-colored segments (bar positions computed in Figma code), detail table from pre-computed segments |

### Agent Instructions

Each spec type has its own instruction file that defines the agent's behavior, data schema, and examples. See the Reference Files table at the bottom.

### Template Infrastructure

Template component keys are stored in `uspecs.config.json` at the project root and configured via `@setup-library`. Each skill reads its key from this file and imports the template directly via `figma_execute` calling `figma.importComponentByKeyAsync`.

### Template Key Config

The template infrastructure uses a config file for extensibility:

**Template key config (`uspecs.config.json`):**

```json
{
  "templateKeys": {
    "screenReader": "key-from-setup-library",
    "colorAnnotation": "key-from-setup-library",
    "anatomyOverview": "key-from-setup-library",
    "apiOverview": "key-from-setup-library",
    "propertyOverview": "key-from-setup-library",
    "structureSpec": "key-from-setup-library",
    "changelog": "key-from-setup-library",
    "motionSpec": "key-from-setup-library"
  }
}
```

**Adding a new template type requires:**

1. Add a new key to `uspecs.config.json` under `templateKeys`
2. Create a new SKILL.md that reads the key and uses `figma.importComponentByKeyAsync` to import the template
3. Update `@setup-library` to search for and extract the new template's component key

## Cloning Logic

All skills follow a shared "clone from pristine template, fill, hide/remove original" pattern implemented inline within each `figma_execute` call:

1. **Find template** — Locate the hidden template node by name (e.g., `#section-template`, `#variant-template`, `#row-template`)
2. **Clone per data item** — For each item in the data array, call `template.clone()` and append the clone to the template's parent
3. **Set visible** — Each clone sets `visible = true` (templates are hidden by default)
4. **Fill content** — Load fonts, set text fields, configure properties on each clone
5. **Remove or hide original** — After all clones are created, either `template.remove()` (for row-level templates) or `template.visible = false` (for section-level templates)

This pattern nests at multiple levels. For example, the screen reader skill clones state templates, then within each state clones platform section templates, then within each section clones table templates, then within each table clones row templates. Each nesting level follows the same clone-fill-remove pattern within a single `figma_execute` call.

## Stability

Each skill splits work across multiple `figma_execute` calls to avoid timeouts — typically one call per section, variant, or state. This keeps each call's execution time short and lets Figma process between calls. Complex specs (e.g., structure with many sections, screen reader with many states) benefit most from this pattern.

## Documentation Site

The uSpec docs are hosted at **https://docs.uspec.design** using [Mintlify](https://mintlify.com).

### Updating docs

1. Reference `@mintlify.mdc` in your Cursor prompt so the agent uses the correct writing style and Mintlify components
2. Edit the MDX files in `docs/` (and update `docs/docs.json` if adding or removing pages)
3. Push to `main` — Mintlify auto-deploys within 1–2 minutes

### Docs file structure

```
docs/
├── docs.json              # Site config: theme, navigation, colors, metadata
├── index.mdx              # Homepage
├── getting-started.mdx    # Setup and first spec guide
├── how-it-works.mdx       # System overview
├── specs/                 # Specification type docs
├── help/                  # Troubleshooting, contribute, changelog
├── images/                # Screenshots, videos, and spec output demos
├── logo/                  # Logo files (light.svg, dark.svg)
└── favicon.svg
```

### Key references

- **Writing rule**: `.cursor/rules/mintlify.mdc` — component syntax, writing style, content standards
- **Mintlify MCP**: `SearchMintlify` tool — look up component syntax, configuration, and best practices
- **Local preview**: `npx mintlify dev` from `docs/` directory (port 3000)
- **Site config schema**: https://mintlify.com/docs.json

## Reference Files

### Skills

| File | Content |
|------|---------|
| `.cursor/skills/create-anatomy/SKILL.md` | Anatomy: extraction, marker rendering, attribute table |
| `.cursor/skills/create-property/SKILL.md` | Property: variant axis and boolean toggle exhibits |
| `.cursor/skills/create-voice/SKILL.md` | Screen reader: merge analysis, platform sections, property tables |
| `.cursor/skills/create-color/SKILL.md` | Color: variant sections, element-to-token mapping tables |
| `.cursor/skills/create-api/SKILL.md` | API: main table, sub-component tables, configuration examples |
| `.cursor/skills/create-structure/SKILL.md` | Structure: dynamic columns, hierarchy indicators, dimensional tables |
| `.cursor/skills/create-changelog/SKILL.md` | Changelog: template import, entry cloning, bullet formatting |
| `.cursor/skills/update-changelog/SKILL.md` | Changelog update: clone entry into existing frame |
| `.cursor/skills/convert-changelog/SKILL.md` | Changelog convert: extract existing Figma changelog to JSON |
| `.cursor/skills/create-motion/SKILL.md` | Motion: timeline bars, pre-computed easing segments, detail table |
| `.cursor/skills/setup-library/SKILL.md` | Setup: scan template library, extract component keys |

### Agent Instruction Files

| File | Content |
|------|---------|
| `anatomy/agent-anatomy-instruction.md` | Anatomy annotation: element classification rules, note-writing guidelines, property-aware unhide decisions |
| `screen-reader/agent-screenreader-instruction.md` | Screen reader spec: data schema, examples, agent behavior |
| `screen-reader/voiceover.md` | iOS accessibility properties reference |
| `screen-reader/talkback.md` | Android semantics and roles reference |
| `screen-reader/aria.md` | ARIA roles and states reference |
| `color/agent-color-instruction.md` | Color annotation: data schema, examples, agent behavior |
| `api/agent-api-instruction.md` | API overview: data schema, examples, agent behavior |
| `api/api-library.md` | API documentation reference patterns |
| `structure/agent-structure-instruction.md` | Structure spec: data schema, examples, agent behavior |
| `changelog/agent-changelog-instruction.md` | Changelog: writing style, schema, validation rules |
| `motion/agent-motion-instruction.md` | Motion spec: JSON schema (with pre-computed segments), rendering rules, timeline layout |
| `motion/export-timeline.jsx` | After Effects export script: keyframe extraction, segment computation, cubic-bezier conversion, value formatting, keyframe stripping |
