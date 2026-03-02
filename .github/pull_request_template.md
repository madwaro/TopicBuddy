## Checklist

### Topic YAML
- [ ] Folder is named `WorkdayEmployee<ScenarioName>` under `EmployeeScenarios/`
- [ ] `scenarioName` follows `msdyn_HRWorkday<Domain><Action>` pattern
- [ ] Employee identified via `Global.ESS_UserContext_Employee_Id`
- [ ] All Workday API calls go through `WorkdaySystemGetCommonExecution`
- [ ] API errors are humanized with `AnswerQuestionWithAI` before showing to user
- [ ] On failure, a retry is offered using `GotoAction`

### Adaptive Cards
- [ ] All text (headings, labels, buttons) uses sentence case
- [ ] Headings use `size: Large` + `weight: Bolder`
- [ ] Body text uses default size (no `size` property)
- [ ] Secondary/tertiary text uses `size: Small`
- [ ] `FactSet` used instead of `Table` for key-value summaries
- [ ] No Workday icon or "Workday" branding inside cards

### Standalone card files
- [ ] One `.json` file per card step under `cards/`
- [ ] Files are prefixed `step1-`, `step2-`, `step3-`, etc.
- [ ] Static sample data replaces Power Fx expressions
- [ ] All card files are valid JSON
