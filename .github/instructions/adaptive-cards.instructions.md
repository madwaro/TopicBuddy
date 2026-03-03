---
applyTo: "**/cards/*.json"
---

# Adaptive Card conventions

- All card text (headings, labels, button titles) must use **sentence case** — only the first word and proper nouns are capitalized.
- Text size conventions:
  - **Headings**: `"size": "Large"` with `"weight": "Bolder"`
  - **Body text**: default size (omit the `size` property)
  - **Secondary/tertiary text** (hints, captions, field labels): `"size": "Small"`
- Use `FactSet` for displaying key-value data summaries (e.g. request details, confirmation screens). Only use `Table` when you need more complex layout control that `FactSet` cannot provide.
- Do not include the Workday icon or "Workday" branding inside cards.
- Cards extracted as standalone test files go in a `cards/` subfolder and are prefixed with their step number: `step1-`, `step2-`, `step3-`, etc.
- Standalone card files use static sample data in place of Power Fx expressions so they can be tested directly in the [Adaptive Card Designer](https://adaptivecards.io/designer/).
- **Never use `ActionSet` in the card `body`.** Always place submit/cancel actions in the card's top-level `actions` array. This applies to all cards in all scenarios.
- **Confirmation step headings with emojis**: place the emoji at the **end** of the heading text, not the beginning (e.g. `"Request cancelled ✅"`, not `"✅ Request cancelled"`).
