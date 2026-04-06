# project-config-template.md

Copy this file, rename it `project-config.md`, and fill in your project details.
The agent will read this file before generating any spec. The more accurate your
answers, the better the output.

---

## 1. Project basics

**Project name:**
<!-- e.g. MSDAH, Aviva, internal CF design system -->

**Design system name:**
<!-- e.g. Base, Mosaic, Aurora — the name used in Figma and tokens -->

**Figma file URL:**
<!-- Paste the URL of your main component library file -->

---

## 2. Token naming conventions

**Token format:**
<!-- Describe how your tokens are named. Examples:
- components/button/global/colorprimary
- color.brand.primary
- --cf-color-primary -->

**Token categories used:**
<!-- List which token types exist in your system. Examples:
- Color tokens
- Spacing tokens
- Typography tokens
- Radius tokens
- Shadow tokens -->

---

## 3. Component taxonomy

**List your main component categories:**
<!-- e.g. Actions (Button, CTA, Link), Inputs (Text field, Dropdown, Checkbox),
Feedback (Toast, Alert, Modal), Layout (Card, Grid, Divider) -->

**Variant naming conventions:**
<!-- How are variants named in your Figma components? Examples:
- Status=Default, Status=Hover, Status=Disabled
- Type=Primary, Type=Secondary
- Size=SM, Size=MD, Size=LG -->

**Boolean property naming:**
<!-- How are boolean toggles named? Examples:
- Has Icon=True/False
- Show Label=True/False -->

---

## 4. Typography

**Font families used:**
<!-- e.g. Invention, Inter, Söhne -->

**Type scale names:**
<!-- List your named text styles as they appear in Figma. Examples:
- Heading/1, Heading/2, Heading/3, Heading/4
- Body/LG, Body/MD, Body/SM
- Label/Normal, Label/Strong -->

---

## 5. Spec preferences

**Which spec types do you need?**
<!-- Delete the ones you don't need for this project -->
- [ ] Anatomy
- [ ] API
- [ ] Color annotation
- [ ] Structure
- [ ] Screen reader
- [ ] Motion

**Output language:**
<!-- e.g. English, French, German — affects screen reader spec terminology -->
