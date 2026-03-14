# Live Call & Leads Call — Technical Flow Document

---

## 1. Module Overview

| Module | Type | Description |
|---|---|---|
| **Live Call** | Single page | Two sub-menus: Fresh Call and Used Calls |
| **Leads Call** | Separate page | Lead records, activities, and follow-up management |

**Live Call** maintains shared company context across both sub-menus. Switching menus clears all filters, reminder state, search, and triggers a fresh data fetch.

---

## 2. Boot Flow

On page load → fetch company list → auto-select first company → load Fresh Call data.

**`POST userService/getAllCompanyList`**

```json
// Request
{ "signature": "user_auth_token" }

// Response
{
  "data": {
    "data": [
      { "id": "12345", "name": "Example Corp", "type": "Primary" }
    ]
  }
}
```

Selected company context (reused in all subsequent API calls):
```json
{ "companyId": "12345", "companyName": "Example Corp", "companyType": "Primary" }
```

---

## 3. Fresh Call

Records that have **never been called**. `call_logs` is always empty.  
Fetched all at once — no pagination. Filtering is **client-side only**.

**`POST callLogsService/get-call-logs-leads`**

```json
// Request
{
  "signature": "user_auth_token",
  "companyId": "12345",
  "companyType": "Primary"
}

// Response — stored as response.data.data
[
  {
    "id": "record_1",
    "name": "Customer Name",
    "mobileNumber": "9876543210",
    "city": "Mumbai",
    "district": "Mumbai Suburban",
    "state": "Maharashtra",
    "country": "India",
    "call_logs": [],
    "createdAt": "2026-03-14T10:30:00.000Z"
  }
]
```

### Client-Side Filters

| Filter | Behavior |
|---|---|
| City / District / State / Country | Exact match |
| Start Date / End Date | Compared against `createdAt` |

> **Note:** Outcome/type filter is **not applicable** to Fresh Calls. No calls have been made, so there is no outcome history. Outcome filtering belongs to Used Calls only.

---

## 4. Used Calls

Records with **existing call history** in `call_logs`.  
Filtering and pagination are **fully backend-handled**.

**`POST callLogsService/get-used-call-logs`**

```json
// Request
// Required: signature, companyId, companyType, page, limit
// All others are optional — send only when selected by user
{
  "signature": "user_auth_token",
  "companyId": "12345",
  "companyType": "Primary",
  "page": 1,
  "limit": 10,
  "country": "India",
  "state": "Maharashtra",
  "district": "Mumbai Suburban",
  "city": "Mumbai",
  "type": "Call Later",
  "start_date": "2026-03-01",
  "end_date": "2026-03-14",
  "reminder_preset": "today",
  "reminder_date_from": "2026-03-14",
  "reminder_date_to": "2026-03-14",
  "search": "john"
}

// Response — records at response.data.data.data, pagination at response.data.data.pagination
{
  "data": {
    "data": [
      {
        "id": "used_1",
        "name": "Customer Name",
        "mobileNumber": "9876543210",
        "city": "Mumbai",
        "district": "Mumbai Suburban",
        "state": "Maharashtra",
        "country": "India",
        "call_logs": [
          {
            "type": "Call Later",
            "date": "2026-03-14T10:30:00.000Z",
            "note": "Requested callback",
            "reminder_date": "2026-03-15T11:00:00.000Z"
          }
        ],
        "updatedAt": "2026-03-14T10:35:00.000Z"
      }
    ],
    "pagination": {
      "page": 1,
      "totalPages": 5,
      "total": 50,
      "limit": 10,
      "hasNextPage": true,
      "hasPrevPage": false
    }
  }
}
```

### call_logs History Display Rule

`call_logs[].type` stores the **child value** (not a display label). Always map it to a human-readable label before showing it.

1. Read `call_logs[].type`  
2. Match the child `value` in the outcome table  
3. Show the child `label` — optionally group under `parent`

---

## 5. Call Outcome Reference

All outcomes follow a **Parent → Child** hierarchy. The child `value` is stored in `call_logs.type`.

