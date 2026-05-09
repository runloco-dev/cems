# CEMS — Conference and Event Management Software
## Design Specification
**Date:** 2026-05-08
**Status:** Approved

---

## 1. Problem Statement

A federal training facility operates hotels, conference facilities, classrooms, and structured training programs on a multi-building campus. The facility serves federal employees, contractors, civilian trainees, instructors, and external groups booking the venue for their own events. No unified system manages enrollment, scheduling, lodging, and access control — these are handled in silos or manually. CEMS replaces that with a single platform on Salesforce Government Cloud Plus.

---

## 2. Platform and Compliance

**Salesforce Government Cloud Plus (FedRAMP High)**

| Cloud / Product | Purpose |
|---|---|
| Service Cloud | Case management, service console, agent workspace |
| Sales Cloud | Agency/sponsor accounts, event booking pipeline |
| Experience Cloud | Self-service enrollment portal for trainees and agencies |
| Salesforce Scheduler | Room, instructor, and facility calendar management |
| Agentforce | Three domain agents — enrollment, scheduling, badging |
| Shield Platform Encryption | CUI field-level encryption (FedRAMP High requirement) |
| Shield Event Monitoring | Full API and login audit, feeds agency SIEM |
| Field Audit Trail | 10-year retention on sensitive field changes |

**Compliance scaffolding is deployed in Phase 1 and applies to all subsequent phases.**

Authentication:
- MFA enforced org-wide
- PIV/CAC via Salesforce Identity for admin and restricted-area users
- Experience Cloud portal: standard MFA (TOTP/passkey); PIV/CAC required only for restricted-tier access
- SSO to agency IdP: open item — see Section 8

Data classification:
- CEMS handles CUI and below only
- Salesforce GovCloud+ is not authorized for classified data
- Clearance tier is stored as a reference field (None / Public Trust / Secret / TS / TS/SCI) — not the authoritative adjudication record
- Classified program content and adjudication records remain off-platform

---

## 3. People Model

All individuals are represented as **Contacts** linked to **Accounts** (agencies or organizations).

**Contact key fields:**

| Field | Type | Notes |
|---|---|---|
| `Person_Type__c` | Picklist | Federal Employee, Contractor, Civilian Trainee, Instructor, External Attendee |
| `Clearance_Tier__c` | Picklist | None, Public Trust, Secret, TS, TS/SCI — reference only, encrypted |
| `PIV_CAC_ID__c` | Encrypted Text | Federal credential identifier, Shield-encrypted |
| `Agency__c` | Lookup (Account) | Sponsoring agency or employer |
| `Is_On_Campus__c` | Checkbox | Updated by badging workflow and ACS events |

**Account types:**
- Sponsoring Agency (federal agencies nominating trainees)
- External Event Organization (groups booking the facility)
- Campus Vendor / Partner (supporting roles)

**Salesforce Users** represent CEMS staff, instructors with system access, and security personnel. External trainees and agency nominators access the system via Experience Cloud (no Salesforce license required).

---

## 4. Data Model

### 4.1 Training Programs

```
Training_Program__c
  — Name, description, prerequisites, clearance tier required, max duration
  └── Course_Offering__c
        — Start date, end date, max enrollment, status, location (Room lookup)
        — Waitlist enabled flag
        └── Session__c
              — Date, start/end time, topic, instructor (User lookup), Room_Booking__c
        └── Enrollment__c
              — Contact, status (Nominated / Enrolled / Waitlisted / Completed / Cancelled)
              — Waitlist position, nominated by (Account)
              └── Assessment__c
                    — Session or offering level, score, pass/fail, grader
              └── Certificate__c
                    — Issue date, certificate number, expiry (if applicable)
```

### 4.2 Facility

```
Campus__c
  — Name, address, primary contact
  └── Building__c
        — Name, type (Hotel | Classroom | Conference Center | Admin | Restricted)
        — Access tier required (Standard Campus | Restricted Area | PIV/CAC Required)
        └── Room__c
              — Name, capacity, type (Classroom | Conference Room | Hotel Room | Lab)
              — Access tier required (overrides Building if more restrictive)
              — AV equipment, amenities (text/checklist)
              — Is active flag
```

Hotel rooms are `Room__c` records under a Building with `Type__c = 'Hotel'`. No separate hierarchy — keeps the facility model unified and scheduling logic consistent.

### 4.3 Scheduling and Events

