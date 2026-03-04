---
applyTo: "**/cards/*.json,**/topic.yaml"
---

# Adaptive Card conventions

These rules apply to all Adaptive Cards — both standalone JSON files in `cards/` and inline cards within `topic.yaml`.

## Text & style

- All card text (headings, labels, button titles) must use **sentence case** — only the first word and proper nouns are capitalized.
- Text size conventions:
  - **Headings**: `"size": "Large"` with `"weight": "Bolder"`
  - **Body text**: default size (omit the `size` property)
  - **Secondary/tertiary text** (hints, captions, field labels): `"size": "Small"`
- **Confirmation step headings with emojis**: place the emoji at the **end** of the heading text, not the beginning (e.g. `"Request cancelled ✅"`, not `"✅ Request cancelled"`).
- Do not include the Workday icon or "Workday" branding inside cards. Every element must display meaningful content — no empty branding blocks.

## Layout & structure

- Use `FactSet` for displaying key-value data summaries (e.g. request details, confirmation screens). Only use `Table` when you need more complex layout control that `FactSet` cannot provide.
- **Never use `ActionSet` in the card `body`.** Always place submit/cancel actions in the card's top-level `actions` array. This applies to all cards in all scenarios.

### Body array — prevent Power Fx union type pollution

When the card `body[]` array contains mixed element types, Power Fx infers a union record type and serializes ALL properties from ALL types on EVERY element (with `null` for unused ones), causing `adaptiveCardInvalid` errors.

**FIX:** The body array should contain ONLY: `TextBlock`, `Container`, and `ColumnSet`. Wrap other inputs:
- `Input.ChoiceSet` → wrap in `ColumnSet > Column > items: [...]`
- `Input.Date`, `Input.Number`, `Input.Text` → wrap in `Container > items: [...]`

### Column rules

- **Every Column MUST have `items`** — even spacer Columns: `items: []`
- **Always add `verticalContentAlignment: "Center"`** to every Column
- Width-stretch spacer Columns: `{ type: "Column", width: "stretch", items: [], verticalContentAlignment: "Center" }`

### TextBlock rules

- **`text` must NEVER be null**. Use `If(IsBlank(value), "—", value)` as fallback for any dynamic text.

### Input rules

- **`isRequired: true` inputs MUST have a `label`** property
- **`errorMessage`** should be provided for all required inputs

## Standalone card files

- Cards extracted as standalone test files go in a `cards/` subfolder and are prefixed with their step number: `step1-`, `step2-`, `step3-`, etc.
- Standalone card files use static sample data in place of Power Fx expressions so they can be tested directly in the [Adaptive Card Designer](https://adaptivecards.io/designer/).