| Parent | Stored Value (`type`) | Display Label |
|---|---|---|
| Add To Zyvo | `Lead is qualified & relevant` | Lead is qualified & relevant |
| Call Connected | `Connected` | Call Connected |
| No Connection | `Not Picked` | Not Picked |
| No Connection | `Not Reachable / Switch Off` | Not Reachable / Switch Off |
| No Connection | `Invalid Number` | Invalid Number |
| Call Disconnected | `Disconnected - After Opening` | After Opening |
| Call Disconnected | `Disconnected - During Conversation` | During Conversation |
| Call Later ⚠️ | `Call Later` | Schedule a Callback |
| Reference | `Reference` | Gave a reference |
| Not Interested | `Not Interested - Distance Issue` | Distance Issue |
| Not Interested | `Not Interested - High Fee` | High Fee |
| Not Interested | `Not Interested - Govt. College` | Govt. College |
| Not Interested | `Not Interested - Admitted Outside` | Admitted Outside |
| Not Interested | `Not Interested - Others` | Others |

> ⚠️ **Call Later** — reminder date & time is **mandatory** when this outcome is selected.  
> **Not Interested** — exact child reason must be selected, not just the parent.

---

## 6. Used Call Filtering

### Filter Options API

Filter dropdown values are fetched from the backend and are **scoped to actual used-call data** for that company. If a company's used-call records exist only for India and England, the country filter will show only those two countries — never a generic master list.

**`POST callLogsService/get-used-call-filter-options`**

```json
// Request
{
  "signature": "user_auth_token",
  "companyId": "12345",
  "companyType": "Primary"
}

// Response — all values derived from existing used-call records only
{
  "data": {
    "countries": ["India", "England"],
    "states": ["Maharashtra", "Greater London"],
    "districts": ["Mumbai Suburban"],
    "cities": ["Mumbai", "London"],
    "outcomes": ["Connected", "Call Later", "Not Interested - High Fee"]
  }
}
```

> **Current behavior:** Location filters (`countries`, `states`, `districts`, `cities`) are backend-driven from this API.  
> **Call outcome filter (`type`)** is currently built from the full `REFERENCE_OPTIONS` child list on the client.  
> **Future requirement:** Outcomes should also be backend-scoped so only outcomes present in the company's used-call history appear as filter options.

---

## 7. Reminder System

### Used Call Reminders

Quick-select reminder chips available on the Used Calls view:

| Preset | Trigger Behavior | Backend Filter Logic |
|---|---|---|
| `overdue` | Immediate refetch | `reminder_date < today` |
| `today` | Immediate refetch | `reminder_date = today` |
| `tomorrow` | Immediate refetch | `reminder_date = tomorrow` |
| `custom` | Shows date range picker; refetch fires on both dates selected | `reminder_date between from and to` |

```json
// Custom reminder fields sent in used-call request
{
  "reminder_preset": "custom",
  "reminder_date_from": "2026-03-14",
  "reminder_date_to": "2026-03-20"
}
```

### Leads Follow-up Filter (separate system)

The Leads Call page manages follow-up dates client-side via `follow_up_date` on each lead record.

| Value | Logic |
|---|---|
| `all` | No filter |
| `today` | `follow_up_date` = today |
| `tomorrow` | `follow_up_date` = tomorrow |
| `overdue` | `follow_up_date` < today |
| `custom` | `follow_up_date` = selected date |
| `none` | `follow_up_date` is null |

---

## 8. Search (Used Calls Only)

Search is **backend-driven**, searching across name, mobile number, city, state, and district.

- User types → stored in `tempSearchQuery`
- On Enter or Search button → promoted to `searchQuery`, refetch fires
- Clearing the field → triggers refetch with empty search

```json
// Add to standard used-call request
{ "search": "john mumbai" }
```

---

## 9. Call Initiation Flow

### Pre-call checks (in order)
1. Is a call already active in this tab? → **Block**
2. Is another browser tab holding the call lock? → **Block with error message**
3. Acquire call lock for current tab → proceed

### Call Record Built Before Socket Emit

