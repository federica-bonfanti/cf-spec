---
name: firstrun
description: First-time setup for uSpec. Prompts for MCP provider and environment (Cursor, Claude Code, Codex), syncs skills to the chosen platform, then configures your Figma template library. Use when the user mentions "firstrun", "first run", "setup", "setup library", "configure templates", or "link templates".
---

# First Run

Set up uSpec for your environment. This skill asks which Figma MCP you're using, which platform you're on, deploys skills to the right directory, then extracts template component keys from your Figma library and writes the configuration.

## Inputs Expected

This skill collects inputs interactively — do not require them up front.

## Workflow

Copy this checklist and update as you progress:

```
Task Progress:
- [ ] Step 1: Select MCP provider
- [ ] Step 1b: Verify MCP connection
- [ ] Step 2: Select environment
- [ ] Step 3: Sync skills (if needed)
- [ ] Step 4: Get library link
- [ ] Step 5: Navigate to the library file / extract fileKey
- [ ] Step 6: Search for template components
- [ ] Step 7: Extract component keys
- [ ] Step 7b: Detect font family from template
- [ ] Step 8: Write config to uspecs.config.json
- [ ] Step 9: Display success message
```

### Step 1: Select MCP Provider

Ask the user:

> **Which Figma MCP are you using?**
> 1. Figma Console MCP (Southleft) — requires Desktop Bridge plugin
> 2. Figma MCP (Native) — official Figma MCP with write access

Wait for the user's answer before proceeding. Save the choice as `MCP_PROVIDER` (one of `figma-console` or `figma-mcp`).

### Step 1b: Verify MCP Connection

Verify the selected MCP is connected before continuing — catching a broken connection early avoids wasting time on setup that will fail later.

**If `MCP_PROVIDER` = `figma-console`:**
- Call `figma_get_status` — Confirm Desktop Bridge plugin is active

If connection fails:
> Please open Figma Desktop and run the Desktop Bridge plugin. Then try again.

**If `MCP_PROVIDER` = `figma-mcp`:**
- Make a lightweight `use_figma` call to verify connectivity:
  - `fileKey`: any valid fileKey (you can use `"test"` — the call will fail with a clear error if the MCP itself is not connected, vs. a file-not-found error which confirms the MCP works)
  - `code`: `return "ok";`
  - `description`: `Verify MCP connection`

If the MCP itself is not reachable (tool not found, server error):
> The native Figma MCP is not responding. Please check your MCP configuration and ensure the Figma MCP server is running.

If the call returns a file-not-found error, that's fine — it means the MCP is connected. Proceed.

### Step 2: Select Environment

Ask the user:

> **Which tool are you configuring uSpec for?**
> 1. Cursor
> 2. Claude Code CLI
> 3. Codex CLI

Wait for the user's answer before proceeding. Save the choice as `ENVIRONMENT` (one of `cursor`, `claude-code`, `codex`).

### Step 3: Sync Skills (if needed)

Based on the environment selected in Step 2:

- **Cursor** — Skip this step. All skills are already in `.cursor/skills/`.
- **Claude Code CLI** — Run the sync script to deploy skills:
  ```bash
  ./utils/sync-skills.sh --target claude
  ```
- **Codex CLI** — Run the sync script to deploy skills:
  ```bash
  ./utils/sync-skills.sh --target codex
  ```

If the sync script fails, guide the user:
> The skill sync failed. Make sure you're running from the uSpec project root and that `utils/sync-skills.sh` is executable (`chmod +x utils/sync-skills.sh`).

### Step 4: Get Library Link

Ask the user:

> **Paste the link to your Figma template library file.**
> Uber designers can skip this — type "skip" to use the internal library.

Wait for the user's answer. Save the URL as `LIBRARY_URL`.

If the user types "skip", use the pre-configured internal library URL (if one exists in `uspecs.config.json`). If no pre-configured URL exists, tell the user:
> No internal library URL is configured. Please provide a Figma link to your template library.