| Object | Purpose |
|---|---|
| `Room_Booking__c` | Reserves a Room for a Session or Facility_Event__c; enforces no double-booking |
| `Facility_Event__c` | External group booking — links to Account, has date range, headcount, room assignments, lodging block |
| Salesforce Scheduler `Service Resource` | Instructors as schedulable resources with availability |
| Salesforce Scheduler `Service Territory` | Campus buildings as territories |

`Room_Booking__c` fields: Room__c, Start_DateTime__c, End_DateTime__c, Booked_For_Session__c, Booked_For_Event__c (one or the other populated), Status__c.

Double-booking prevention: before-insert/update trigger validates no overlapping active bookings for the same room.

### 4.4 Lodging

```
Lodging_Reservation__c
  — Contact__c (who is staying)
  — Room__c (hotel room — Room__c where Building.Type = Hotel)
  — Check_In_Date__c, Check_Out_Date__c
  — Enrollment__c (if staying for a training program) OR Facility_Event__c (if attending an event)
  — Status__c (Requested | Confirmed | Checked In | Checked Out | Cancelled)
  — Special_Requirements__c
```

Availability is managed via query on Lodging_Reservation__c for date range conflicts on the same room.

### 4.5 Badging

| Object | Purpose |
|---|---|
| `Badge_Request__c` | Workflow record: Requested → Vetted → Approved → Issued / Denied |
| `Badge__c` | Issued badge — type, tier, issue date, expiry, status, Contact lookup |
| `Clearance_Reference__c` | Mocked clearance confirmation: Contact, tier, confirming admin, date verified |
| `Access_Log__c` | Read-only landing zone for ACS Platform Event subscriber |

**Badge tiers:**
- Standard Campus — general access, all non-restricted buildings
- Restricted Area — access to buildings/rooms with `Access_Tier_Required__c = 'Restricted'`
- PIV/CAC Required — highest tier, required for classified-adjacent areas

