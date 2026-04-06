# cf-spec

A CF starting point for generating component specs directly into Figma using an AI agent.

Based on [uSpec](https://github.com/redongreen/uSpec) by Ian Guisard at Uber. MIT licensed.

---

## What this is

cf-spec uses an AI agent (Claude Code) and the Figma MCP to automatically generate structured component documentation directly inside your Figma file.

You describe a component, the agent reads your Figma file, and renders a finished spec page in minutes.

It is not a plug-and-play tool. It requires a small amount of setup per project, but once configured it saves significant time on component documentation.

---

## What it generates

- **Anatomy** — numbered markers and attribute tables for every element
- **API** — properties, values, defaults, and configuration examples
- **Color** — design token mapping for every element and state
- **Structure** — dimensions, spacing, and padding across variants
- **Screen reader** — VoiceOver, TalkBack, and ARIA accessibility specs

---

## Prerequisites

Before using cf-spec on a project you need:

- Claude Code installed
- Figma Desktop (not browser)
- Figma MCP connected to Claude Code
- A Figma file with source components (named layers, defined variants, real tokens)

---

## How to set up a new project

1. Clone this repo to your machine
2. Duplicate `project-config-template.md` and rename it `project-config.md`
3. Fill in your project-specific details (design system name, token conventions, component taxonomy)
4. Open Figma Desktop and navigate to your component file
5. Open Claude Code in the cf-spec directory
6. Run your first spec using the prompt format in the template

---

## Project config

The `project-config.md` file is how you teach the agent about your specific design system. Without it, the agent will make assumptions that may not match your project.

See `project-config-template.md` for full instructions.

---

## Credits

Built on top of [uSpec](https://github.com/redongreen/uSpec) by Ian Guisard (Uber).
MIT License — see LICENSE file.
