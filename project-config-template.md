# project-config-template.md

Copy this file, rename it `project-config.md`, and fill in your project details.
The agent reads this file before generating any spec. The more accurate your
answers, the better the output.

---

## 1. Project basics

**Project name:**
<!-- e.g. MSDAH, Aviva, internal CF design system -->

**Design system name:**
<!-- e.g. Base, Mosaic, Aurora -->

**Figma file URL:**
<!-- Paste the URL of your main component library file -->

---

## 2. Audience

Who is this spec primarily written for?

**Primary audience:**
<!-- Choose one: Developer handoff / Designer reference / Both -->

**If developer handoff:**
<!-- - Should node IDs be included in attribute tables? Yes / No -->
<!-- - Should token hex values be shown alongside token names? Yes / No -->
<!-- - Should spacing and sizing values be shown in px and rem? Yes / No -->

**If designer reference:**
<!-- - Should do's and don'ts be included? Yes / No -->
<!-- - Should usage guidance be included per component? Yes / No -->
<!-- - Should state examples be shown in the preview? Yes / No -->

---

## 3. Depth

How detailed should each spec be?

**Depth level:**
<!-- Choose one: -->
<!-- Essential — anatomy and tokens only. Fast, minimal output. Good for simple components. -->
<!-- Standard — anatomy, tokens, properties, and states. Recommended default. -->
<!-- Comprehensive — all of the above plus accessibility, do's and don'ts, and usage notes. For complex or high-risk components. -->

**Override per component:**
<!-- You can override depth for specific components here if needed -->
<!-- e.g. Button=Essential, DataTable=Comprehensive -->

---

## 4. Sections

Which spec types do you need for this project?
Delete the ones you don't need.

- [ ] Anatomy — numbered markers and attribute tables
- [ ] API — properties, values, defaults, and configuration examples
- [ ] Color annotation — token mapping for every element and state
- [ ] Structure — dimensions, spacing, and padding across variants
- [ ] Screen reader — VoiceOver, TalkBack, and ARIA accessibility specs
- [ ] Motion — animation timeline and easing details

---

## 5. Look and feel

Controls the visual style of the generated spec frames in Figma.

**Font family:**
<!-- The font used for all text in spec frames -->
<!-- e.g. Inter, Söhne, Helvetica Neue -->
<!-- Default: Inter -->

**Primary colour:**
<!-- Used for markers, table headers, and highlights -->
<!-- Provide a hex value e.g. #0057FF -->
<!-- Default: #E82E2E (red) -->

**Secondary colour:**
<!-- Used for dividers, row stripes, and backgrounds -->
<!-- Provide a hex value e.g. #F4F4F4 -->
<!-- Default: #F0F0F4 -->

**Spec frame width:**
<!-- Choose one: Compact (720px) / Standard (820px) / Wide (1080px) -->
<!-- Default: Standard -->

**Preview background:**
<!-- The background colour of the component preview area in the spec frame -->
<!-- e.g. #1A1A1A for dark, #FFFFFF for light, #F8F8F8 for neutral -->
<!-- Default: #3D3D47 (dark) -->

---

## 6. Token naming conventions

**Token format:**
<!-- Describe how your tokens are named. Examples: -->
<!-- components/button/global/colorprimary -->
<!-- color.brand.primary -->
<!-- --cf-color-primary -->

**Token categories used:**
<!-- List which token types exist in your system -->
<!-- e.g. Color, Spacing, Typography, Radius, Shadow, Motion -->

---

## 7. Component taxonomy

**Component categories:**
<!-- e.g. Actions (Button, CTA, Link), Inputs (Text field, Dropdown), -->
<!-- Feedback (Toast, Alert, Modal), Layout (Card, Grid, Divider) -->

**Variant naming:**
<!-- e.g. Status=Default, Status=Hover / Type=Primary, Size=SM -->

**Boolean properties:**
<!-- e.g. Has Icon=True/False, Show Label=True/False -->

---

## 8. Typography

**Font families:**
<!-- e.g. Invention, Inter, Söhne -->

**Type scale names:**
<!-- As they appear in Figma, e.g. -->
<!-- Heading/1, Heading/2, Body/LG, Body/MD, Label/Normal -->

---

## 9. Example components

List the first components you want to spec, with their Figma links:

| Component | Figma link | Depth override | Notes |
|-----------|------------|----------------|-------|
| | | | |
| | | | |
| | | | |

---

## 10. Notes for the agent

<!-- Any other rules the agent should follow on this project. Examples: -->
<!-- "This system uses density variants: Comfortable, Compact, Dense" -->
<!-- "All components have a dark mode variant using Figma variable modes" -->
<!-- "Specs should follow WCAG 2.1 AA as minimum for accessibility" -->
<!-- "Never include internal utility components like Spacer or Divider" -->
