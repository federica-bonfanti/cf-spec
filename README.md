# cf-spec

A CF tool for generating component specs directly into Figma — using an AI agent instead of doing it manually.

Based on [uSpec](https://github.com/redongreen/uSpec) by Ian Guisard at Uber. MIT licensed.

---

## What this is

Writing component specs in Figma is slow. Anatomy tables, token annotations, property lists, accessibility notes — for a single component it can take hours. Across a full design system it becomes a project in itself.

cf-spec automates that process. You tell the agent about your design system once, point it at a component in Figma, and it generates a finished spec frame in minutes — using your actual layer names, your actual tokens, and your actual variants.

You still review and refine the output. The agent handles the extraction and the formatting.

---

## Before you start — a few reassurances

**You do not need to be a developer to use this.**
The setup requires a few one-time steps that might feel unfamiliar. Each one is explained below in plain language. If you get stuck, ask a developer to help with the initial setup — it takes them about 15 minutes.

**You are not going to break anything.**
This tool only creates new frames in your Figma file. It does not delete, move, or modify any of your existing components or pages.

**This is just a folder on your computer.**
When you download cf-spec, you get a folder of files. Think of it like a Figma plugin that lives on your machine instead of inside Figma. The most important file in that folder is the one you fill in — `project-config.md`.

**The agent is just Claude.**
When cf-spec runs, it uses Claude Code — Anthropic's AI assistant — to read your Figma file and write the spec. You are having a conversation with it, just through a terminal window instead of a chat window.

---

## What it generates

- **Anatomy** — numbered markers and attribute tables identifying every element
- **API** — all component properties, their values, and defaults
- **Color** — design token mapping for every element and state
- **Structure** — dimensions, spacing, and padding across size variants
- **Screen reader** — VoiceOver, TalkBack, and ARIA accessibility specs

---

## What you need before you start

These are one-time setup steps. Once done, they do not need to be repeated for future projects.

**1. Claude Code**
This is the AI agent that does the work. It runs in Terminal — the black window with text on your Mac.
→ [How to install Claude Code](https://docs.anthropic.com/en/docs/claude-code)

**2. Figma Desktop**
The desktop app, not the browser version. The agent connects to Figma through a local connection that only works in the desktop app.
→ Download from [figma.com/downloads](https://www.figma.com/downloads)

**3. Figma MCP connected to Claude Code**
MCP is the bridge that lets Claude Code talk to your Figma file. Think of it as a cable between the two apps — without it the agent cannot read or write to Figma. Ask a developer to help with this step if needed. It takes about 10 minutes.

**4. A Figma file with source components**
cf-spec works best when your components are well structured — meaningful layer names, clearly defined variants, and colours and spacing using tokens rather than hardcoded values. The better your component is built, the better the spec output will be.

---

## How to set up a new project

**Step 1 — Download the cf-spec folder onto your machine**

Click the green **Code** button at the top of this GitHub page, then select **Download ZIP**. Unzip it somewhere easy to find, like your Documents folder. That is it — you now have cf-spec on your machine!

**Step 2 — Create your project config file**

Inside the cf-spec folder, find the file called project-config-template.md. Duplicate it and rename the copy to project-config.md.
This is the most important thing you will do. It teaches the agent about your project — your design system, your audience, how detailed you want the output, and how it should look. Without it the agent will make assumptions that may not match your project.

 ** The easiest way to fill it in is with Claude's help.**
  Open a conversation with Claude at claude.ai and paste this:
  "I am setting up cf-spec to generate component specs for a Figma design system.
  Please help me fill in my project-config.md by asking me one section at a time.
  The sections are: project basics, audience, depth, sections, look and feel,
  token naming conventions, component taxonomy, typography, and example components.
  Ask me questions for each section and then give me the completed text to paste in."
  
  Claude will guide you through each section, ask the right questions, and give you the completed text to paste directly into your project-config.md. It takes about 15 minutes and you only need to do it once per project.

**Step 3 — Open Figma Desktop**

Open the Figma desktop app and navigate to your component library file. Make sure it is open — the agent connects to whatever file is currently active.

**Step 4 — Open Terminal and navigate to the cf-spec folder**

Open Terminal (press Cmd+Space and type Terminal). Type the following, replacing the path with wherever you saved cf-spec:
```bash
cd /path/to/cf-spec
```

Then type:
```bash
claude
```

You are now talking to the agent.

**Step 5 — Run your first spec**

Paste this prompt into Claude Code, replacing the bracketed parts with your own details:
Read /path/to/cf-spec/project-config.md for design system context.
Then generate a component anatomy spec for this Figma component: [paste your Figma component link here]
Use the component's actual layer names and token values.
Create a new spec frame in the same Figma file called "[Component name] — Anatomy Spec".

Watch your Figma file — the spec frame will appear directly on the canvas.

---

## The project config file

`project-config.md` is where you define the rules for your project. It covers:

- **Who the spec is for** — developer handoff, designer reference, or both
- **How detailed it should be** — essential, standard, or comprehensive
- **Which sections to generate** — anatomy, API, colour, structure, accessibility
- **How it should look** — font, colours, and frame width for the spec output
- **Your design system** — token format, component taxonomy, typography scale

Without this file the agent will still generate specs, but it will make assumptions that may not match your project. Filling it in takes about 15–20 minutes and makes every spec after that significantly better.

See `project-config-template.md` for the full template with instructions.

---

## What to expect from the output

cf-spec produces a strong first draft, not a finished spec. In testing:

- Token names and layer structure are extracted accurately
- Output formatting sometimes needs manual cleanup
- Simple components may be faster to spec manually
- Complex, content-rich components benefit most from the automation

Review the output, refine what needs refining, and treat it as your starting point rather than your final deliverable.

---

## Credits

Built on top of [uSpec](https://github.com/redongreen/uSpec) by Ian Guisard at Uber. MIT License — see LICENSE file.

This is a CF internal tool, not officially supported by the uSpec project. For issues specific to cf-spec, raise them within CF. For issues with the underlying engine, refer to the [uSpec repository](https://github.com/redongreen/uSpec).
