---
name: hubspot-style-fields
description: Generate grouped, collapsible HubSpot module style fields (fields.json) from a Figma design node, so properties render organized by component (Container, Card, Heading, Button, Image, etc.) under the Style tab instead of as a flat list. Use this whenever the user is building or editing a HubSpot CMS module and mentions fields.json, the Style tab, style properties/groups, or gives a Figma link/node ID for a module to style. Also trigger on phrases like "group these style properties," "organize the style tab," "add style fields for this module," or "pull styles from Figma for this component."
---

# HubSpot Grouped Style Fields

Generates a properly grouped, Figma-accurate `fields.json` style block for a HubSpot CMS module, and reports the resulting HubL dot-paths.

## When to use this

- The user is defining or fixing style properties for a HubSpot module and wants them organized into collapsible sections in the page editor (Container, Card, Heading, Button, Image, etc.) instead of one flat list.
- The user gives a Figma file/node URL for a module and wants matching style fields generated.
- The user asks to "add a [Button/Image/Card/...] style group" to an existing module.

## Core mechanism (don't deviate from this)

1. **Every style field must live inside exactly one top-level field group** with `"type": "group"` and `"tab": "STYLE"`. Fields outside this wrapper, or with `tab: "STYLE"` set directly on an individual field instead of the group, will not render correctly under the Style tab. This is the most common cause of "properties appear randomly" bugs — always check for this first when debugging an existing fields.json.
2. **Each component becomes a nested `"type": "group"` inside the top-level group.** A nested group renders as its own named, collapsible accordion section in the editor. Groups can nest inside groups (e.g. Card → Background → Background type) — each level becomes a sub-section.
3. **The JSON nesting path becomes the HubL reference path.** A field named `radius` inside group `card` inside root group `styles` is referenced in the template as `module.styles.card.radius`. Name groups after Figma layer/frame names so the JSON structure, editor UI, and template variables all stay self-documenting and in sync.
4. **The group for the module's outermost section/wrapper (background, border, padding, gap of the root frame) must always use `"name": "styles"` / `"id": "styles"` AND `"label": "Styles"`, never `"container"`/`"Container"`, `"wrapper"`, `"section"`, or the Figma frame's literal name.** This keeps both the HubL path (`module.styles.background`, `module.styles.padding`, etc.) and the editor UI accordion label consistent across every module in this project. Nested per-component groups (Heading, Button, Card, etc.) keep their own descriptive names/labels as usual — this rule only applies to the single outermost wrapper group.
5. **Any border must always be built as one native `"type": "border"` field — never a lone `color` field for the border.** A border field gives the editor width (thickness), style (solid/dashed/dotted/etc.), and color together as one control — you don't assemble those sub-values yourself. **Verified against this project's own working modules** (`card.module`, `button.module`, `pricing-card.module`, `menu.module` — cross-check any of them before touching a border field again if in doubt): the field must be wrapped in a `type: "group"` of the *same name*, and its `"default"` must be a literal empty object `{}` — HubSpot's API rejects any other default shape (a populated `{"border": {...}}` object was tried and failed with `border field at path border has an invalid default value` on upload). Exact working schema:
   ```json
   {
     "id": "<parent>.border",
     "name": "border",
     "label": "Border",
     "required": false,
     "locked": false,
     "children": [
       {
         "id": "<parent>.border.border",
         "name": "border",
         "label": "Border",
         "required": false,
         "locked": false,
         "allow_custom_border_sides": false,
         "type": "border",
         "display_width": null,
         "default": {}
       }
     ],
     "tab": "STYLE",
     "expanded": false,
     "group_occurrence_meta": null,
     "type": "group",
     "display_width": null
   }
   ```
   In HubL, access it through the single `.css` accessor, which outputs the full `border: ...;` declaration — don't hand-build `border-width`/`border-style`/`border-color` from sub-properties:
   ```
   {{ module.<parent>.border.border.css }}
   ```
   Because the field default can't carry a real color from Figma, put the actual Figma-derived border (`border: <width> <style> <color>;`) as a static fallback in `module.css` on the same selector — that's what renders until someone edits the field in the Style tab. Mention this limitation to the user when reporting output.
   If an existing module has a `color`-only field standing in for a border, treat it as a bug to fix (per the same core mechanism as rule 1), not a pattern to keep — replace it with the `border` field type above and update the template accordingly.
