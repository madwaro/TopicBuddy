---
applyTo: "**/topic.yaml,**/msdyn_*.xml,**/README.md"
---

# ESS Workday Topic Builder

You are an expert agent that creates Microsoft Employee Self-Service (ESS) topics for Copilot Studio that integrate with Workday via SOAP APIs. You produce production-ready `topic.yaml` and `msdyn_*.xml` files, plus a `README.md` for each topic.

---

## Architecture

Every ESS Workday topic consists of **2+ files per topic folder**:

| File | Purpose |
|------|---------|
| `topic.yaml` | Copilot Studio AdaptiveDialog — conversation flow, triggers, Adaptive Cards, API calls |
| `msdyn_*.xml` | ESS Template Configuration — Workday SOAP request body + XPath response extraction |
| `README.md` | Documentation — what it does, SOAP operations, permissions needed, tenant config, install steps |

All topics route SOAP calls through a shared dialog: `msdyn_copilotforemployeeselfservice.topic.WorkdaySystemGetCommonExecution`

> **Important:** The correct solution prefix is `msdyn_copilotforemployeeselfservice` (no `hr` suffix). Using `msdyn_copilotforemployeeselfservicehr` causes "Selected topic is no longer available" redirect errors.

### Key Globals

| Variable | Description |
|----------|-------------|
| `Global.ESS_UserContext_Employee_Id` | Current user's Workday Employee ID |
| `Global.ESS_UserContext_ManagerOrganizationId` | Current user's Manager Organization ID |
| `Global.ESS_UserContext_Employee_Firstname` | Current user's first name |
| `Global.ESS_UserContext_Employee_Lastname` | Current user's last name |

### Shared Dialogs

| Dialog | Purpose |
|--------|---------|
| `WorkdaySystemGetCommonExecution` | Executes SOAP call using template config. Input: `scenarioName` + `parameters`. Output: `isSuccess`, `errorResponse`, `workdayResponse` |
| `WorkdayManagerCheck` | Validates the requesting user is a manager before running manager topics |

---

## XML Template Configuration

### Structure

```xml
<workdayEntityConfigurationTemplate>
  <scenario name="SCENARIO_NAME">
    <apiRequests>
      <apiRequest>
        <authType>User</authType>
        <endpoint>
          <request>TEMPLATE_NAME</request>
          <serviceName>SERVICE_NAME</serviceName>
          <version>v45.0</version>
        </endpoint>
        <responseProperties>
          <property>
            <extractPath>XPATH_EXPRESSION</extractPath>
            <key>FIELD_KEY</key>
          </property>
        </responseProperties>
      </apiRequest>
    </apiRequests>
  </scenario>
  <requestTemplates>
    <requestTemplate name="TEMPLATE_NAME">
      <bsvc:Operation_Request xmlns:bsvc="urn:com.workday/bsvc" bsvc:version="v45.0">
        <!-- SOAP body with {placeholder} params -->
      </bsvc:Operation_Request>
    </requestTemplate>
  </requestTemplates>
</workdayEntityConfigurationTemplate>
```

### Critical Rules for XML Templates

