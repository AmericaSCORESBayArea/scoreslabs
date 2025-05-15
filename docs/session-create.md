# `POST /sessions`

**Effective Date: May 15 2025**  

Creates a new `Session__c` record in Salesforce and, if requested (and by default), an accompanying **“Take Attendance” `SCORES_Task__c`**.

A task is created *only if*:

1. the request body contains `CoachId`, **and**  
2. the query parameter&nbsp;`createAttendanceTask` is omitted **or** set to `true`.

---

## Triggering API Action

| HTTP&nbsp;Method | Endpoint    | Description                                                                                     |
|------------------|-------------|-------------------------------------------------------------------------------------------------|
| `POST`           | `/sessions` | Creates a `Session__c` and (optionally) an attendance-task assigned to the specified coach. |

### Query Parameters

| Name                  | Type / Allowed Values | Default | Purpose                                         |
|-----------------------|-----------------------|---------|-------------------------------------------------|
| `createAttendanceTask`| `true` \| `false`     | `true`  | Whether to also create a **Take Attendance** task. |

### JSON Request Body

| Field          | Type (example)     | Required | Salesforce Mapping        |
|----------------|--------------------|----------|---------------------------|
| `TeamSeasonId` | `"a1B…"`           | **Yes**  | `Team_Season__c`          |
| `SessionDate`  | `"2025-05-21"`     | **Yes**  | `Session_Date__c`         |
| `SessionStart` | `"18:00"`          | No       | `Session_Start__c`        |
| `SessionEnd`   | `"19:30"`          | No       | `Session_End__c`          |
| `SessionTopic` | `"Dribbling"`      | No       | `Session_Topic__c`        |
| `CoachId`      | `"003…"`           | No\*     | (needed for task creation)|

\*Only required if you want the system to create the task.

---

## Creation Workflow

```mermaid
flowchart TD
    A["Start (request received)"]
    A --> B["Extract body & query params"]
    B --> C["Build Session__c payload"]
    C --> D["Create Session__c"]
    D --> E{"Creation successful?"}
    E -- No --> Z1["Raise internal error (500)"]
    E -- Yes --> F["Store SessionId"]
    F --> G{"createAttendanceTask=true AND CoachId present?"}
    G -- No --> H["Skip task creation"]
    G -- Yes --> I["Build SCORES_Task__c payload"]
    I --> J["POST /tasks"]
    J --> K{"Task created?"}
    K -- No --> Z2["Surface task error (500)"]
    K -- Yes --> L["Capture TaskId and Attendance Records"]
    H & L --> Z3["Return success (201)"]
    Z1 & Z2 & Z3 --> End["End"]