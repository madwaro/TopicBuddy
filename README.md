# Topic Buddy

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

## Deploying and testing a topic in Copilot Studio

### 1. Upload XML templates
For each XML file in the scenario folder:
1. Open the Employee Self-Service agent in Copilot Studio
2. Go to **Solutions → Default → New → More → Other → Employee Self-Service Template Configuration**
3. Fill in the form:
   - **Name**: a human-readable label (e.g. `WorkdayEmployeeCancelTimeOff - Get Requests`)
   - **Unique name**: copy the `name` attribute from `<requestTemplate name="...">` in the XML file
   - **Value**: paste the entire XML file contents
4. Select **Save and close** — repeat for each XML file in the scenario

### 2. Import the topic
1. In Copilot Studio, go to the **Topics** tab
2. Select **+ Add a topic → From blank**
3. Select **More → Open code editor** (top-right)
4. Paste the full contents of `topic.yaml` and save

### 3. Configure Workday permissions
Work with your Workday admin to grant the required security domain permissions for the new business processes. Consult the [Workday extensibility article](https://learn.microsoft.com/en-us/copilot/microsoft-365/employee-self-service/workday-extensibility) for guidance.

### 4. Test in Copilot Studio
Use the built-in **Test** panel with trigger phrases from the scenario's `README.md`.

## Conventions

See [`.github/copilot-instructions.md`](.github/copilot-instructions.md) for the full set of authoring conventions. Key points:

- Scenario folders: `WorkdayEmployee<ScenarioName>`
- `scenarioName` values: `msdyn_HRWorkday<Domain><Action>`
- All card text in **sentence case**
- Headings: `Large` + `Bolder` — body text: default size — secondary text: `Small`
- Prefer `FactSet` over `Table` for key-value summaries
- No Workday icon inside cards
- Always handle API errors with `AnswerQuestionWithAI` + retry loop
- **Always initialize variables with `SetVariable` before using them in conditions**, even if the value will be overwritten by a `BeginDialog` output — Copilot Studio requires variables to be declared before they are referenced
