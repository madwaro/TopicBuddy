# Copilot instructions for Workday Copilot Studio scenarios

This repo contains Copilot Studio topic definitions and Adaptive Card samples for Workday Employee Self-Service scenarios.

## Topic YAML conventions

- Each scenario lives in its own folder under `src/Workday/EmployeeScenarios/` or `src/Workday/ManagerScenarios/`, e.g. `src/Workday/EmployeeScenarios/WorkdayEmployeeCancelTimeOff/`.
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

See [`.github/instructions/adaptive-cards.instructions.md`](.github/instructions/adaptive-cards.instructions.md) for all card conventions. That file is automatically applied to all `cards/*.json` files.

## Trigger phrases

Write trigger phrases in plain, natural employee language (e.g. "Cancel my time off", "I need to take some time off").
