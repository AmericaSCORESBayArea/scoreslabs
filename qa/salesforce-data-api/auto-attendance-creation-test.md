# QA Test: Automated Attendance Creation

**Date**: May 6, 2025

This series of tests verifies the functionality of [automated attendance creation](./../../docs/auto-attendance-creation.md).

---

## 1. `POST /tasks`

### Test A: Create Task with `TaskType = HR Requirement`

<img src="./images/auto-attendance-creation-test_1.png" alt="HR Requirement task creation" height="500px">

**Expected:** No attendance records are created.  
**Result:** Confirmed. ✅

---

### Test B: Create Task with `TaskType = Take Attendance`

<img src="./images/auto-attendance-creation-test_2.png" alt="Take Attendance task creation" height="500px">

**Expected:** Attendance records are automatically created and returned.  
**Result:** Confirmed. ✅

---

## 2. `PATCH /tasks/{taskId}`

### Test A: Change Task Type from "HR Requirement" to "Take Attendance"

<img src="./images/auto-attendance-creation-test_3.png" alt="Patch to Take Attendance" height="500px">

**Expected:** Attendance records are created and returned.  
**Result:** Confirmed. ✅

---

### Test B: Change Task Type from "Take Attendance" to "HR Requirement"

<img src="./images/auto-attendance-creation-test_4.png" alt="Patch back to HR Requirement" height="500px">

**Expected:** No new attendance records are created or returned.  
**Result:** Confirmed. ✅ 
⚠️ **Note:** Attendance records previously created still exist in the database.

---

## 3. `GET /tasks/{taskId}`

### Test A: Retrieve Task with `TaskType = HR Requirement`

<img src="./images/auto-attendance-creation-test_5.png" alt="GET HR Requirement task" height="500px">

**Expected:** No attendance records are attached.  
**Result:** Confirmed. ✅ 

---

### Test B: Retrieve Task with `TaskType = Take Attendance`

<img src="./images/auto-attendance-creation-test_6.png" alt="GET Take Attendance task" height="500px">

**Expected:** Attendance records are returned.  
**Result:** Confirmed. ✅ 

---

### Test C: Update Attendance Record and Retrieve Task

Update the first record’s `Attended` field to `true` using Workbench, then retrieve the task again.

<img src="./images/auto-attendance-creation-test_7.png" alt="Update attendance in Workbench">
<img src="./images/auto-attendance-creation-test_8.png" alt="GET task after update" height="500px">

**Expected:** Updated attendance records are returned, with `Attended = true`.  
**Result:** Confirmed. ✅ 

---

### Test D: Delete Attendance Record and Retrieve Task

Delete the first attendance record (`a0lcX000000fta2QAA`) and retrieve the task again.

<img src="./images/auto-attendance-creation-test_9.png" alt="Delete attendance record">
<img src="./images/auto-attendance-creation-test_10.png" alt="GET task after deletion" height="500px">

**Expected:** The deleted attendance record is recreated.  
**Result:** Confirmed. ✅ 

---

## 4. `POST /sessions?createAttendances=True`

### Test A: Create Session with `createAttendances=False`

<img src="./images/auto-attendance-creation-test_11.png" alt="Create session without attendances" height="320px">

**Expected:** No attendance records are created.  
**Result:** Confirmed. ✅ 

---

### Test B: Create Session with `createAttendances=True`

<img src="./images/auto-attendance-creation-test_12.png" alt="Create session with attendances" height="500px">

**Expected:** Attendance records are created.  
**Result:** Confirmed. ✅ 

---

## 5. `GET /coach/{coachId}/teamseasons/{teamSeasonId}/sessions/{sessionId}/attendances`

### Test A: Retrieve Existing Attendance Records

<img src="./images/auto-attendance-creation-test_13.png" alt="GET session attendances" height="500px">

**Expected:** Existing attendance records are returned.  
**Result:** Confirmed. ✅ 

---

### Test B: Enroll New Student and Retrieve Attendances Again

1. Create a new contact.
2. Enroll the contact into the team season.
3. Retrieve attendances again.

<img src="./images/auto-attendance-creation-test_14.png" alt="New student enrollment" height="500px">
<img src="./images/auto-attendance-creation-test_15.png" alt="New student enrollment continued" height="500px">
<img src="./images/auto-attendance-creation-test_16.png" alt="Attendance record for new student" height="500px">

**Expected:** A new attendance record is created for the newly enrolled student.  
**Result:** Confirmed. ✅ 