**For `figma-mcp` only:** Extract `FILE_KEY` from the URL. Figma URLs follow the pattern `figma.com/design/:fileKey/:fileName`. For branch URLs (`figma.com/design/:fileKey/branch/:branchKey/:fileName`), use `:branchKey` as the `FILE_KEY`. Save this for all subsequent `use_figma` / `search_design_system` / `get_screenshot` calls.

### Step 5: Navigate to the Library File

**If `MCP_PROVIDER` = `figma-console`:**
- `figma_navigate` — Open the template library URL

**If `MCP_PROVIDER` = `figma-mcp`:**
- No navigation call needed — `use_figma` takes `fileKey` directly. The `FILE_KEY` extracted in Step 4 is used for all subsequent calls. Optionally, verify the file is accessible:
  - `use_figma` with `fileKey = FILE_KEY`, `code = "return figma.root.children.map(p => p.name);"`, `description = "List pages in template library"`

### Step 6: Search for Template Components

Required template names (case-insensitive search):
1. "Screen reader"
2. "Color Annotation"
3. "Anatomy"
4. "API"
5. "Property"
6. "Structure"
7. "Motion"

**If `MCP_PROVIDER` = `figma-console`:**
- `figma_search_components` with query for each template name

**If `MCP_PROVIDER` = `figma-mcp`:**
- `search_design_system` with `query` for each template name, `fileKey = FILE_KEY`, and `includeComponents: true`

### Step 7: Extract Component Keys

For each found component, extract its component key. The search results include the `componentKey` (Console MCP) or `key` (native MCP) field.

Build a mapping of template type to key:
- screenReader: key from "Screen reader" component
- colorAnnotation: key from "Color Annotation" component
- anatomyOverview: key from "Anatomy" component
- apiOverview: key from "API" component
- propertyOverview: key from "Property" component
- structureSpec: key from "Structure" component
- motionSpec: key from "Motion" component

If any template is not found, report which ones are missing:
> Could not find the following templates: [list]. Please ensure your library file contains components with these exact names.

### Step 7b: Detect Font Family from Template

Using the node ID of one of the found template components (e.g., the Overview or API component):

**If `MCP_PROVIDER` = `figma-console`:**
- Use `figma_execute` to run the font detection script below.

**If `MCP_PROVIDER` = `figma-mcp`:**
- Use `use_figma` with `fileKey = FILE_KEY`, `description = "Detect font family from template"`, and the same script below as `code`.

```javascript
const node = await figma.getNodeByIdAsync('NODE_ID_FROM_STEP_6');
const textNode = node.findOne(n => n.type === 'TEXT');
if (textNode) {
  return textNode.fontName.family;
} else {
  return 'Inter';
}
```

Save the result as `DETECTED_FONT_FAMILY`. If the script returns an error or no text node is found, default to `Inter`.

### Step 8: Write Config to uspecs.config.json

Write the extracted keys, detected font family, MCP provider, and environment to `uspecs.config.json` at the project root. The file structure is:

```json
{
  "mcpProvider": "MCP_PROVIDER",
  "environment": "ENVIRONMENT",
  "fontFamily": "DETECTED_FONT_FAMILY",
  "templateKeys": {
    "screenReader": "KEY_FROM_STEP_7",
    "colorAnnotation": "KEY_FROM_STEP_7",
    "anatomyOverview": "KEY_FROM_STEP_7",
    "apiOverview": "KEY_FROM_STEP_7",
    "propertyOverview": "KEY_FROM_STEP_7",
    "structureSpec": "KEY_FROM_STEP_7",
    "motionSpec": "KEY_FROM_STEP_7"
  }
}
```

Replace `MCP_PROVIDER` with the value from Step 1, `ENVIRONMENT` with the value from Step 2, `DETECTED_FONT_FAMILY` with the font detected in Step 7b, and each template key with the actual component key from Step 7.

### Step 9: Success Message

Display:

> **Setup complete!**
>
> You are now ready to use uSpec. For instructions, go to [docs.uspec.design](https://docs.uspec.design).
