# Sessions. Start Time and End Time

**Effective as of May 2, 2025**

This document outlines how we handle session start and end times across all three levels: **Salesforce database**, **API server**, and **frontend apps**. It **CANNOT** be used as a reference for any other similar cases.

## General Rule

Always interpret time data retrieved from **Salesforce** as **Pacific Time (PT)**, even though it is stored in **UTC** format.

## Saleforce

At the **Salesforce** level, all times are stored in **UTC format**. However, we **treat these times as Pacific Time (PT)**. Ignore `Z` at the end.

This is necessary because the Salesforce UI does **not** convert local time to UTC when users create sessions, meaning the time entered by users (in PT) is saved directly in UTC without adjustment.

<img src="./images/session_times_salesforce.png" alt="Image description">

## API Level

The **API server** also treats all times as **local time (PT)**. When creating or updating sessions, providing a time in UTC format does **not** trigger any transformation - it is saved exactly as provided.

For example, consider 2 requests:
- A request with `16:00:00.000Z`
- A request with `16:00`

Both result in the same value being saved in Salesforce: `16:00:00.000Z`.

**`POST` example**:

<img src="./images/session_times_post.png" alt="Image description">


**`PATCH` example**:

<img src="./images/session_times_patch.png" alt="Image description">


When returning time values from the API, we remove the trailing `Z` to avoid the impression that the time is in UTC.

**`GET` example**:

<img src="./images/session_times_get.png" alt="Image description">

## Frontend

In **frontend applications**, users should be clearly informed that all times are in **Pacific Time (PT)**. 

- **Sending time to the API (`POST`/`PATCH`)**:
Time selected by the user (in PT) should be sent to the API without any timezone conversion.

- **Receiving time from the API (`GET`)**:
Time returned by the API will not include a timezone designator (e.g., no trailing `Z`). It should be displayed directly to the user as PT. 

### Daylight Saving Time (DST)

Although Pacific Time (PT) includes both **PST (UTC−08:00)** and **PDT (UTC−07:00)** depending on the time of year, we do **not** shift session times for Daylight Saving Time.

This means:

- An 8:00 AM session always starts at 8:00 AM PT, regardless of whether it falls under standard time (PST) or daylight time (PDT).
- The frontend should **not** apply any DST-based transformation to the time.

##  Notes

**For Mulesoft (API server) developers:**

Add `-Duser.timezone=UTC` to your VM arguments when testing or developing the app locally. This ensures that dropping `Z` does not transform time. 


**Note regarding the Old Coach app and its Legacy Server:**

To maintain compatibility with the legacy Coach app (which cannot be updated), the API implements special handling for time values:

- **`POST` & `PATCH`**: No changes - times are saved exactly as provided.

- **`GET`**: The API legacy server subtracts **3 hours** from the stored time to match the legacy Coach app’s expectations.

> ⚠️ This adjustment is specific to the legacy app and should not be applied in other contexts.