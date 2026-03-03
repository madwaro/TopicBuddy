# Request time off

Allows an employee to submit a time off request in Workday directly from Copilot Studio. The topic fetches the employee's current leave balances, presents a form to select type, dates, and hours, and submits the request to Workday via the Absence Management API.

## Features

- Displays current leave balances (vacation, sick leave, floating holiday) before the form
- Supports multi-day requests via a date-loop that builds a comma-separated date list
- Validates that end date is not before start date
- Handles leave types not listed with a graceful redirect to Workday
- Humanizes Workday error messages using AI before surfacing them to the employee
- On failure, offers retry that loops back to the form

## Trigger phrases

- "Request time off"
- "I need to take some time off"
- "Submit a vacation request"
- "I need to request sick leave for next week"
- "Book time off from March 10 to March 14"

## Files

| File | Description |
|---|---|
| `topic.yaml` | Main AdaptiveDialog topic |
| `cards/step1-request-form.json` | Standalone card: leave balance + request form |
| `cards/step2-success.json` | Standalone card: submission confirmation |
| `msdyn_HRWorkdayHCMEmployeeGetVacationBalance.xml` | Workday API template — get leave balances |
| `msdyn_HRWorkdayAbsenceEnterTimeOff_MultiDay.xml` | Workday API template — submit time off request |

## Workday APIs

| Scenario name | Service | Operation |
|---|---|---|
| `msdyn_HRWorkdayHCMEmployeeGetVacationBalance` | Human_Resources v42.0 | Get_Workers (with leave balance) |
| `msdyn_HRWorkdayAbsenceEnterTimeOff_MultiDay` | Absence_Management v42.0 | Enter_Time_Off_Request |

> **Note:** The XPath extraction paths in `msdyn_HRWorkdayHCMEmployeeGetVacationBalance.xml` are best-guess estimates. Validate them against a live Workday `Get_Workers` response from your tenant.

## Conversation flow

```
Employee: "I need to request vacation next week"
  ↓
[Fetch leave balances from Workday]
  ↓
STEP 1 — Request form card
  Shows current balances, collects: type, start date, end date, hours/day
  ↓
[Validate: end date ≥ start date]
[Build comma-separated date list via loop]
  ↓
[Submit Enter_Time_Off_Request to Workday]
  ↓
  Success → STEP 2 — Confirmation card (type, date range, total hours)
  Failure → Friendly AI error message → offer retry
```

## Configuration

Two tables in the topic are intended to be customized per tenant:

**`Topic.LeaveTypeConfig`** — maps natural language keywords to Workday Time Off Type IDs:
```
Vacation_Hours       → vacation, annual, pto, holiday pay
Floating_Holiday_Hours → floating, floater, float day
Sick_Hours           → sick, illness, medical, unwell, not feeling well
```

**`Topic.PlanConfig`** — maps Workday Plan IDs to display names shown in the balance section:
```
FH_USA              → Floating holiday
ABSENCE_PLAN-6-159  → Sick leave
ABSENCE_PLAN-6-158  → Vacation
```
Update `PlanID` values to match the absence plan IDs in your Workday tenant.

## Dependencies

- Microsoft Copilot for Employee Self-Service solution installed
- `msdyn_copilotforemployeeselfservice.topic.WorkdaySystemGetCommonExecution` topic available
- Employee context available via `Global.ESS_UserContext_Employee_Id`
- Workday security domain permissions for `Human_Resources` (Get_Workers) and `Absence_Management` (Enter_Time_Off_Request)
- Both XML files uploaded to the Workday configuration in Power Platform