1. **`scenarioName` in YAML maps to `requestTemplate name` in XML**, NOT the `scenario name`.
2. **Keep XML compact (~2KB max)**. Copilot Studio truncates large configs. Drop all `Include_*: false` flags (they're defaults). Remove inline comments.
3. **XPath uses `local-name()` to ignore namespaces**: `//*[local-name()='ElementName']/text()`
4. **All extractPaths must return the same count**. The plugin merges arrays by index — mismatched counts misalign data.
5. **Parameters in the topic's `parameters` JSON must match `{placeholder}` names in the XML exactly**. Extra params → "length of api request is not 1" error. Missing params → literal `{placeholder}` text sent.

### XPath Patterns

| What | XPath |
|------|-------|
| Simple text | `//*[local-name()='Date']/text()` |
| Nested child | `//*[local-name()='Parent']/*[local-name()='Child']/text()` |
| Attribute filter | `//*[local-name()='ID'][@*[local-name()='type']='WID']/text()` |
| Descriptor attribute | `//*[local-name()='Type_Reference']/@*[local-name()='Descriptor']` |

### Multi-Day Write Pattern (repeat-for)

```xml
<wd:Enter_Time_Off_Entry_Data wd:repeat-for="{timeoff_Dates}" wd:repeat-item="{timeoff_Date}">
  <wd:Date>{timeoff_Date}</wd:Date>
  <wd:Requested>{timeoff_Hours_Per_Day}</wd:Requested>
</wd:Enter_Time_Off_Entry_Data>
```

---

## Topic YAML Structure

```yaml
kind: AdaptiveDialog

inputs:                        # NLU-extracted inputs
  - kind: AutomaticTaskInput
    propertyName: PropertyName
    description: "Description for NLU extraction"
    entity: DateTimePrebuiltEntity
    shouldPromptUser: false
    inputSettings:
      repeatCount: 0
      defaultValue: =Blank()

modelDescription: |-           # LLM guardrails (max 1024 chars)
  Description of what this topic does...

beginDialog:
  kind: OnRecognizedIntent
  id: main
  intent:
    triggerQueries:
      - "trigger phrase 1"
      - "trigger phrase 2"

  actions:
    - kind: SetVariable
    - kind: SendActivity
    - kind: AdaptiveCardPrompt
    - kind: BeginDialog
    - kind: ParseValue
    - kind: ConditionGroup
    - kind: AnswerQuestionWithAI
    - kind: GotoAction
    - kind: CancelAllDialogs

inputType:
  properties:
    PropertyName:
      displayName: PropertyName
      description: "..."
      type: DateTime  # or String, Number, Boolean

outputType: {}
```

### Available Prebuilt Entities

```
StringPrebuiltEntity    NumberPrebuiltEntity    DateTimePrebuiltEntity
PersonNamePrebuiltEntity  BooleanPrebuiltEntity  EmailPrebuiltEntity
```

### The API Call Pattern

```yaml
- kind: BeginDialog
  id: call_workday
  displayName: Call Workday API
  input:
    binding:
      parameters: ="{""params"":[{""key"":""{Employee_ID}"",""value"":""" & Global.ESS_UserContext_Employee_Id & """},{""key"":""{Start_Date}"",""value"":""" & Topic.startDate & """}]}"
      scenarioName: msdyn_TemplateName
  dialog: msdyn_copilotforemployeeselfservice.topic.WorkdaySystemGetCommonExecution
  output:
    binding:
      errorResponse: Topic.errorResponse
      isSuccess: Topic.isSuccess
      workdayResponse: Topic.workdayResponse
```

### Parsing Parallel Arrays into a Table

```yaml
- kind: ParseValue
  id: parse_response
  variable: Topic.parsed
  valueType:
    kind: Record
    properties:
      FieldA:
        type:
          kind: Table
          properties:
            Value: String
      FieldB:
        type:
          kind: Table
          properties:
            Value: String
  value: =Topic.workdayResponse

- kind: SetVariable
  id: merge_arrays
  variable: Topic.mergedTable
  value: |-
    =ForAll(
      Sequence(CountRows(Topic.parsed.FieldA)),
      {
        FieldA: Last(FirstN(Topic.parsed.FieldA, Value)).Value,
        FieldB: Last(FirstN(Topic.parsed.FieldB, Value)).Value
      }
    )
```

### Power Fx Context Rules

| Context | Evaluates `=` expressions? | Notes |
|---------|---------------------------|-------|
| `value:` in SetVariable | ✅ Yes | Primary place for all logic |
| `condition:` in ConditionGroup | ✅ Yes | Must return boolean |
| `card:` in AdaptiveCardPrompt | ✅ Yes | Returns Adaptive Card JSON object |
| `activity:` in SendActivity | ❌ No | Literal string. Use `{Topic.Var}` interpolation only |
| `prompt:` in Question | ❌ No | Same — use `{Topic.Var}` interpolation |

**CRITICAL:** To use dynamic text in `SendActivity`, compute in a `SetVariable` first, then reference:
```yaml
- kind: SetVariable
  id: set_msg
  variable: Topic.msg
  value: ="Your balance is " & Text(Topic.balance) & " hours."
- kind: SendActivity
  id: send_msg
  activity: "{Topic.msg}"
```

---

## Adaptive Card Rules

All Adaptive Card conventions (body array structure, Column rules, TextBlock/Input rules, branding, styling) are defined in `adaptive-cards.instructions.md`. That file applies to both standalone card JSON files and inline cards in `topic.yaml`.

---

## Topic Flow Patterns

### Pattern 1: Employee READ

```
User triggers → Set config vars → Intro message with Workday link → API call →
AnswerQuestionWithAI formats response → End
```

### Pattern 2: Employee WRITE (with Adaptive Card form)

```
User triggers → Set config vars → Intro message → AdaptiveCardPrompt collects inputs →
[Optional: validation API call] → Review/confirmation AdaptiveCardPrompt →
Submit API call → Success/Error card → End
```

### Pattern 3: Manager READ

```
User triggers → WorkdayManagerCheck → Intro message → API call (lightweight list) →
Filter by EmployeeName (exclude self-name!) → If multiple matches: disambiguation →
API call 2 (full detail) → Display Adaptive Card → End
```

### Pattern 4: Employee CANCEL/MODIFY

```
User triggers → Set config vars → Fetch list API call → Parse + filter to relevant items →
Auto-select if single match or date match → If multiple: selection AdaptiveCardPrompt →
Confirmation AdaptiveCardPrompt → Execute API call → Success/Error → End
```

---

## Workday SOAP API Reference

### How to Look Up Operations

1. Go to: `https://community.workday.com/sites/default/files/file-hosting/productionapi/index.html`
2. Find the service (e.g., `Absence_Management`, `Human_Resources`, `Time_Tracking`)
3. Click the operation to see request/response element structure
4. Download sample XML: `{ServiceName}/v45.2/samples/{OperationName}_Request.xml`

### Key Services for ESS

| Service | Common Operations |
|---------|-------------------|
| **Absence_Management** | `Get_Time_Off_Plan_Balances`, `Enter_Time_Off`, `Cancel_Time_Off_Event`, `Get_Absence_Inputs` |
| **Human_Resources** | `Get_Workers`, `Change_Home_Contact_Information`, `Change_Emergency_Contacts` |
| **Time_Tracking** | `Get_Calculated_Time_Blocks` (schedule detection — preferred over `Get_Work_Schedule`) |
| **Talent** | `Give_Feedback`, `Manage_Education`, `Get_Skills` |
| **Staffing** | `Get_Workers` (alternate), position management |

### API Choice: Schedule Detection

Use **`Time_Tracking → Get_Calculated_Time_Blocks`** for determining working days and scheduled hours. Do NOT use `Human_Resources → Get_Work_Schedule` — it returns 0 hours for many tenants where Workforce Management isn't fully configured. `Get_Calculated_Time_Blocks` requires a tenant-specific `Calculation_Tag_ID`.

### SOAP Envelope

The ESS template XML contains ONLY the body element. The shared `WorkdaySystemGetCommonExecution` dialog handles envelope wrapping, authentication, and transport. The namespace is always `urn:com.workday/bsvc`.

---

## Mandatory Checklist — Verify Before Every Topic

### YAML & modelDescription

- `modelDescription` max 1024 characters. Use abbreviations (e.g., "Emp ID" not "Employee ID")
- **Every action node MUST have an `id` property** — this includes `SetVariable`, `BeginDialog`, `SendActivity`, `AdaptiveCardPrompt`, `ConditionGroup`, `Question`, `ParseValue`, `GotoAction`, `CancelAllDialogs`, and all other action kinds
- `modelDescription` uses `=` assignments, explicit formatting verbs, ends with `"Double check if rules are followed, fix if not"`
- WRITE topics: `modelDescription` describes the operation context. READ topics: concise field listings
- No `displayName` on ConditionGroup `conditions:` items — only `id` and `condition` allowed
- Every `condition` in a ConditionGroup has an `id`

### Adaptive Cards (inline in topic.yaml)

- All rules from `adaptive-cards.instructions.md` apply (body array structure, Column items, TextBlock null fallbacks, Input labels, no branding blocks)

### Power Fx & Messages

- `SendActivity` `activity:` does NOT evaluate Power Fx — set variable first, then `activity: "{Topic.var}"`
- **Single response per turn** — no back-to-back SendActivity, no SendActivity before Question/AdaptiveCardPrompt. Merge into card TextBlock or Question prompt
- `AnswerQuestionWithAI` auto-sends — do NOT follow with SendActivity showing the same content

### XML Templates

- XML stays compact (~2KB). Drop all `Include_*: false` defaults, remove comments
- Parameters match XML placeholders exactly — no extras, no missing
- **SOAP element names verified against WSDL** — NEVER assume by analogy. `Worker_Reference`, `Employee_Reference`, `Payee_Reference` are different elements in different operations. Always check `https://community.workday.com/sites/default/files/file-hosting/productionapi/{ServiceName}/v45.2/{OperationName}.html`
- All extractPaths return same count (parallel arrays must align)

### Topic Flow

- Intro message with Workday link sent before API call
- Tenant URL uses `<TENANT_NAME>` placeholder — never hardcoded
- Manager topics: exclude self-name from EmployeeName filter (all 5 variants: first, last, "first last", "last first", "last, first") using `Lower(Trim(...))`
- Topic gets to the point — no unnecessary preamble
- **Always initialize variables with `SetVariable` before using them in conditions**, even if the value will be overwritten by a `BeginDialog` output binding

### Tenant Configuration

- `Calculation_Tag_ID` is tenant-specific (Time_Tracking topics)
- Cap per-day deduction at `Min(scheduledHours, requestedHoursPerDay)`
- Leave type IDs in config tables match tenant's Workday setup

### Documentation

- `README.md` in topic folder with: purpose, SOAP operations, required permissions, tenant config, install steps

---

## Workday Permissions Template

Include this in the README for every new topic:

```markdown
## Required Workday Permissions

The Integration System User (ISU) used by the Power Platform Workday SOAP connector must have:

| Permission | Domain Security Policy | Access |
|------------|----------------------|--------|
| **[Operation Name]** | [Domain Name] | **Get** or **Put** |

### How to Grant Permissions

1. In Workday, search "Integration System Security"
2. Find the ISU associated with the Power Platform connector
3. Add the security group(s) / domain security policies listed above
4. Search "Activate All Pending Security Policy Changes" and activate
5. Test the topic — "task submitted is not authorized" error should resolve
```

---

## Complete Working Examples

### Example: GET XML Template (Absence Inputs by Employee)

```xml
<workdayEntityConfigurationTemplate>
  <scenario name="msdyn_HRWorkdayAbsenceGetTimeOffEntries">
    <apiRequests>
      <apiRequest>
        <authType>User</authType>
        <endpoint>
          <request>Template_GetAbsenceInputsRequest</request>
          <serviceName>Absence_Management</serviceName>
          <version>v45.0</version>
        </endpoint>
        <responseProperties>
          <property>
            <extractPath>//*[local-name()='Absence_Input']/*[local-name()='Absence_Input_Data']/*[local-name()='Event_Reference']/*[local-name()='ID']/text()</extractPath>
            <key>EventID</key>
          </property>
          <property>
            <extractPath>//*[local-name()='Absence_Input']/*[local-name()='Absence_Input_Data']/*[local-name()='Date']/text()</extractPath>
            <key>TimeOffDate</key>
          </property>
        </responseProperties>
      </apiRequest>
    </apiRequests>
  </scenario>
  <requestTemplates>
    <requestTemplate name="Template_GetAbsenceInputsRequest">
      <bsvc:Get_Absence_Inputs_Request xmlns:bsvc="urn:com.workday/bsvc" bsvc:version="v45.0">
        <bsvc:Request_Criteria>
          <bsvc:Employee_Reference>
            <bsvc:ID bsvc:type="Employee_ID">{Employee_ID}</bsvc:ID>
          </bsvc:Employee_Reference>
        </bsvc:Request_Criteria>
      </bsvc:Get_Absence_Inputs_Request>
    </requestTemplate>
  </requestTemplates>
</workdayEntityConfigurationTemplate>
```

### Example: WRITE XML Template (Cancel Time Off Event)

```xml
<workdayEntityConfigurationTemplate>
  <scenario name="msdyn_HRWorkdayAbsenceCancelTimeOffEvent">
    <apiRequests>
      <apiRequest>
        <authType>User</authType>
        <endpoint>
          <request>Template_CancelTimeOffEventRequest</request>
          <serviceName>Absence_Management</serviceName>
          <version>v45.0</version>
        </endpoint>
        <responseProperties>
          <property>
            <extractPath>//*[local-name()='Event_Reference']/*[local-name()='ID']/text()</extractPath>
            <key>CancelledEventID</key>
          </property>
        </responseProperties>
      </apiRequest>
    </apiRequests>
  </scenario>
  <requestTemplates>
    <requestTemplate name="Template_CancelTimeOffEventRequest">
      <bsvc:Cancel_Time_Off_Event_Request xmlns:bsvc="urn:com.workday/bsvc" bsvc:version="v45.0">
        <bsvc:Business_Process_Parameters>
          <bsvc:Auto_Complete>true</bsvc:Auto_Complete>
          <bsvc:Run_Now>true</bsvc:Run_Now>
        </bsvc:Business_Process_Parameters>
        <bsvc:Cancel_Time_Off_Event_Data>
          <bsvc:Event_Reference>
            <bsvc:ID bsvc:type="WID">{Time_Off_Event_ID}</bsvc:ID>
          </bsvc:Event_Reference>
        </bsvc:Cancel_Time_Off_Event_Data>
      </bsvc:Cancel_Time_Off_Event_Request>
    </requestTemplate>
  </requestTemplates>
</workdayEntityConfigurationTemplate>
```

### Example: Configuration Block (standard for all topics)

```yaml
actions:
  - kind: SetVariable
    id: set_workday_url
    variable: Topic.WorkdayUrl
    value: ="https://impl.workday.com/<TENANT_NAME>/d/home.htmld"

  - kind: SetVariable
    id: set_workday_icon_url
    variable: Topic.WorkdayIconUrl
    value: ="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAQAAAC1HAwCAAAAC0lEQVR42mNkYAAAAAYAAjCB0C8AAAAASUVORK5CYII="

  - kind: SetVariable
    id: set_leave_type_config
    variable: Topic.LeaveTypeConfig
    value: |-
      =Table(
        {TypeID: "Vacation_Hours", DisplayName: "Vacation"},
        {TypeID: "Floating_Holiday_Hours", DisplayName: "Floating holiday"},
        {TypeID: "Sick_Hours", DisplayName: "Sick leave"}
      )
```

### Example: Error Handling Pattern

```yaml
- kind: ConditionGroup
  id: cg_error
  conditions:
    - id: ci_error
      condition: =Topic.isSuccess = false
      actions:
        - kind: SetVariable
          id: set_error_msg
          variable: Topic.errorMsg
          value: ="I wasn't able to complete that request. You can try directly in [Workday](" & Topic.WorkdayUrl & ")."
        - kind: SendActivity
          id: send_error
          activity: "{Topic.errorMsg}"
        - kind: CancelAllDialogs
          id: cancel_on_error
```

### Example: Manager Self-Name Exclusion

```yaml
condition: =!IsBlank(Topic.EmployeeName)&&
    Lower(Trim(Topic.EmployeeName)) <> Lower(Trim(Global.ESS_UserContext_Employee_Firstname))
    &&
    Lower(Trim(Topic.EmployeeName)) <> Lower(Trim(Global.ESS_UserContext_Employee_Lastname))
    &&
    Lower(Trim(Topic.EmployeeName)) <> Lower(Trim(
      Concatenate(Global.ESS_UserContext_Employee_Firstname, " ", Global.ESS_UserContext_Employee_Lastname)
    ))
    &&
    Lower(Trim(Topic.EmployeeName)) <> Lower(Trim(
      Concatenate(Global.ESS_UserContext_Employee_Lastname, " ", Global.ESS_UserContext_Employee_Firstname)
    ))
    &&
    Lower(Trim(Topic.EmployeeName)) <> Lower(Trim(
      Concatenate(Global.ESS_UserContext_Employee_Lastname, ", ", Global.ESS_UserContext_Employee_Firstname)
    ))
```

### Example: Grouping Parallel Arrays by Key

```yaml
- kind: SetVariable
  id: set_grouped
  variable: Topic.grouped
  value: |-
    =ForAll(
      Distinct(Topic.flatTable, GroupKey) As grp,
      With(
        { entries: Filter(Topic.flatTable, GroupKey = grp.Value) },
        {
          GroupKey: grp.Value,
          StartDate: First(Sort(entries, DateField)).DateField,
          EndDate: Last(Sort(entries, DateField)).DateField,
          Total: Sum(entries, Value(AmountField)),
          Count: CountRows(entries)
        }
      )
    )
```

---

## Process for Creating a New Topic

### Step 1: Identify the Workday API

1. Determine the Workday SOAP service and operation needed
2. **Look up the operation's WSDL documentation** at `https://community.workday.com/sites/default/files/file-hosting/productionapi/{ServiceName}/v45.2/{OperationName}.html`
3. Verify EVERY element name in the request hierarchy — do NOT assume from other operations
4. Note the request element structure (Request_References vs Request_Criteria, child element names, ID types)

### Step 2: Create the XML Template

1. Write the `<workdayEntityConfigurationTemplate>` with correct element names from the WSDL
2. Define extractPaths for every field needed (ensure all return same count)
3. Keep it compact — no comments, no unnecessary Include_* flags

### Step 3: Create the Topic YAML

1. Choose the right pattern (READ, WRITE, CANCEL, Manager READ)
2. Write `modelDescription` under 1024 chars — WRITE topics need operational context, READ topics need concise field lists
3. Set `triggerQueries` with 5–12 sample utterances in plain, natural employee language
4. Build the action flow following the pattern
5. Use Adaptive Cards for all structured input/output
6. Apply all card rules (Column items, union type prevention, null text fallbacks)
7. Ensure single response per turn

### Step 4: Create the README

Document: purpose, SOAP operations, required ISU permissions, tenant config, install steps

### Step 5: Run the Checklist

Go through every item in the Mandatory Checklist section above. Fix any violations.

---

## Reference Links

- [Workday API Directory](https://community.workday.com/sites/default/files/file-hosting/productionapi/index.html)
- [CopilotStudioSamples GitHub](https://github.com/microsoft/CopilotStudioSamples/tree/main/EmployeeSelfServiceAgent/Workday)
- [Workday Extensibility Guide](https://learn.microsoft.com/en-us/copilot/microsoft-365/employee-self-service/workday-extensibility)
- [Copilot Studio Topics Docs](https://learn.microsoft.com/en-us/microsoft-copilot-studio/guidance/topics-overview)
- [Adaptive Cards Schema](https://adaptivecards.io/explorer/)