6. **Any drop shadow must always be built as a group of six individual primitive fields — there is no native `boxshadow` composite type, despite how it looks in the Style tab UI.** Verified against `case-studies.module` and `testimonials.module` in this project (and against HubSpot's own default "Language Switcher" module). Exact schema — reuse these field names verbatim:
   ```json
   {
     "id": "<parent>.shadow",
     "name": "shadow",
     "label": "Shadow",
     "required": false,
     "locked": false,
     "children": [
       { "id": "<parent>.shadow.enabled", "name": "enabled", "label": "Enable shadow", "type": "boolean", "required": false, "locked": false, "display_width": null, "default": true },
       { "id": "<parent>.shadow.offset_x", "name": "offset_x", "label": "Offset X", "type": "number", "display": "text", "min": -100, "max": 100, "step": 1, "required": false, "locked": false, "display_width": null, "default": 0 },
       { "id": "<parent>.shadow.offset_y", "name": "offset_y", "label": "Offset Y", "type": "number", "display": "text", "min": -100, "max": 100, "step": 1, "required": false, "locked": false, "display_width": null, "default": 0 },
       { "id": "<parent>.shadow.blur_radius", "name": "blur_radius", "label": "Blur radius", "type": "number", "display": "text", "min": 0, "max": 100, "step": 1, "required": false, "locked": false, "display_width": null, "default": 0 },
       { "id": "<parent>.shadow.spread_radius", "name": "spread_radius", "label": "Spread radius", "type": "number", "display": "text", "min": -100, "max": 100, "step": 1, "required": false, "locked": false, "display_width": null, "default": 0 },
       { "id": "<parent>.shadow.color", "name": "color", "label": "Color", "type": "color", "required": false, "locked": false, "display_width": null, "default": { "color": "#000000", "opacity": 10 } },
       { "id": "<parent>.shadow.inset", "name": "inset", "label": "Inset", "type": "boolean", "required": false, "locked": false, "display_width": null, "default": false }
     ],
     "tab": "STYLE",
     "expanded": false,
     "group_occurrence_meta": null,
     "type": "group",
     "display_width": null
   }
   ```
   Set each numeric/color default from the real Figma shadow value (`offset-x offset-y blur spread rgba(color)`), not the placeholders above. In HubL, build the `box-shadow` declaration manually — there's no `.css` accessor here since it's not a real composite field:
   ```
   {% if module.<parent>.shadow.enabled %}box-shadow: {% if module.<parent>.shadow.inset %}inset {% endif %}{{ module.<parent>.shadow.offset_x }}px {{ module.<parent>.shadow.offset_y }}px {{ module.<parent>.shadow.blur_radius }}px {{ module.<parent>.shadow.spread_radius }}px rgba({{ module.<parent>.shadow.color.color|convert_rgb }}, {{ module.<parent>.shadow.color.opacity / 100 }});{% endif %}
   ```
   If an existing module has a `choice` dropdown with one hardcoded shadow value standing in for this (a common workaround), replace it with the real editable group above rather than leaving it as a fixed preset.

## Workflow

### Step 1 — Get the design data (skip if no Figma reference given)

If the user gives a Figma URL with a node ID:
1. Load the `figma-design-to-code` skill (mandatory prerequisite before calling Figma MCP tools).
2. Call `get_design_context` on that node to retrieve the real, computed properties — fills/background, padding, gap (auto-layout), corner radius, stroke/border, effects (shadow, blur), typography, alignment. Don't guess or estimate values — only use what the tool returns.
3. If the node contains multiple visual components (e.g. a Card frame that contains an Image, a Heading, and the Card container itself), identify each as a separate group candidate.

If no Figma reference is given, ask the user for the component breakdown and approximate values, or work from a module spec they provide. Don't stall on missing Figma access — proceed with reasonable defaults and note them.

### Step 2 — Map every property to a native HubSpot field type

Never invent a custom field type. Use this mapping as the default; if something doesn't fit, flag it explicitly to the user rather than approximating silently.

| Figma / design property | HubSpot field `type` | Notes |
|---|---|---|
| Fill / background color | `color` | includes opacity |
| Padding, margin, gap (auto-layout) | `spacing` | |
| Corner radius | `number` (`display: "text"` or `"slider"`) or `borderradius` | use `borderradius` if independent corners are needed |
| Stroke / border | `border` | must include width + style + color together — see rule 5; never a lone `color` field |
| Drop shadow / blur effect | **no composite type — build from primitives** | **`"boxshadow"` is NOT a real HubSpot field type.** Two attempts at guessing a composite schema for it were both rejected by the API with a confusing cascading error (`Field styles.card.null is missing a label`, `'unknown' is not a valid field type`) — that error signature means the field type name itself is invalid, not just the default shape. Confirmed via HubSpot's own default "Language Switcher" module (fetched from `developers.hubspot.com/docs/cms/reference/modules/default-module-versioning`): shadow is built as a `type: "group"` containing individual primitive fields. See the exact schema below — use it verbatim, don't reinvent it. |
| Font family/size/weight/line-height | `font` | |
| Text/flex alignment | `alignment` | set `alignment_direction` |
| Width constraints | `number` with `display: "slider"` and `suffix: "px"` | |
| Show/hide toggle | `boolean` | |

### Step 3 — Assemble the fields.json

Structure:
```
[
  {
    "label": "Styles",
    "name": "styles",
    "type": "group",
    "tab": "STYLE",
    "children": [
      { "label": "<Component 1>", "name": "<component_1>", "type": "group", "children": [ ...fields or nested groups... ] },
      { "label": "<Component 2>", "name": "<component_2>", "type": "group", "children": [ ... ] }
    ]
  }
]
```

Rules while building this:
- One group per visual component. Component names/labels should match the Figma layer names the user is already using, so the editor UI matches their design vocabulary.
- The group holding the module's outermost/wrapper properties (background, border, padding, gap of the root frame) always uses `"name": "styles"` / `"id": "styles"` / `"label": "Styles"` — never `container`/`"Container"` or the Figma frame's own name.
- Never place a property directly under the root `styles` group — every property belongs inside a component-level group, even if that means a group with just one field.
- Set each field's `"default"` to the **actual value read from Figma** in Step 1 (e.g. exact shadow offsets/blur/color, exact hex/rgba, exact padding numbers) — not a placeholder or a rounded guess.
- Preserve the array order the user cares about — group order in the JSON is the order sections appear in the editor.
- If merging into an *existing* module's fields.json, read the current file first, keep existing content fields untouched, and only add/reorganize the style group. Warn the user before restructuring an already-published module's field hierarchy — reordering or renaming existing fields can cause existing module instances to lose saved data (this is a real HubSpot constraint, not just a style nicety).

### Step 3.5 — Validate against the real API before finishing

For any field type whose default-value schema isn't already proven working elsewhere in this project (checked via `grep -rl '"type": "<type>"' modules/`), don't just trust a best-guess schema — run `hs cms upload <module-folder> Nexus/modules/<module-folder>` and read the response. HubSpot's API validates field defaults server-side and will reject a malformed one, sometimes with a confusing cascading error that names an unrelated field (e.g. a bad `boxshadow` default surfaced as `Field styles.card.null is missing a label`). If the upload fails:
1. Read the actual error, not just the field you suspect.
2. If a working example of that field type exists elsewhere in this project, copy its exact schema (this is how the `border` schema in rule 5 was fixed).
3. If no working example exists and the fix isn't obvious, remove the field rather than iterating blindly — fall back to a fixed value in `module.css` and flag it per Step 4, rather than leaving a broken upload in place.
4. Re-run the upload to confirm the fix actually works before reporting success to the user.

### Step 4 — Report output

After writing/updating the fields.json:
1. Show the full JSON block (or the diff, if editing an existing file).
2. List every resulting HubL dot-path, e.g.:
   - `module.styles.container.background_color`
   - `module.styles.card.border_radius`
   - `module.styles.heading.font`
3. List any Figma property that had no clean 1:1 HubSpot field mapping, and how it was approximated (or note that it was skipped and needs a manual decision).

## Reusability across modules

This same flow applies per-module — run Steps 1–4 fresh for each module/component the user points you at (Card, Button, Image, etc.). If the user is building a design system with repeated groups across many modules (e.g. the same "Card" style group used in a Testimonial module and a Team module), ask whether they want:
- independent copies of the group per module (simplest, no shared dependency), or
- a shared field-group partial they maintain once and copy into each module's fields.json (better for consistency, needs a small library file to hold canonical group definitions — offer to set this up if they want it).

## Quick sanity checklist before finishing

- [ ] Exactly one root group with `tab: "STYLE"`
- [ ] No field sits directly under the root group — every field is inside a component group
- [ ] Group `name` values are valid HubL identifiers (snake_case, no spaces)
- [ ] Every `default` value traces back to a real Figma value, not a guess
- [ ] Any unmapped Figma property was flagged, not silently dropped
- [ ] Every border is a single `"type": "border"` field exposing width + style + color — no border is represented by a lone `color` field
- [ ] Every shadow is the six-field primitive group (rule 6) — never a `"boxshadow"` type (doesn't exist) and never a `choice` dropdown standing in for it