- Resolves `name` from `record.response["Full Name"]` or `record.name`
- Normalizes `mobileNumber`
- Reads last 2 `call_logs` entries → groups by date → maps to activity items using outcome labels

```json
// Built call record
{
  "id": "record_1",
  "mobileNumber": "9876543210",
  "name": "Customer Name",
  "activities": [
    {
      "date": "2026-03-14",
      "activities": [
        {
          "title": "Schedule a Callback",
          "time": "2026-03-14T10:30:00.000Z",
          "description": "Requested callback",
          "reminderDate": "2026-03-15T11:00:00.000Z"
        }
      ]
    }
  ]
}
```

### Socket Event

```json
{
  "event": "startCall",
  "payload": {
    "userId": "current_user_id",
    "data": { /* built call record above */ }
  }
}
```

### Redux State After Call Start

```json
{
  "activeCall": { /* call record */ },
  "isAutoCall": false,
  "selectedCompany": { "companyId": "12345", "companyType": "Primary" },
  "isLeadsMode": false
}
```

---

## 10. Auto Call

Dials through the **entire filtered list sequentially** without manual intervention.

1. User taps **Start Auto Call** → `isAutoCall = true`, index resets to `0`
2. First record called immediately
3. After each call ends → index increments → next record called automatically
4. No next record → `isAutoCall = false`, `autoCallStopped = true`, `refreshCallsAfterSubmission` event fires

> Switching company or menu is blocked while auto-call is active.

---

## 11. Modals

### Call History Modal

- **Trigger:** User clicks **View Logs** on any used-call record
- **Data received:** Full `call_logs` array for that record

```json
[
  {
    "type": "Connected",
    "date": "2026-03-14T10:30:00.000Z",
    "note": "Customer interested",
    "reminder_date": "2026-03-15T11:00:00.000Z"
  }
]
```

### In-Call Screen

Rendered via shared Redux `activeCall` state. User inputs:

| Field | Required | Notes |
|---|---|---|
| Call Outcome | Yes | Hierarchical — select parent group then child value |
| Notes | Yes | Free-text description of call |
| Reminder Date & Time | Conditional | **Mandatory** for `Call Later` outcome |
| Pipeline Stage | Leads only | Update the lead's current pipeline stage |

---

## 12. Leads Call

A **separate page** — does not share the Live Call tab system or its state.

### Lead Fetch

**`POST leadService/getAllLeadsResponse`**

```json
// Request
{
  "signature": "user_auth_token",
  "company_id": "12345",
  "company_type": "Primary",
  "form_id": "form_1",
  "start_date": "2026-03-01",
  "end_date": "2026-03-31",
  "user_id": "0",       // "0" = all users; specific ID = filter by assigned user
  "limit": 100000
}

// Response
{
  "data": {
    "data": [
      {
        "id": "lead_row_1",
        "lead_id": "123456789012",
        "mobile_no": "9876543210",
        "pipeline_char": "New Lead",
        "pipeline_id": "pipeline_1",
        "lead_source": "Campaign",
        "lead_source_child": "Facebook",
        "assigned_to": 101,
        "follow_up_date": "2026-03-20T10:00:00.000Z",
        "response": {
          "Full Name": "Jane Doe",
          "Mobile No": "9876543210",
          "Email": "jane@example.com"
        },
        "createdAt": "2026-03-14T09:00:00.000Z"
      }
    ],
    "transferLeadIds": ["lead_row_1"]
  }
}
```

### Lead Activity Fetch

**`POST leadService/getLeadActivities`**

```json
// Request
{ "signature": "user_auth_token", "lead_id": "lead_row_1" }

// Response
{
  "data": {
    "data": [
      {
        "type": "Call",
        "date": "2026-03-14T10:30:00.000Z",
        "note": "Follow-up scheduled",
        "reminder_date": "2026-03-15T11:00:00.000Z"
      }
    ]
  }
}
```

### Post-Call Submission (Leads — 3 APIs called together)

**Add Note — `POST leadService/addLeadnote`**
```json
{
  "signature": "user_auth_token",
  "lead_id": "lead_row_1",
  "company_id": "12345",
  "company_type": "Primary",
  "note": "Customer interested, follow up tomorrow"
}
```

