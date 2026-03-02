# Workday Employee Cancel Time Off

## Overview

This topic enables employees to cancel upcoming time off requests through a conversational interface with their Copilot Studio agent. Employees can view their upcoming requests, select one to cancel, and confirm the cancellation — all without leaving the chat.

## Features

- Fetch and display all upcoming time off requests (pending and approved)
- Let the employee select a request to cancel from a list
- Show a confirmation step before submitting the cancellation
- Cancel the time off request directly in Workday
- Friendly error handling using AI-generated messages

## Trigger phrases

- "Cancel my time off request"
- "I need to cancel my vacation"
- "Retract my sick leave request"
- "Cancel my PTO for next week"
- "Cancel my time off"

## Files

| File | Description |
|------|-------------|
| `topic.yaml` | Copilot Studio topic definition with conversation flow |
| `msdyn_HRWorkdayAbsenceGetTimeOffRequests.xml` | Workday API request template for fetching upcoming time off requests |
| `msdyn_HRWorkdayAbsenceCancelTimeOff.xml` | Workday API request template for cancelling a time off request |
| `cards/step1-select-request.json` | Adaptive Card — list of upcoming requests for selection |
| `cards/step2-confirm-cancellation.json` | Adaptive Card — confirmation step before cancelling |
| `cards/step3-cancel-success.json` | Adaptive Card — cancellation confirmation receipt |

## Workday APIs used

| API | Purpose |
|-----|---------|
| `Get_Time_Off_Requests` | Retrieves upcoming time off requests for the employee |
| `Cancel_Time_Off_Request` | Submits the cancellation to Workday |

## Flow overview

```
┌─────────────────────────────────────────────────────────────┐
│                    User triggers topic                       │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│         Fetch upcoming time off requests from Workday        │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│         Display request list — employee selects one          │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│           Show confirmation card with request details        │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│              Submit cancellation to Workday                  │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│              Confirm cancellation to employee                │
└─────────────────────────────────────────────────────────────┘
```

## Configurations

Environment makers need to configure the following in the topic:

| Configuration | Description | Location in topic |
|---------------|-------------|-------------------|
| **Workday URL** | Set your organization's Workday tenant URL | `set_workday_url` variable at the top of the topic |
| **Leave type labels** | Update the `Switch` expressions that map internal type IDs (e.g. `Vacation_Hours`) to display names | `build_requests_table` variable |

## Dependencies

- **Employee context**: `Global.ESS_UserContext_Employee_Id` must be set by the Employee Self-Service agent before this topic is invoked
- **Shared execution topic**: `msdyn_copilotforemployeeselfservicehr.topic.WorkdaySystemGetCommonExecution` must be available in the agent
