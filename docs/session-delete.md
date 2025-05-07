# `DELETE /sessions/{sessionId}`

**Effective Date: May 7, 2025**

This document describes how the system deletes a Session and its related Attendance records.

The deletion is *conditional*: a Session can only be removed when there are **no Assessments** linked to it and **no Attendance records marked as attended** (`Attended__c = "true"`).

---

## Triggering API Action

| HTTP Method | Endpoint                | Description                                                                       |
| ----------- | ----------------------- | --------------------------------------------------------------------------------- |
| `DELETE`    | `/sessions/{sessionId}` | Attempts to delete the specified Session and its non‑attended Attendance records. |

*Path Parameter*

* **`sessionId`** – the Salesforce ID of the `Session__c` record to delete.

---
## Deletion Workflow

```mermaid
flowchart TD
    A["Start (request received)"] 
    A --> C["Extract sessionId"]
    C --> D{"Does the session exist?"}
    D -- No --> Z1["Return 'Session not found' (success:false)"]
    D -- Yes --> E["Check for linked Assessments"]
    E --> F["Check for Attendance records"]
    F --> G["Build deletion list (attendance IDs)"]
    G --> H{"No Assessments and no Attendance records marked with 'true'?"}
    H -- No --> Z2["Return 'Session is active' (success:false)"]
    H -- Yes --> I["Delete attendance records + session"]
    I --> J{"Delete successful?"}
    J -- No --> Z3["Raise internal error"]
    J -- Yes --> Z4["Return success:true"]
    Z1 & Z2 & Z3 & Z4 --> K["End"]
```