**Update Pipeline — `POST leadService/updateLeadResponsePipeline`**
```json
{
  "signature": "user_auth_token",
  "pipeline": "Qualified",
  "lead_id": "lead_row_1"
}
```

**Add Follow-up — `POST leadService/addLeadFollowUp`**
```json
{
  "signature": "user_auth_token",
  "lead_id": "lead_row_1",
  "company_id": "12345",
  "company_type": "Primary",
  "follow_up_date": "2026-03-15T11:00:00.000Z"
}
```

### Assignment & Duplicate Handling

- **Auto-assignment:** Backend assigns the lead to the user with the lowest active lead count.
- **Manual assignment:** Directly set during creation; recorded in lead history.
- **Duplicate check:** Before creating a lead, backend checks for same mobile number + company + form combination. If duplicate found → returns error with existing assignee name and creation date.

---

## 13. API Quick Reference

| API | Purpose | Filtering |
|---|---|---|
| `userService/getAllCompanyList` | Company list | — |
| `callLogsService/get-call-logs-leads` | Fresh call records | Client-side |
| `callLogsService/get-used-call-logs` | Used call records (paginated) | Backend |
| `callLogsService/get-used-call-filter-options` | Used call filter values (data-scoped) | — |
| `leadService/getAllLeadsResponse` | Lead records | Server + client |
| `leadService/getLeadActivities` | Lead activity history | — |
| `leadService/addLeadnote` | Save call note to a lead | — |
| `leadService/updateLeadResponsePipeline` | Update lead pipeline stage | — |
| `leadService/addLeadFollowUp` | Set follow-up / reminder on a lead | — |

---

## 14. Redux State Reference

```json
{
  "liveCall": {
    "activeCall": null,           // Current active call object or null
    "isAutoCall": false,          // Auto-call mode is running
    "autoCallStopped": false,     // User manually stopped auto-call
    "selectedCompany": {},        // Active company context
    "isLeadsMode": false,         // true when calling from Leads Call page
    "callDisconnectedData": {}    // Data snapshot from last disconnected call
  }
}
```

---

## 15. End-to-End Flow Summary

### Live Call — Used Calls

```
Open page
  → Fetch companies (userService/getAllCompanyList)
  → Auto-select first company, load Fresh Call data

Switch to Used Calls
  → Fetch used-call records (get-used-call-logs)
  → Fetch filter options (get-used-call-filter-options)

Apply filters / reminder / search
  → Refetch with filter params → paginated results

Tap call button
  → Pre-call checks (active call? tab lock?)
  → Build call record from call_logs + outcome labels
  → Emit startCall socket event
  → Update Redux activeCall state

Complete call
  → Select outcome + add notes + set reminder (if Call Later)
  → Submit → refreshCallsAfterSubmission fired
  → List refetches with current page + filter context
```

### Leads Call

```
Open Leads Call page (separate page)
  → Select company + lead form
  → Fetch lead records (getAllLeadsResponse)
  → Fetch activity per lead (getLeadActivities)

Apply follow-up filter (today / overdue / tomorrow / custom / none)
  → Client-side filter on follow_up_date

Tap call
  → Same socket startCall flow, isLeadsMode = true

Complete call
  → addLeadnote + updateLeadResponsePipeline + addLeadFollowUp
  → Refresh leads list
```

---

## 16. Mobile Integration Notes

| Area | Guidance |
|---|---|
| Fresh Call | Treat as a simple flat list; apply filters client-side |
| Used Calls | Always server-filtered and paginated; send only active filter params |
| Filter options | Trust backend-provided values; do not derive client-side for used calls |
| Outcome display | Map stored `type` value to label using outcome reference table |
| Call Later | Always enforce reminder date input before allowing submission |
| Socket | Manage `startCall` event; handle tab-lock conflicts |
| Refresh | Listen for `refreshCallsAfterSubmission` to reload list after call ends |
| Leads | Maintain separate state and API flow from Live Call |

---
