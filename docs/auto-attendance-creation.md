# Automated Attendance Creation

**Effective Date: May 6, 2025**

This document outlines how attendance records are automatically created at the API level when specific events occur. The system supports implicit attendance generation in response to the following API actions:

- `POST /tasks`  
  A task is created with the type `"Take Attendance"`.

- `PATCH /tasks/{taskId}`  
  An existing task is updated to have the type `"Take Attendance"`.

- `GET /tasks/{taskId}`  
  A task with the type `"Take Attendance"` is retrieved.

- `POST /sessions?createAttendances=True`  
  A session is created with the `createAttendances` flag explicitly set to `True`.

- `GET /coach/{coachId}/teamseasons/{teamSeasonId}/sessions/{sessionId}/attendances`  
  Attendance records for a session are retrieved, triggering creation first.

## Attendance Records Creation

In general, endpoints listed above trigger the core attendance creation flow: `createAttendancesForSessionBasedOnTeamSeasonId` (located in `attendances.xml`)

This flow requires 3 key parameters saved in variables: 
- `vars.sessionId` 
- `vars.teamSeasonId`
- `vars.sessionDate`

The process consists of the following steps:

1. **Retrieve Enrollment**: The system queries the `Enrollment__c` object to find all students enrolled in the teamseason: 
```sql
SELECT 
    Contact__c
FROM 
    Enrollment__c 
WHERE 
    Team_Season__c = ':teamSeasonId' 
    AND Start_Date__c <= :sessionDate
    AND End_Date__c >= :sessionDate
GROUP BY 
    Contact__c
```
	
2. **Retrieve Existing Attendance Records**: The system queries the `Attendance__c` object to find existing records:

  
```sql

SELECT 
    Id,
    Contact__r.Name,
    Contact__c,
    Attended__c 
FROM 
    Attendance__c 
WHERE 
    Session__c = ':sessionId' 
AND Contact_Record_Type__c = 'SCORES Student'
```

3. **Duplicate Check**: The system filters the list of students retrieved in step 1 based on the existing records from step 2:

```DataWeave
%dw 2.0
output application/json
var exists = vars.retrieve map (item, index) -> item.StudentId
---
vars.upsert filter (item) -> not (exists contains item.Contact__c)
```

4. **Create Attendance Records**: The system performs an upsert operation using the filtered list from step 3, generating attendance records for students with active enrollments who don’t already have records.

5. **Retrieve All Attendance Records**: All attendance records are retrieved with the student's name. 

```sql
SELECT 
    Id,
    Contact__r.Name,
    Contact__c,
    Attended__c 
FROM 
    Attendance__c 
WHERE 
    Session__c = ':sessionid' 
    AND Contact_Record_Type__c = 'SCORES Student'
```