Badge tier on `Badge__c` must be >= `Access_Tier_Required__c` on the target Room/Building for access to be granted. This check is enforced in the ACS integration layer, not in Salesforce (since Salesforce doesn't control the physical doors).

---

## 5. Integration Seams

### 5.1 Physical Access Control System (ACS)

- **Direction:** Salesforce → ACS (badge status changes)
- **Mechanism:** Platform Event `Badge_Status_Changed__e`
  - Fields: `Badge_Id__c`, `Contact_Id__c`, `New_Status__c`, `Badge_Tier__c`, `Effective_DateTime__c`
- **ACS → Salesforce:** ACS publishes access events; a connected subscriber writes them to `Access_Log__c`
- **Current state:** Platform Event schema defined; no subscriber wired. ACS integration is a named future milestone.

### 5.2 Clearance API (DISS/NBIS or equivalent)

- **Direction:** Salesforce → Clearance API (query) → Salesforce (result)
- **Mechanism:** Named Credential + Apex callout class `ClearanceApiService`
- **Current state:** Named Credential configured with mock endpoint. `ClearanceApiService` returns hardcoded mock data in dev. Real endpoint and credentials are plugged in when API access is procured.
- **Data returned:** Clearance tier confirmation (yes/no + tier), date of last adjudication — nothing classified stored in Salesforce.

### 5.3 SIEM / Audit

- Event Monitoring logs stream to agency SIEM via existing GovCloud connector (agency-specific configuration, out of scope for CEMS build).

---

## 6. Agentforce Agents

All agent actions are implemented as **invocable Apex methods** — compatible with both Agentforce and Flow. If a GovCloud+ capability is not yet FedRAMP High authorized at build time, the same action runs as a Flow step with no rework.

### 6.1 Enrollment Concierge (Phase 1)

**Persona:** Friendly, efficient intake coordinator. Available 24/7 on the Experience Cloud portal.

**Topics:**
- Program availability queries
- Nominee prerequisite questions
- Waitlist status checks
- Enrollment cancellation and rebooking

**Actions:**
| Action | What it does |
|---|---|
| `QueryCourseOfferings` | Searches available offerings by date, topic, agency, clearance requirement |
| `SubmitNomination` | Creates Enrollment__c in Nominated status |
| `CheckWaitlistPosition` | Returns waitlist rank for a Contact + Offering |
| `CancelEnrollment` | Triggers cancellation Flow; offers alternative dates |

**Escalation:** Clearance questions, billing disputes, policy exceptions → human coordinator via Case.

### 6.2 Scheduling Assistant (Phase 2)

**Persona:** Proactive logistics coordinator. Zero conflicts.

**Topics:**
- Room availability requests
- Instructor conflict resolution
- External event room assignments
- Proactive conflict alerts on scheduling dashboard

**Actions:**
| Action | What it does |
|---|---|
| `FindAvailableRooms` | Queries availability for date range + capacity + access tier |
| `SuggestInstructor` | Finds available Service Resources with matching qualifications |
| `CreateRoomBooking` | Creates Room_Booking__c with conflict check |
| `ResolveConflict` | Proposes alternative rooms or times for a detected conflict |

**Escalation:** Access tier overrides → Security staff.

### 6.3 Badging Coordinator (Phase 4)

**Persona:** Security-aware, precise. Zero tolerance for ambiguity.

**Topics:**
- Badge requests for new arrivals
- Expired badge + active enrollment alerts
- Immediate access revocation requests
- Proactive expiry monitoring

**Actions:**
| Action | What it does |
|---|---|
| `SubmitBadgeRequest` | Creates Badge_Request__c, validates clearance reference exists |
| `CheckBadgeStatus` | Returns current badge status and tier for a Contact |
| `FlagExpiredBadge` | Creates alert Case when expired badge + active enrollment detected |
| `RevokeBadge` | Sets Badge__c status to Revoked, publishes Badge_Status_Changed__e |

**Escalation:** All revocations and tier-change requests require human security staff approval. Agent prepares the record and routes — never auto-approves.

---

## 7. Phased Delivery Plan

### Phase 1: People + Training Programs + Enrollment
*Foundation — all subsequent phases build on this*

- Full object model deployed (all objects, all fields, all relationships)
- Shield encryption configured on CUI fields
- Training Program catalog with Course Offerings and Sessions
- Enrollment flows: agency nomination (Case → Enrollment) + self-service (Experience Cloud portal)
- Waitlist management (Flow)
- Assessment, grading, certificate issuance
- Enrollment Concierge agent

### Phase 2: Scheduling
*Rooms, instructors, and external event bookings*

- Campus / Building / Room hierarchy
- Salesforce Scheduler for instructors and rooms
- Room_Booking__c with double-booking enforcement
- Facility_Event__c for external group bookings
- Conflict detection and resolution
- Scheduling Assistant agent

### Phase 3: Lodging
*Hotel room inventory and reservations*

- Hotel rooms as Room__c under Hotel Buildings
- Lodging_Reservation__c with availability enforcement
- Check-in / check-out workflow in Service Console
- Lodging request in Experience Cloud portal during enrollment
- Automated lodging suggestions on enrollment confirmation (Flow)

### Phase 4: Badging + Access Control
*Tiered access, clearance seam, ACS integration*

- Badge_Request__c workflow end to end
- Badge tiers mapped to Room/Building access tier requirements
- Clearance_Reference__c with mock data; Named Credential + callout shell ready for real API
- Platform Event Badge_Status_Changed__e schema and publisher
- Access_Log__c subscriber skeleton
- PIV/CAC enforcement via Salesforce Identity for restricted-tier record access
- Badging Coordinator agent

---

## 8. Open Items (resolve before Phase 1 build)

1. **GovCloud+ sandbox:** Which sandbox org will be used for development?
2. **Agency IdP:** Is there an existing agency Identity Provider for SSO into the Experience Cloud portal?
3. **Program catalog:** Does CEMS replicate an existing training catalog or define it fresh?
4. **Transcript format:** Are there specific PDF templates or federal form numbers required for certificates?
5. **System Owner:** Who is the System Owner for ATO purposes?
6. **Agentforce GovCloud+ status:** Verify which Agentforce capabilities are currently FedRAMP High authorized before Phase 1 agent build begins.

---

## 9. Risks

| Risk | Impact | Mitigation |
|---|---|---|
| Agentforce not yet FedRAMP High authorized for all capabilities | Agents may ship as Flow-based automation first | All agent actions implemented as invocable Apex — works with both |
| ATO timeline unknown | Could delay production deployment | Build in GovCloud sandbox; ATO documentation starts Phase 1 |
| Clearance API access slow to procure | Delays Phase 4 badging | Mocked seam keeps dev moving |
| PIV/CAC enrollment for portal users | Not all trainees have PIV/CAC | Portal uses TOTP/passkey; PIV/CAC required only for restricted-tier |
| Data model scope creep | Federal programs expand requirements mid-build | Lock Phase 1 schema before Phase 2 starts; changes require formal schema review |
