# `DELETE /sessions/{sessionId}`

**Effective Date: May 15, 2025**

This document describes how the system deletes a Session and its related Attendance records.

The deletion is *conditional*: a Session can only be removed when there are **no Assessments** linked to it and **no Attendance records marked as attended** (`Attended__c = "true"`).

---

## Triggering API Action

| HTTP Method | Endpoint                | Description                                                                       |
| ----------- | ----------------------- | --------------------------------------------------------------------------------- |
| `DELETE`    | `/sessions/{sessionId}` | Attempts to delete the specified Session and related records: **tasks** and **attendances**. |

*Path Parameter*

* **`sessionId`** – the Salesforce ID of the `Session__c` record to delete.

---
## Deletion Workflow

```mermaid
flowchart TD
    A["Start (request received)"]
    A --> C["Extract sessionId"]
    C --> D{"Does the session exist?"}
    D -- No --> Z1["Return 'Session does not exist' (404)"]
    D -- Yes --> E["Query linked Assessments"]
    E --> F{"Any Assessments found?"}
    F -- Yes --> Z2["Return 'Session is active' (409)"]
    F -- No --> G["Query linked SCORES_Task__c records"]
    G --> H["Query linked Attendance__c records"]
    H --> I["Filter Attendance to those with Attended__c = 'true'"]
    I --> J{"Any attended records?"}
    J -- Yes --> Z2
    J -- No --> K["Build list of Task IDs + Attendance IDs + sessionId"]
    K --> L["Delete Tasks, Attendance records, and Session"]
    L --> M{"Deletion successful?"}
    M -- No --> Z3["Raise internal error (500)"]
    M -- Yes --> Z4["Return success:true (200)"]
    Z1 & Z2 & Z3 & Z4 --> End["End"]
```