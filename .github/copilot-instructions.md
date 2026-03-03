# Copilot instructions for Workday Copilot Studio scenarios

This repo contains Copilot Studio topic definitions and Adaptive Card samples for Workday Employee Self-Service scenarios.

## Topic YAML conventions

- Each scenario lives in its own folder under `EmployeeScenarios/`, e.g. `WorkdayEmployeeCancelTimeOff/`.
- The main file is always `topic.yaml` using `kind: AdaptiveDialog`.
- Reuse the shared execution wrapper for all Workday API calls:
  ```yaml
  dialog: msdyn_copilotforemployeeselfservice.topic.WorkdaySystemGetCommonExecution
  ```
  The correct solution prefix is `msdyn_copilotforemployeeselfservice` (no `hr` suffix). Using `msdyn_copilotforemployeeselfservicehr` will cause "Selected topic is no longer available" redirect errors in the topic checker.
- The `WorkdaySystemGetCommonExecution` dialog accepts two inputs (`parameters`: String, `scenarioName`: String) and returns three outputs (`errorResponse`: String, `isSuccess`: Boolean, `workdayResponse`: String). Always bind using these exact names and types.
- Follow the scenario naming pattern for `scenarioName` values:
  `msdyn_HRWorkday<Domain><Action>` — e.g. `msdyn_HRWorkdayAbsenceCancelTimeOff`.
- Always identify the employee via `Global.ESS_UserContext_Employee_Id`.
- Use `AnswerQuestionWithAI` to humanize Workday error messages before showing them to the user.
- On API failure, always offer a retry that loops back to the relevant form or selection step using `GotoAction`.
- **Always initialize variables with `SetVariable` before using them in conditions**, even if the value will be overwritten by a `BeginDialog` output binding. Copilot Studio requires variables to be declared before they are referenced in expressions. This applies to **all** output binding variables — initialize booleans to `false` and strings to `""`.

## Adaptive Card conventions

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

## Trigger phrases

Write trigger phrases in plain, natural employee language (e.g. "Cancel my time off", "I need to take some time off").
