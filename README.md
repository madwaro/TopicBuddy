# Workday Copilot Studio scenarios

Copilot Studio topic definitions and Adaptive Card samples for Workday Employee Self-Service scenarios.

## Getting started

1. Install the recommended VS Code extensions (see below)
2. Clone this repo
3. Open in VS Code — Copilot will automatically pick up conventions from `.github/copilot-instructions.md`
4. Create a branch for your scenario: `git checkout -b scenario/<your-scenario-name>`
5. Add your scenario folder under `EmployeeScenarios/`
6. Open a PR when ready for review

## Recommended VS Code extensions

| Extension | Purpose |
|---|---|
| [Adaptive Card Previewer](https://marketplace.visualstudio.com/items?itemName=TeamsDevApp.vscode-adaptive-cards) | Preview card JSON files directly in VS Code. Iterate on card visuals without needing Copilot Studio. |
| [YAML](https://marketplace.visualstudio.com/items?itemName=redhat.vscode-yaml) | Syntax highlighting and validation for `topic.yaml` files. |

## Folder structure

```
EmployeeScenarios/
└── WorkdayEmployee<ScenarioName>/
    ├── topic.yaml          # Copilot Studio AdaptiveDialog topic
    └── cards/
        ├── step1-<name>.json   # Standalone Adaptive Card for testing
        ├── step2-<name>.json
        └── step3-<name>.json
```

## Testing cards

Paste any file from `cards/` into the [Adaptive Card Designer](https://adaptivecards.io/designer/) to preview it. Card files use static sample data in place of Power Fx expressions.

## Conventions

See [`.github/copilot-instructions.md`](.github/copilot-instructions.md) for the full set of authoring conventions. Key points:

- Scenario folders: `WorkdayEmployee<ScenarioName>`
- `scenarioName` values: `msdyn_HRWorkday<Domain><Action>`
- All card text in **sentence case**
- Headings: `Large` + `Bolder` — body text: default size — secondary text: `Small`
- Prefer `FactSet` over `Table` for key-value summaries
- No Workday icon inside cards
- Always handle API errors with `AnswerQuestionWithAI` + retry loop
