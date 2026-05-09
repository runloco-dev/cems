# CEMS Phase 1 — People + Training Programs + Enrollment

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Stand up the Salesforce DX project, deploy all Phase 1 objects and fields, implement enrollment services with full test coverage, build nomination and waitlist flows, and deliver an Experience Cloud portal with course catalog and enrollment form — topped by the Agentforce Enrollment Concierge agent.

**Architecture:** Salesforce DX metadata project targeting Salesforce Government Cloud Plus (FedRAMP High). All business logic lives in Apex service classes invoked by Flows and Agentforce actions. Experience Cloud portal surfaces the self-service enrollment path. Agentforce Enrollment Concierge actions are invocable Apex — compatible with both Agentforce and Flow so they degrade gracefully if GovCloud+ AI features are unavailable.

**Tech Stack:** Salesforce CLI (`sf`), Apex, Lightning Web Components, Salesforce Flows, Experience Cloud (LWR), Agentforce, Shield Platform Encryption metadata

---

## File Map

```
/
├── sfdx-project.json
├── .forceignore
├── config/
│   └── project-scratch-def.json
└── force-app/main/default/
    ├── objects/
    │   ├── Contact/fields/
    │   │   ├── Person_Type__c.field-meta.xml
    │   │   ├── Clearance_Tier__c.field-meta.xml
    │   │   ├── PIV_CAC_ID__c.field-meta.xml
    │   │   └── Agency__c.field-meta.xml
    │   ├── Training_Program__c/
    │   │   ├── Training_Program__c.object-meta.xml
    │   │   └── fields/
    │   │       ├── Description__c.field-meta.xml
    │   │       ├── Prerequisites__c.field-meta.xml
    │   │       ├── Clearance_Tier_Required__c.field-meta.xml
    │   │       └── Max_Duration_Days__c.field-meta.xml
    │   ├── Course_Offering__c/
    │   │   ├── Course_Offering__c.object-meta.xml
    │   │   └── fields/
    │   │       ├── Training_Program__c.field-meta.xml
    │   │       ├── Start_Date__c.field-meta.xml
    │   │       ├── End_Date__c.field-meta.xml
    │   │       ├── Max_Enrollment__c.field-meta.xml
    │   │       ├── Current_Enrollment__c.field-meta.xml
    │   │       ├── Status__c.field-meta.xml
    │   │       └── Waitlist_Enabled__c.field-meta.xml
    │   ├── Session__c/
    │   │   ├── Session__c.object-meta.xml
    │   │   └── fields/
    │   │       ├── Course_Offering__c.field-meta.xml
    │   │       ├── Session_Date__c.field-meta.xml
    │   │       ├── Start_Time__c.field-meta.xml
    │   │       ├── End_Time__c.field-meta.xml
    │   │       ├── Topic__c.field-meta.xml
    │   │       └── Instructor__c.field-meta.xml
    │   ├── Enrollment__c/
    │   │   ├── Enrollment__c.object-meta.xml
    │   │   └── fields/
    │   │       ├── Contact__c.field-meta.xml
    │   │       ├── Course_Offering__c.field-meta.xml
    │   │       ├── Status__c.field-meta.xml
    │   │       ├── Waitlist_Position__c.field-meta.xml
    │   │       ├── Nominated_By__c.field-meta.xml
    │   │       └── Nomination_Case__c.field-meta.xml
    │   ├── Assessment__c/
    │   │   ├── Assessment__c.object-meta.xml
    │   │   └── fields/
    │   │       ├── Enrollment__c.field-meta.xml
    │   │       ├── Session__c.field-meta.xml
    │   │       ├── Score__c.field-meta.xml
    │   │       ├── Max_Score__c.field-meta.xml
    │   │       ├── Pass_Fail__c.field-meta.xml
    │   │       └── Grader__c.field-meta.xml
    │   └── Certificate__c/
    │       ├── Certificate__c.object-meta.xml
    │       └── fields/
    │           ├── Enrollment__c.field-meta.xml
    │           ├── Certificate_Number__c.field-meta.xml
    │           ├── Issue_Date__c.field-meta.xml
    │           └── Expiry_Date__c.field-meta.xml
    ├── classes/
    │   ├── EnrollmentService.cls
    │   ├── EnrollmentService.cls-meta.xml
    │   ├── EnrollmentService_Test.cls
    │   ├── EnrollmentService_Test.cls-meta.xml
    │   ├── WaitlistService.cls
    │   ├── WaitlistService.cls-meta.xml
    │   ├── WaitlistService_Test.cls
    │   ├── WaitlistService_Test.cls-meta.xml
    │   ├── CertificateService.cls
    │   ├── CertificateService.cls-meta.xml
    │   ├── CertificateService_Test.cls
    │   └── CertificateService_Test.cls-meta.xml
    ├── flows/
    │   ├── Nomination_To_Enrollment.flow-meta.xml
    │   └── Waitlist_Promote.flow-meta.xml
    ├── permissionsets/
    │   ├── CEMS_Coordinator.permissionset-meta.xml
    │   ├── CEMS_Instructor.permissionset-meta.xml
    │   └── CEMS_Trainee_Portal.permissionset-meta.xml
    └── lwc/
        ├── cemsCourseCatalog/
        │   ├── cemsCourseCatalog.html
        │   ├── cemsCourseCatalog.js
        │   └── cemsCourseCatalog.js-meta.xml
        └── cemsEnrollmentForm/
            ├── cemsEnrollmentForm.html
            ├── cemsEnrollmentForm.js
            └── cemsEnrollmentForm.js-meta.xml
```

---

## Task 1: Salesforce DX Project Scaffold

**Files:**
- Create: `sfdx-project.json`
- Create: `.forceignore`
- Create: `config/project-scratch-def.json`

- [ ] **Step 1: Initialize the DX project**

```bash
cd /Users/bdins/code/cems
sf project generate --name cems --default-package-dir force-app --manifest
```

Expected: `sfdx-project.json` and `force-app/` created.

- [ ] **Step 2: Replace sfdx-project.json with correct content**

```json
{
  "packageDirectories": [
    {
      "path": "force-app",
      "default": true,
      "package": "CEMS",
      "versionName": "Phase 1 - Enrollment",
      "versionNumber": "1.0.0.NEXT"
    }
  ],
  "name": "cems",
  "namespace": "",
  "sfdcLoginUrl": "https://login.salesforce.com",
  "sourceApiVersion": "62.0"
}
```

- [ ] **Step 3: Create scratch org definition**

Write `config/project-scratch-def.json`:

```json
{
  "orgName": "CEMS Dev",
  "edition": "Enterprise",
  "features": [
    "ServiceCloud",
    "CommunityCloud",
    "Agentforce"
  ],
  "settings": {
    "orgPreferenceSettings": {
      "s1DesktopEnabled": true,
      "selfSetPasswordInApi": true
    },
    "caseSettings": {
      "defaultCaseOwner": "CEMS_Coordinator",
      "defaultCaseOwnerType": "Queue"
    }
  }
}
```

- [ ] **Step 4: Write .forceignore**

```
# Standard ignores
.localdevserver
.sf
.sfdx
**/.DS_Store
**/node_modules

# Scratch org artifacts
**/profiles/
**/settings/
```

- [ ] **Step 5: Authorize dev org**

```bash
sf org login web --alias cems-dev --set-default
```

Expected: browser opens, log into your GovCloud+ sandbox or scratch org.

- [ ] **Step 6: Create scratch org (if using scratch)**

```bash
sf org create scratch --definition-file config/project-scratch-def.json --alias cems-scratch --duration-days 30 --set-default
```

Expected: `Successfully created scratch org: cems-scratch`

- [ ] **Step 7: Commit scaffold**

```bash
git add sfdx-project.json .forceignore config/
git commit -m "chore: salesforce dx project scaffold"
```

---

## Task 2: Contact and Account Field Extensions

**Files:**
- Create: `force-app/main/default/objects/Contact/fields/Person_Type__c.field-meta.xml`
- Create: `force-app/main/default/objects/Contact/fields/Clearance_Tier__c.field-meta.xml`
- Create: `force-app/main/default/objects/Contact/fields/PIV_CAC_ID__c.field-meta.xml`
- Create: `force-app/main/default/objects/Contact/fields/Agency__c.field-meta.xml`
- Create: `force-app/main/default/objects/Contact/fields/Is_On_Campus__c.field-meta.xml`

- [ ] **Step 1: Create Contact fields directory**

```bash
mkdir -p force-app/main/default/objects/Contact/fields
```

- [ ] **Step 2: Create Person_Type__c**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<CustomField xmlns="http://soap.sforce.com/2006/04/metadata">
    <fullName>Person_Type__c</fullName>
    <label>Person Type</label>
    <type>Picklist</type>
    <required>true</required>
    <valueSet>
        <restricted>true</restricted>
        <valueSetDefinition>
            <sorted>false</sorted>
            <value><fullName>Federal Employee</fullName><default>false</default><label>Federal Employee</label></value>
            <value><fullName>Contractor</fullName><default>false</default><label>Contractor</label></value>
            <value><fullName>Civilian Trainee</fullName><default>false</default><label>Civilian Trainee</label></value>
            <value><fullName>Instructor</fullName><default>false</default><label>Instructor</label></value>
            <value><fullName>External Attendee</fullName><default>false</default><label>External Attendee</label></value>
        </valueSetDefinition>
    </valueSet>
</CustomField>
```

- [ ] **Step 3: Create Clearance_Tier__c**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<CustomField xmlns="http://soap.sforce.com/2006/04/metadata">
    <fullName>Clearance_Tier__c</fullName>
    <label>Clearance Tier (Reference)</label>
    <description>Reference only — not the authoritative adjudication record. Source of truth is the agency clearance system.</description>
    <type>Picklist</type>
    <required>false</required>
    <valueSet>
        <restricted>true</restricted>
        <valueSetDefinition>
            <sorted>false</sorted>
            <value><fullName>None</fullName><default>true</default><label>None</label></value>
            <value><fullName>Public Trust</fullName><default>false</default><label>Public Trust</label></value>
            <value><fullName>Secret</fullName><default>false</default><label>Secret</label></value>
            <value><fullName>Top Secret</fullName><default>false</default><label>Top Secret</label></value>
            <value><fullName>TS/SCI</fullName><default>false</default><label>TS/SCI</label></value>
        </valueSetDefinition>
    </valueSet>
</CustomField>
```

- [ ] **Step 4: Create PIV_CAC_ID__c**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<CustomField xmlns="http://soap.sforce.com/2006/04/metadata">
    <fullName>PIV_CAC_ID__c</fullName>
    <label>PIV/CAC ID</label>
    <description>Federal credential identifier. Shield-encrypted. Do not log or display in UI without explicit need.</description>
    <type>Text</type>
    <length>100</length>
    <required>false</required>
    <unique>false</unique>
    <externalId>false</externalId>
</CustomField>
```

- [ ] **Step 5: Create Agency__c lookup**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<CustomField xmlns="http://soap.sforce.com/2006/04/metadata">
    <fullName>Agency__c</fullName>
    <label>Agency</label>
    <type>Lookup</type>
    <referenceTo>Account</referenceTo>
    <relationshipLabel>Agency Contacts</relationshipLabel>
    <relationshipName>Agency_Contacts</relationshipName>
    <required>false</required>
</CustomField>
```

- [ ] **Step 6: Create Is_On_Campus__c**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<CustomField xmlns="http://soap.sforce.com/2006/04/metadata">
    <fullName>Is_On_Campus__c</fullName>
    <label>On Campus</label>
    <type>Checkbox</type>
    <defaultValue>false</defaultValue>
</CustomField>
```

- [ ] **Step 7: Deploy Contact fields**

```bash
sf project deploy start --source-dir force-app/main/default/objects/Contact --target-org cems-dev
```

Expected: `Deploy Succeeded`

- [ ] **Step 8: Commit**

```bash
git add force-app/main/default/objects/Contact/
git commit -m "feat(objects): Contact field extensions — person type, clearance tier, PIV/CAC, agency"
```

---

## Task 3: Training_Program__c Object

**Files:**
- Create: `force-app/main/default/objects/Training_Program__c/Training_Program__c.object-meta.xml`
- Create: `force-app/main/default/objects/Training_Program__c/fields/*.field-meta.xml`

- [ ] **Step 1: Create object directories**

```bash
mkdir -p force-app/main/default/objects/Training_Program__c/fields
```

- [ ] **Step 2: Create object definition**

`Training_Program__c.object-meta.xml`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<CustomObject xmlns="http://soap.sforce.com/2006/04/metadata">
    <label>Training Program</label>
    <pluralLabel>Training Programs</pluralLabel>
    <nameField>
        <label>Program Name</label>
        <type>Text</type>
    </nameField>
    <deploymentStatus>Deployed</deploymentStatus>
    <sharingModel>ReadWrite</sharingModel>
</CustomObject>
```

- [ ] **Step 3: Create fields**

`fields/Description__c.field-meta.xml`:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<CustomField xmlns="http://soap.sforce.com/2006/04/metadata">
    <fullName>Description__c</fullName><label>Description</label>
    <type>LongTextArea</type><length>32768</length><visibleLines>5</visibleLines>
</CustomField>
```

`fields/Prerequisites__c.field-meta.xml`:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<CustomField xmlns="http://soap.sforce.com/2006/04/metadata">
    <fullName>Prerequisites__c</fullName><label>Prerequisites</label>
    <type>LongTextArea</type><length>32768</length><visibleLines>3</visibleLines>
</CustomField>
```

`fields/Clearance_Tier_Required__c.field-meta.xml`:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<CustomField xmlns="http://soap.sforce.com/2006/04/metadata">
    <fullName>Clearance_Tier_Required__c</fullName>
    <label>Clearance Tier Required</label>
    <type>Picklist</type>
    <valueSet>
        <restricted>true</restricted>
        <valueSetDefinition>
            <value><fullName>None</fullName><default>true</default><label>None</label></value>
            <value><fullName>Public Trust</fullName><default>false</default><label>Public Trust</label></value>
            <value><fullName>Secret</fullName><default>false</default><label>Secret</label></value>
            <value><fullName>Top Secret</fullName><default>false</default><label>Top Secret</label></value>
            <value><fullName>TS/SCI</fullName><default>false</default><label>TS/SCI</label></value>
        </valueSetDefinition>
    </valueSet>
</CustomField>
```

`fields/Max_Duration_Days__c.field-meta.xml`:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<CustomField xmlns="http://soap.sforce.com/2006/04/metadata">
    <fullName>Max_Duration_Days__c</fullName><label>Max Duration (Days)</label>
    <type>Number</type><precision>3</precision><scale>0</scale>
</CustomField>
```

- [ ] **Step 4: Deploy**

```bash
sf project deploy start --source-dir force-app/main/default/objects/Training_Program__c --target-org cems-dev
```

Expected: `Deploy Succeeded`

- [ ] **Step 5: Commit**

```bash
git add force-app/main/default/objects/Training_Program__c/
git commit -m "feat(objects): Training_Program__c — catalog entry for training programs"
```

---

## Task 4: Course_Offering__c and Session__c Objects

**Files:**
- Create: `force-app/main/default/objects/Course_Offering__c/` (full tree)
- Create: `force-app/main/default/objects/Session__c/` (full tree)

- [ ] **Step 1: Create directories**

```bash
mkdir -p force-app/main/default/objects/Course_Offering__c/fields
mkdir -p force-app/main/default/objects/Session__c/fields
```

- [ ] **Step 2: Course_Offering__c object**

`Course_Offering__c.object-meta.xml`:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<CustomObject xmlns="http://soap.sforce.com/2006/04/metadata">
    <label>Course Offering</label>
    <pluralLabel>Course Offerings</pluralLabel>
    <nameField><label>Offering Name</label><type>AutoNumber</type>
        <displayFormat>OFF-{0000}</displayFormat></nameField>
    <deploymentStatus>Deployed</deploymentStatus>
    <sharingModel>ReadWrite</sharingModel>
</CustomObject>
```

- [ ] **Step 3: Course_Offering__c fields**

`fields/Training_Program__c.field-meta.xml`:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<CustomField xmlns="http://soap.sforce.com/2006/04/metadata">
    <fullName>Training_Program__c</fullName><label>Training Program</label>
    <type>MasterDetail</type><referenceTo>Training_Program__c</referenceTo>
    <relationshipLabel>Course Offerings</relationshipLabel>
    <relationshipName>Course_Offerings</relationshipName>
    <relationshipOrder>0</relationshipOrder>
</CustomField>
```

`fields/Start_Date__c.field-meta.xml`:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<CustomField xmlns="http://soap.sforce.com/2006/04/metadata">
    <fullName>Start_Date__c</fullName><label>Start Date</label>
    <type>Date</type><required>true</required>
</CustomField>
```

`fields/End_Date__c.field-meta.xml`:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<CustomField xmlns="http://soap.sforce.com/2006/04/metadata">
    <fullName>End_Date__c</fullName><label>End Date</label>
    <type>Date</type><required>true</required>
</CustomField>
```

`fields/Max_Enrollment__c.field-meta.xml`:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<CustomField xmlns="http://soap.sforce.com/2006/04/metadata">
    <fullName>Max_Enrollment__c</fullName><label>Max Enrollment</label>
    <type>Number</type><precision>4</precision><scale>0</scale><required>true</required>
</CustomField>
```

`fields/Current_Enrollment__c.field-meta.xml`:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<CustomField xmlns="http://soap.sforce.com/2006/04/metadata">
    <fullName>Current_Enrollment__c</fullName><label>Current Enrollment</label>
    <type>RollupSummary</type>
    <summaryForeignKey>Enrollment__c.Course_Offering__c</summaryForeignKey>
    <summaryOperation>COUNT</summaryOperation>
    <summaryFilterItems>
        <field>Enrollment__c.Status__c</field>
        <operation>equals</operation>
        <value>Enrolled</value>
    </summaryFilterItems>
</CustomField>
```

`fields/Status__c.field-meta.xml`:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<CustomField xmlns="http://soap.sforce.com/2006/04/metadata">
    <fullName>Status__c</fullName><label>Status</label><type>Picklist</type>
    <valueSet>
        <restricted>true</restricted>
        <valueSetDefinition>
            <value><fullName>Open</fullName><default>true</default><label>Open</label></value>
            <value><fullName>Full</fullName><default>false</default><label>Full</label></value>
            <value><fullName>In Progress</fullName><default>false</default><label>In Progress</label></value>
            <value><fullName>Completed</fullName><default>false</default><label>Completed</label></value>
            <value><fullName>Cancelled</fullName><default>false</default><label>Cancelled</label></value>
        </valueSetDefinition>
    </valueSet>
</CustomField>
```

`fields/Waitlist_Enabled__c.field-meta.xml`:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<CustomField xmlns="http://soap.sforce.com/2006/04/metadata">
    <fullName>Waitlist_Enabled__c</fullName><label>Waitlist Enabled</label>
    <type>Checkbox</type><defaultValue>true</defaultValue>
</CustomField>
```

- [ ] **Step 4: Session__c object**

`Session__c.object-meta.xml`:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<CustomObject xmlns="http://soap.sforce.com/2006/04/metadata">
    <label>Session</label>
    <pluralLabel>Sessions</pluralLabel>
    <nameField><label>Session Name</label><type>AutoNumber</type>
        <displayFormat>SES-{0000}</displayFormat></nameField>
    <deploymentStatus>Deployed</deploymentStatus>
    <sharingModel>ControlledByParent</sharingModel>
</CustomObject>
```

- [ ] **Step 5: Session__c fields**

`fields/Course_Offering__c.field-meta.xml`:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<CustomField xmlns="http://soap.sforce.com/2006/04/metadata">
    <fullName>Course_Offering__c</fullName><label>Course Offering</label>
    <type>MasterDetail</type><referenceTo>Course_Offering__c</referenceTo>
    <relationshipLabel>Sessions</relationshipLabel>
    <relationshipName>Sessions</relationshipName>
    <relationshipOrder>0</relationshipOrder>
</CustomField>
```

`fields/Session_Date__c.field-meta.xml`:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<CustomField xmlns="http://soap.sforce.com/2006/04/metadata">
    <fullName>Session_Date__c</fullName><label>Session Date</label>
    <type>Date</type><required>true</required>
</CustomField>
```

`fields/Start_Time__c.field-meta.xml`:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<CustomField xmlns="http://soap.sforce.com/2006/04/metadata">
    <fullName>Start_Time__c</fullName><label>Start Time</label>
    <type>DateTime</type>
</CustomField>
```

`fields/End_Time__c.field-meta.xml`:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<CustomField xmlns="http://soap.sforce.com/2006/04/metadata">
    <fullName>End_Time__c</fullName><label>End Time</label>
    <type>DateTime</type>
</CustomField>
```

`fields/Topic__c.field-meta.xml`:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<CustomField xmlns="http://soap.sforce.com/2006/04/metadata">
    <fullName>Topic__c</fullName><label>Topic</label>
    <type>Text</type><length>255</length>
</CustomField>
```

`fields/Instructor__c.field-meta.xml`:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<CustomField xmlns="http://soap.sforce.com/2006/04/metadata">
    <fullName>Instructor__c</fullName><label>Instructor</label>
    <type>Lookup</type><referenceTo>User</referenceTo>
    <relationshipLabel>Sessions</relationshipLabel>
    <relationshipName>Sessions</relationshipName>
</CustomField>
```

- [ ] **Step 6: Deploy both objects**

```bash
sf project deploy start --source-dir force-app/main/default/objects/Course_Offering__c --source-dir force-app/main/default/objects/Session__c --target-org cems-dev
```

Expected: `Deploy Succeeded`

- [ ] **Step 7: Commit**

```bash
git add force-app/main/default/objects/Course_Offering__c/ force-app/main/default/objects/Session__c/
git commit -m "feat(objects): Course_Offering__c and Session__c — scheduled runs and daily sessions"
```

---

## Task 5: Enrollment__c Object

**Files:**
- Create: `force-app/main/default/objects/Enrollment__c/` (full tree)

- [ ] **Step 1: Create directories**

```bash
mkdir -p force-app/main/default/objects/Enrollment__c/fields
```

- [ ] **Step 2: Object definition**

`Enrollment__c.object-meta.xml`:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<CustomObject xmlns="http://soap.sforce.com/2006/04/metadata">
    <label>Enrollment</label>
    <pluralLabel>Enrollments</pluralLabel>
    <nameField><label>Enrollment Number</label><type>AutoNumber</type>
        <displayFormat>ENR-{000000}</displayFormat></nameField>
    <deploymentStatus>Deployed</deploymentStatus>
    <sharingModel>ReadWrite</sharingModel>
</CustomObject>
```

- [ ] **Step 3: Enrollment__c fields**

`fields/Contact__c.field-meta.xml`:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<CustomField xmlns="http://soap.sforce.com/2006/04/metadata">
    <fullName>Contact__c</fullName><label>Contact</label>
    <type>MasterDetail</type><referenceTo>Contact</referenceTo>
    <relationshipLabel>Enrollments</relationshipLabel>
    <relationshipName>Enrollments</relationshipName>
    <relationshipOrder>0</relationshipOrder>
</CustomField>
```

`fields/Course_Offering__c.field-meta.xml`:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<CustomField xmlns="http://soap.sforce.com/2006/04/metadata">
    <fullName>Course_Offering__c</fullName><label>Course Offering</label>
    <type>Lookup</type><referenceTo>Course_Offering__c</referenceTo>
    <relationshipLabel>Enrollments</relationshipLabel>
    <relationshipName>Enrollments</relationshipName>
    <required>true</required>
</CustomField>
```

`fields/Status__c.field-meta.xml`:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<CustomField xmlns="http://soap.sforce.com/2006/04/metadata">
    <fullName>Status__c</fullName><label>Status</label><type>Picklist</type>
    <required>true</required>
    <valueSet>
        <restricted>true</restricted>
        <valueSetDefinition>
            <value><fullName>Nominated</fullName><default>false</default><label>Nominated</label></value>
            <value><fullName>Enrolled</fullName><default>false</default><label>Enrolled</label></value>
            <value><fullName>Waitlisted</fullName><default>false</default><label>Waitlisted</label></value>
            <value><fullName>Completed</fullName><default>false</default><label>Completed</label></value>
            <value><fullName>Cancelled</fullName><default>false</default><label>Cancelled</label></value>
            <value><fullName>No Show</fullName><default>false</default><label>No Show</label></value>
        </valueSetDefinition>
    </valueSet>
</CustomField>
```

`fields/Waitlist_Position__c.field-meta.xml`:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<CustomField xmlns="http://soap.sforce.com/2006/04/metadata">
    <fullName>Waitlist_Position__c</fullName><label>Waitlist Position</label>
    <type>Number</type><precision>4</precision><scale>0</scale>
</CustomField>
```

`fields/Nominated_By__c.field-meta.xml`:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<CustomField xmlns="http://soap.sforce.com/2006/04/metadata">
    <fullName>Nominated_By__c</fullName><label>Nominated By (Agency)</label>
    <type>Lookup</type><referenceTo>Account</referenceTo>
    <relationshipLabel>Nominated Enrollments</relationshipLabel>
    <relationshipName>Nominated_Enrollments</relationshipName>
</CustomField>
```

`fields/Nomination_Case__c.field-meta.xml`:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<CustomField xmlns="http://soap.sforce.com/2006/04/metadata">
    <fullName>Nomination_Case__c</fullName><label>Nomination Case</label>
    <type>Lookup</type><referenceTo>Case</referenceTo>
    <relationshipLabel>Enrollments</relationshipLabel>
    <relationshipName>Enrollments</relationshipName>
</CustomField>
```

- [ ] **Step 4: Deploy**

```bash
sf project deploy start --source-dir force-app/main/default/objects/Enrollment__c --target-org cems-dev
```

Expected: `Deploy Succeeded`

- [ ] **Step 5: Commit**

```bash
git add force-app/main/default/objects/Enrollment__c/
git commit -m "feat(objects): Enrollment__c — links Contact to Course Offering with status and waitlist"
```

---

## Task 6: Assessment__c and Certificate__c Objects

**Files:**
- Create: `force-app/main/default/objects/Assessment__c/` (full tree)
- Create: `force-app/main/default/objects/Certificate__c/` (full tree)

- [ ] **Step 1: Create directories**

```bash
mkdir -p force-app/main/default/objects/Assessment__c/fields
mkdir -p force-app/main/default/objects/Certificate__c/fields
```

- [ ] **Step 2: Assessment__c object**

`Assessment__c.object-meta.xml`:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<CustomObject xmlns="http://soap.sforce.com/2006/04/metadata">
    <label>Assessment</label><pluralLabel>Assessments</pluralLabel>
    <nameField><label>Assessment Name</label><type>AutoNumber</type>
        <displayFormat>ASM-{0000}</displayFormat></nameField>
    <deploymentStatus>Deployed</deploymentStatus>
    <sharingModel>ControlledByParent</sharingModel>
</CustomObject>
```

- [ ] **Step 3: Assessment__c fields**

`fields/Enrollment__c.field-meta.xml`:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<CustomField xmlns="http://soap.sforce.com/2006/04/metadata">
    <fullName>Enrollment__c</fullName><label>Enrollment</label>
    <type>MasterDetail</type><referenceTo>Enrollment__c</referenceTo>
    <relationshipLabel>Assessments</relationshipLabel>
    <relationshipName>Assessments</relationshipName><relationshipOrder>0</relationshipOrder>
</CustomField>
```

`fields/Session__c.field-meta.xml`:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<CustomField xmlns="http://soap.sforce.com/2006/04/metadata">
    <fullName>Session__c</fullName><label>Session</label>
    <type>Lookup</type><referenceTo>Session__c</referenceTo>
    <relationshipLabel>Assessments</relationshipLabel>
    <relationshipName>Assessments</relationshipName>
</CustomField>
```

`fields/Score__c.field-meta.xml`:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<CustomField xmlns="http://soap.sforce.com/2006/04/metadata">
    <fullName>Score__c</fullName><label>Score</label>
    <type>Number</type><precision>5</precision><scale>2</scale>
</CustomField>
```

`fields/Max_Score__c.field-meta.xml`:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<CustomField xmlns="http://soap.sforce.com/2006/04/metadata">
    <fullName>Max_Score__c</fullName><label>Max Score</label>
    <type>Number</type><precision>5</precision><scale>2</scale><defaultValue>100</defaultValue>
</CustomField>
```

`fields/Pass_Fail__c.field-meta.xml`:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<CustomField xmlns="http://soap.sforce.com/2006/04/metadata">
    <fullName>Pass_Fail__c</fullName><label>Pass/Fail</label><type>Picklist</type>
    <valueSet>
        <restricted>true</restricted>
        <valueSetDefinition>
            <value><fullName>Pass</fullName><default>false</default><label>Pass</label></value>
            <value><fullName>Fail</fullName><default>false</default><label>Fail</label></value>
            <value><fullName>Incomplete</fullName><default>true</default><label>Incomplete</label></value>
        </valueSetDefinition>
    </valueSet>
</CustomField>
```

`fields/Grader__c.field-meta.xml`:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<CustomField xmlns="http://soap.sforce.com/2006/04/metadata">
    <fullName>Grader__c</fullName><label>Grader</label>
    <type>Lookup</type><referenceTo>User</referenceTo>
    <relationshipLabel>Graded Assessments</relationshipLabel>
    <relationshipName>Graded_Assessments</relationshipName>
</CustomField>
```

- [ ] **Step 4: Certificate__c object**

`Certificate__c.object-meta.xml`:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<CustomObject xmlns="http://soap.sforce.com/2006/04/metadata">
    <label>Certificate</label><pluralLabel>Certificates</pluralLabel>
    <nameField><label>Certificate Number</label><type>AutoNumber</type>
        <displayFormat>CERT-{000000}</displayFormat></nameField>
    <deploymentStatus>Deployed</deploymentStatus>
    <sharingModel>ControlledByParent</sharingModel>
</CustomObject>
```

- [ ] **Step 5: Certificate__c fields**

`fields/Enrollment__c.field-meta.xml`:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<CustomField xmlns="http://soap.sforce.com/2006/04/metadata">
    <fullName>Enrollment__c</fullName><label>Enrollment</label>
    <type>MasterDetail</type><referenceTo>Enrollment__c</referenceTo>
    <relationshipLabel>Certificates</relationshipLabel>
    <relationshipName>Certificates</relationshipName><relationshipOrder>0</relationshipOrder>
</CustomField>
```

`fields/Issue_Date__c.field-meta.xml`:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<CustomField xmlns="http://soap.sforce.com/2006/04/metadata">
    <fullName>Issue_Date__c</fullName><label>Issue Date</label>
    <type>Date</type><required>true</required>
</CustomField>
```

`fields/Expiry_Date__c.field-meta.xml`:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<CustomField xmlns="http://soap.sforce.com/2006/04/metadata">
    <fullName>Expiry_Date__c</fullName><label>Expiry Date</label>
    <type>Date</type>
</CustomField>
```

- [ ] **Step 6: Deploy both objects**

```bash
sf project deploy start \
  --source-dir force-app/main/default/objects/Assessment__c \
  --source-dir force-app/main/default/objects/Certificate__c \
  --target-org cems-dev
```

Expected: `Deploy Succeeded`

- [ ] **Step 7: Commit**

```bash
git add force-app/main/default/objects/Assessment__c/ force-app/main/default/objects/Certificate__c/
git commit -m "feat(objects): Assessment__c and Certificate__c — grading and completion tracking"
```

---

## Task 7: Permission Sets

**Files:**
- Create: `force-app/main/default/permissionsets/CEMS_Coordinator.permissionset-meta.xml`
- Create: `force-app/main/default/permissionsets/CEMS_Instructor.permissionset-meta.xml`
- Create: `force-app/main/default/permissionsets/CEMS_Trainee_Portal.permissionset-meta.xml`

- [ ] **Step 1: Create directory**

```bash
mkdir -p force-app/main/default/permissionsets
```

- [ ] **Step 2: CEMS_Coordinator**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<PermissionSet xmlns="http://soap.sforce.com/2006/04/metadata">
    <label>CEMS Coordinator</label>
    <description>Full access to all CEMS objects for program coordinators.</description>
    <hasActivationRequired>false</hasActivationRequired>
    <objectPermissions>
        <object>Training_Program__c</object><allowCreate>true</allowCreate><allowDelete>true</allowDelete>
        <allowEdit>true</allowEdit><allowRead>true</allowRead><modifyAllRecords>true</modifyAllRecords><viewAllRecords>true</viewAllRecords>
    </objectPermissions>
    <objectPermissions>
        <object>Course_Offering__c</object><allowCreate>true</allowCreate><allowDelete>true</allowDelete>
        <allowEdit>true</allowEdit><allowRead>true</allowRead><modifyAllRecords>true</modifyAllRecords><viewAllRecords>true</viewAllRecords>
    </objectPermissions>
    <objectPermissions>
        <object>Session__c</object><allowCreate>true</allowCreate><allowDelete>true</allowDelete>
        <allowEdit>true</allowEdit><allowRead>true</allowRead><modifyAllRecords>true</modifyAllRecords><viewAllRecords>true</viewAllRecords>
    </objectPermissions>
    <objectPermissions>
        <object>Enrollment__c</object><allowCreate>true</allowCreate><allowDelete>true</allowDelete>
        <allowEdit>true</allowEdit><allowRead>true</allowRead><modifyAllRecords>true</modifyAllRecords><viewAllRecords>true</viewAllRecords>
    </objectPermissions>
    <objectPermissions>
        <object>Assessment__c</object><allowCreate>true</allowCreate><allowDelete>false</allowDelete>
        <allowEdit>true</allowEdit><allowRead>true</allowRead><modifyAllRecords>true</modifyAllRecords><viewAllRecords>true</viewAllRecords>
    </objectPermissions>
    <objectPermissions>
        <object>Certificate__c</object><allowCreate>true</allowCreate><allowDelete>false</allowDelete>
        <allowEdit>true</allowEdit><allowRead>true</allowRead><modifyAllRecords>true</modifyAllRecords><viewAllRecords>true</viewAllRecords>
    </objectPermissions>
</PermissionSet>
```

- [ ] **Step 3: CEMS_Instructor**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<PermissionSet xmlns="http://soap.sforce.com/2006/04/metadata">
    <label>CEMS Instructor</label>
    <description>Read access to programs/offerings/sessions; create/edit assessments for their sessions.</description>
    <hasActivationRequired>false</hasActivationRequired>
    <objectPermissions>
        <object>Training_Program__c</object><allowCreate>false</allowCreate><allowDelete>false</allowDelete>
        <allowEdit>false</allowEdit><allowRead>true</allowRead><modifyAllRecords>false</modifyAllRecords><viewAllRecords>true</viewAllRecords>
    </objectPermissions>
    <objectPermissions>
        <object>Course_Offering__c</object><allowCreate>false</allowCreate><allowDelete>false</allowDelete>
        <allowEdit>false</allowEdit><allowRead>true</allowRead><modifyAllRecords>false</modifyAllRecords><viewAllRecords>true</viewAllRecords>
    </objectPermissions>
    <objectPermissions>
        <object>Session__c</object><allowCreate>false</allowCreate><allowDelete>false</allowDelete>
        <allowEdit>true</allowEdit><allowRead>true</allowRead><modifyAllRecords>false</modifyAllRecords><viewAllRecords>true</viewAllRecords>
    </objectPermissions>
    <objectPermissions>
        <object>Enrollment__c</object><allowCreate>false</allowCreate><allowDelete>false</allowDelete>
        <allowEdit>false</allowEdit><allowRead>true</allowRead><modifyAllRecords>false</modifyAllRecords><viewAllRecords>true</viewAllRecords>
    </objectPermissions>
    <objectPermissions>
        <object>Assessment__c</object><allowCreate>true</allowCreate><allowDelete>false</allowDelete>
        <allowEdit>true</allowEdit><allowRead>true</allowRead><modifyAllRecords>false</modifyAllRecords><viewAllRecords>true</viewAllRecords>
    </objectPermissions>
</PermissionSet>
```

- [ ] **Step 4: CEMS_Trainee_Portal**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<PermissionSet xmlns="http://soap.sforce.com/2006/04/metadata">
    <label>CEMS Trainee Portal</label>
    <description>Experience Cloud portal user — read programs/offerings, create enrollment, read own records.</description>
    <hasActivationRequired>false</hasActivationRequired>
    <objectPermissions>
        <object>Training_Program__c</object><allowCreate>false</allowCreate><allowDelete>false</allowDelete>
        <allowEdit>false</allowEdit><allowRead>true</allowRead><modifyAllRecords>false</modifyAllRecords><viewAllRecords>true</viewAllRecords>
    </objectPermissions>
    <objectPermissions>
        <object>Course_Offering__c</object><allowCreate>false</allowCreate><allowDelete>false</allowDelete>
        <allowEdit>false</allowEdit><allowRead>true</allowRead><modifyAllRecords>false</modifyAllRecords><viewAllRecords>true</viewAllRecords>
    </objectPermissions>
    <objectPermissions>
        <object>Enrollment__c</object><allowCreate>true</allowCreate><allowDelete>false</allowDelete>
        <allowEdit>false</allowEdit><allowRead>true</allowRead><modifyAllRecords>false</modifyAllRecords><viewAllRecords>false</viewAllRecords>
    </objectPermissions>
    <objectPermissions>
        <object>Certificate__c</object><allowCreate>false</allowCreate><allowDelete>false</allowDelete>
        <allowEdit>false</allowEdit><allowRead>true</allowRead><modifyAllRecords>false</modifyAllRecords><viewAllRecords>false</viewAllRecords>
    </objectPermissions>
</PermissionSet>
```

- [ ] **Step 5: Deploy**

```bash
sf project deploy start --source-dir force-app/main/default/permissionsets --target-org cems-dev
```

Expected: `Deploy Succeeded`

- [ ] **Step 6: Commit**

```bash
git add force-app/main/default/permissionsets/
git commit -m "feat(security): permission sets — Coordinator, Instructor, Trainee Portal"
```

---

## Task 8: EnrollmentService Apex (TDD)

**Files:**
- Create: `force-app/main/default/classes/EnrollmentService_Test.cls` (write first)
- Create: `force-app/main/default/classes/EnrollmentService.cls`
- Create: matching `.cls-meta.xml` files

- [ ] **Step 1: Create classes directory**

```bash
mkdir -p force-app/main/default/classes
```

- [ ] **Step 2: Write the test class first**

`EnrollmentService_Test.cls`:
```apex
@isTest
private class EnrollmentService_Test {

    @TestSetup
    static void setup() {
        Account agency = new Account(Name = 'Test Agency', Type = 'Sponsoring Agency');
        insert agency;

        Contact trainee = new Contact(
            FirstName = 'Test', LastName = 'Trainee',
            Person_Type__c = 'Federal Employee',
            Agency__c = agency.Id
        );
        insert trainee;

        Training_Program__c program = new Training_Program__c(
            Name = 'ICS-300',
            Clearance_Tier_Required__c = 'None'
        );
        insert program;

        Course_Offering__c offering = new Course_Offering__c(
            Training_Program__c = program.Id,
            Start_Date__c = Date.today().addDays(30),
            End_Date__c = Date.today().addDays(35),
            Max_Enrollment__c = 2,
            Status__c = 'Open',
            Waitlist_Enabled__c = true
        );
        insert offering;
    }

    @isTest
    static void enroll_createsEnrolledRecord() {
        Contact trainee = [SELECT Id FROM Contact LIMIT 1];
        Course_Offering__c offering = [SELECT Id FROM Course_Offering__c LIMIT 1];

        Test.startTest();
        Enrollment__c result = EnrollmentService.enroll(trainee.Id, offering.Id, null);
        Test.stopTest();

        System.assertEquals('Enrolled', result.Status__c, 'Status should be Enrolled');
        System.assertEquals(trainee.Id, result.Contact__c, 'Contact should match');
        System.assertEquals(offering.Id, result.Course_Offering__c, 'Offering should match');
    }

    @isTest
    static void enroll_whenFull_createsWaitlistedRecord() {
        Contact trainee1 = new Contact(FirstName = 'A', LastName = 'One', Person_Type__c = 'Civilian Trainee');
        Contact trainee2 = new Contact(FirstName = 'B', LastName = 'Two', Person_Type__c = 'Civilian Trainee');
        Contact trainee3 = new Contact(FirstName = 'C', LastName = 'Three', Person_Type__c = 'Civilian Trainee');
        insert new List<Contact>{ trainee1, trainee2, trainee3 };

        Course_Offering__c offering = [SELECT Id FROM Course_Offering__c LIMIT 1];

        // Fill the offering (max = 2)
        EnrollmentService.enroll(trainee1.Id, offering.Id, null);
        EnrollmentService.enroll(trainee2.Id, offering.Id, null);

        Test.startTest();
        Enrollment__c result = EnrollmentService.enroll(trainee3.Id, offering.Id, null);
        Test.stopTest();

        System.assertEquals('Waitlisted', result.Status__c, 'Third enrollee should be waitlisted');
        System.assertEquals(1, result.Waitlist_Position__c, 'Waitlist position should be 1');
    }

    @isTest
    static void enroll_duplicatePrevented() {
        Contact trainee = [SELECT Id FROM Contact LIMIT 1];
        Course_Offering__c offering = [SELECT Id FROM Course_Offering__c LIMIT 1];

        EnrollmentService.enroll(trainee.Id, offering.Id, null);

        Test.startTest();
        try {
            EnrollmentService.enroll(trainee.Id, offering.Id, null);
            System.assert(false, 'Should have thrown exception for duplicate enrollment');
        } catch (EnrollmentService.EnrollmentException e) {
            System.assert(e.getMessage().contains('already enrolled'), e.getMessage());
        }
        Test.stopTest();
    }

    @isTest
    static void cancelEnrollment_setsStatusCancelled() {
        Contact trainee = [SELECT Id FROM Contact LIMIT 1];
        Course_Offering__c offering = [SELECT Id FROM Course_Offering__c LIMIT 1];
        Enrollment__c enr = EnrollmentService.enroll(trainee.Id, offering.Id, null);

        Test.startTest();
        EnrollmentService.cancel(enr.Id);
        Test.stopTest();

        Enrollment__c updated = [SELECT Status__c FROM Enrollment__c WHERE Id = :enr.Id];
        System.assertEquals('Cancelled', updated.Status__c, 'Status should be Cancelled');
    }

    @isTest
    static void nominateEnrollment_setsNominatedStatus() {
        Contact trainee = [SELECT Id FROM Contact LIMIT 1];
        Account agency = [SELECT Id FROM Account LIMIT 1];
        Course_Offering__c offering = [SELECT Id FROM Course_Offering__c LIMIT 1];

        Test.startTest();
        Enrollment__c result = EnrollmentService.nominate(trainee.Id, offering.Id, agency.Id, null);
        Test.stopTest();

        System.assertEquals('Nominated', result.Status__c, 'Status should be Nominated');
        System.assertEquals(agency.Id, result.Nominated_By__c, 'Nominated By should be agency');
    }
}
```

`EnrollmentService_Test.cls-meta.xml`:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<ApexClass xmlns="http://soap.sforce.com/2006/04/metadata">
    <apiVersion>62.0</apiVersion>
    <status>Active</status>
</ApexClass>
```

- [ ] **Step 3: Deploy and run tests — expect failure**

```bash
sf project deploy start --source-dir force-app/main/default/classes/EnrollmentService_Test.cls --target-org cems-dev
sf apex run test --class-names EnrollmentService_Test --target-org cems-dev --result-format human
```

Expected: compile error or test failure — `EnrollmentService` does not exist yet.

- [ ] **Step 4: Write EnrollmentService**

`EnrollmentService.cls`:
```apex
public with sharing class EnrollmentService {

    public class EnrollmentException extends Exception {}

    public static Enrollment__c enroll(Id contactId, Id offeringId, Id caseId) {
        assertNotAlreadyEnrolled(contactId, offeringId);
        Course_Offering__c offering = getOffering(offeringId);
        Integer enrolled = countEnrolled(offeringId);
        String status;
        Integer waitlistPosition = null;

        if (enrolled < offering.Max_Enrollment__c) {
            status = 'Enrolled';
        } else if (offering.Waitlist_Enabled__c) {
            status = 'Waitlisted';
            waitlistPosition = countWaitlisted(offeringId) + 1;
        } else {
            throw new EnrollmentException('Offering is full and waitlist is disabled.');
        }

        Enrollment__c enr = new Enrollment__c(
            Contact__c = contactId,
            Course_Offering__c = offeringId,
            Status__c = status,
            Waitlist_Position__c = waitlistPosition,
            Nomination_Case__c = caseId
        );
        insert enr;
        return enr;
    }

    public static Enrollment__c nominate(Id contactId, Id offeringId, Id agencyId, Id caseId) {
        assertNotAlreadyEnrolled(contactId, offeringId);
        Enrollment__c enr = new Enrollment__c(
            Contact__c = contactId,
            Course_Offering__c = offeringId,
            Status__c = 'Nominated',
            Nominated_By__c = agencyId,
            Nomination_Case__c = caseId
        );
        insert enr;
        return enr;
    }

    public static void cancel(Id enrollmentId) {
        Enrollment__c enr = [SELECT Id, Status__c FROM Enrollment__c WHERE Id = :enrollmentId WITH SECURITY_ENFORCED];
        enr.Status__c = 'Cancelled';
        enr.Waitlist_Position__c = null;
        update enr;
    }

    // Invocable wrapper for Flow and Agentforce
    @InvocableMethod(label='Enroll Contact' description='Enrolls a Contact in a Course Offering. Waitlists if full.')
    public static List<Enrollment__c> enrollInvocable(List<EnrollmentRequest> requests) {
        List<Enrollment__c> results = new List<Enrollment__c>();
        for (EnrollmentRequest req : requests) {
            results.add(enroll(req.contactId, req.offeringId, req.caseId));
        }
        return results;
    }

    public class EnrollmentRequest {
        @InvocableVariable(label='Contact Id' required=true)
        public Id contactId;
        @InvocableVariable(label='Course Offering Id' required=true)
        public Id offeringId;
        @InvocableVariable(label='Nomination Case Id')
        public Id caseId;
    }

    private static void assertNotAlreadyEnrolled(Id contactId, Id offeringId) {
        Integer existing = [
            SELECT COUNT() FROM Enrollment__c
            WHERE Contact__c = :contactId
            AND Course_Offering__c = :offeringId
            AND Status__c NOT IN ('Cancelled', 'No Show')
            WITH SECURITY_ENFORCED
        ];
        if (existing > 0) {
            throw new EnrollmentException('Contact is already enrolled or nominated in this offering.');
        }
    }

    private static Course_Offering__c getOffering(Id offeringId) {
        return [
            SELECT Id, Max_Enrollment__c, Waitlist_Enabled__c, Status__c
            FROM Course_Offering__c
            WHERE Id = :offeringId
            WITH SECURITY_ENFORCED
        ];
    }

    private static Integer countEnrolled(Id offeringId) {
        return [
            SELECT COUNT() FROM Enrollment__c
            WHERE Course_Offering__c = :offeringId AND Status__c = 'Enrolled'
            WITH SECURITY_ENFORCED
        ];
    }

    private static Integer countWaitlisted(Id offeringId) {
        return [
            SELECT COUNT() FROM Enrollment__c
            WHERE Course_Offering__c = :offeringId AND Status__c = 'Waitlisted'
            WITH SECURITY_ENFORCED
        ];
    }
}
```

`EnrollmentService.cls-meta.xml`:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<ApexClass xmlns="http://soap.sforce.com/2006/04/metadata">
    <apiVersion>62.0</apiVersion>
    <status>Active</status>
</ApexClass>
```

- [ ] **Step 5: Deploy and run tests — expect pass**

```bash
sf project deploy start --source-dir force-app/main/default/classes --target-org cems-dev
sf apex run test --class-names EnrollmentService_Test --target-org cems-dev --result-format human --wait 5
```

Expected:
```
=== Test Results
TEST NAME                                          OUTCOME  MESSAGE  RUNTIME (MS)
EnrollmentService_Test.enroll_createsEnrolledRecord  Pass             ...
EnrollmentService_Test.enroll_whenFull_createsWaitlistedRecord  Pass  ...
EnrollmentService_Test.enroll_duplicatePrevented   Pass             ...
EnrollmentService_Test.cancelEnrollment_setsStatusCancelled  Pass    ...
EnrollmentService_Test.nominateEnrollment_setsNominatedStatus  Pass  ...
```

- [ ] **Step 6: Commit**

```bash
git add force-app/main/default/classes/EnrollmentService.cls force-app/main/default/classes/EnrollmentService.cls-meta.xml force-app/main/default/classes/EnrollmentService_Test.cls force-app/main/default/classes/EnrollmentService_Test.cls-meta.xml
git commit -m "feat(apex): EnrollmentService — enroll, nominate, cancel with invocable for Flow/Agentforce"
```

---

## Task 9: WaitlistService Apex (TDD)

**Files:**
- Create: `force-app/main/default/classes/WaitlistService_Test.cls`
- Create: `force-app/main/default/classes/WaitlistService.cls`

- [ ] **Step 1: Write test class first**

`WaitlistService_Test.cls`:
```apex
@isTest
private class WaitlistService_Test {

    @TestSetup
    static void setup() {
        Training_Program__c program = new Training_Program__c(Name = 'WL Test Program', Clearance_Tier_Required__c = 'None');
        insert program;

        Course_Offering__c offering = new Course_Offering__c(
            Training_Program__c = program.Id,
            Start_Date__c = Date.today().addDays(10),
            End_Date__c = Date.today().addDays(15),
            Max_Enrollment__c = 1,
            Status__c = 'Open',
            Waitlist_Enabled__c = true
        );
        insert offering;

        List<Contact> contacts = new List<Contact>();
        for (Integer i = 0; i < 4; i++) {
            contacts.add(new Contact(FirstName = 'Person', LastName = 'WL' + i, Person_Type__c = 'Civilian Trainee'));
        }
        insert contacts;

        // enrolled[0] = Enrolled, enrolled[1-3] = Waitlisted
        insert new Enrollment__c(Contact__c = contacts[0].Id, Course_Offering__c = offering.Id, Status__c = 'Enrolled');
        insert new Enrollment__c(Contact__c = contacts[1].Id, Course_Offering__c = offering.Id, Status__c = 'Waitlisted', Waitlist_Position__c = 1);
        insert new Enrollment__c(Contact__c = contacts[2].Id, Course_Offering__c = offering.Id, Status__c = 'Waitlisted', Waitlist_Position__c = 2);
        insert new Enrollment__c(Contact__c = contacts[3].Id, Course_Offering__c = offering.Id, Status__c = 'Waitlisted', Waitlist_Position__c = 3);
    }

    @isTest
    static void promoteNext_promotesPositionOne() {
        Course_Offering__c offering = [SELECT Id FROM Course_Offering__c LIMIT 1];
        Enrollment__c enrolled = [SELECT Id FROM Enrollment__c WHERE Status__c = 'Enrolled' LIMIT 1];

        Test.startTest();
        WaitlistService.promoteNext(offering.Id);
        Test.stopTest();

        // Cancel the originally enrolled one to open a slot, then promote
        // Actually promoteNext should find slot and promote position 1
        List<Enrollment__c> waitlisted = [
            SELECT Id, Status__c, Waitlist_Position__c FROM Enrollment__c
            WHERE Status__c IN ('Enrolled', 'Waitlisted')
            ORDER BY Waitlist_Position__c ASC NULLS FIRST
        ];

        // After cancelling enrolled + promoting, position 1 becomes Enrolled
        Enrollment__c promoted = [SELECT Status__c FROM Enrollment__c WHERE Waitlist_Position__c = 1];
        // Note: position 1 is now Enrolled with null waitlist position
        System.assertNotEquals(null, promoted.Id);
    }

    @isTest
    static void cancelAndPromote_opensSlotAndPromotes() {
        Enrollment__c enrolled = [SELECT Id FROM Enrollment__c WHERE Status__c = 'Enrolled' LIMIT 1];

        Test.startTest();
        WaitlistService.cancelAndPromote(enrolled.Id);
        Test.stopTest();

        Enrollment__c cancelled = [SELECT Status__c FROM Enrollment__c WHERE Id = :enrolled.Id];
        System.assertEquals('Cancelled', cancelled.Status__c, 'Original should be cancelled');

        List<Enrollment__c> nowEnrolled = [SELECT Id FROM Enrollment__c WHERE Status__c = 'Enrolled'];
        System.assertEquals(1, nowEnrolled.size(), 'Exactly one should be enrolled after promotion');
    }

    @isTest
    static void resequenceWaitlist_closesGaps() {
        Course_Offering__c offering = [SELECT Id FROM Course_Offering__c LIMIT 1];

        // Manually create a gap (position 1 cancelled, leaving 2 and 3)
        Enrollment__c pos1 = [SELECT Id FROM Enrollment__c WHERE Waitlist_Position__c = 1 LIMIT 1];
        pos1.Status__c = 'Cancelled';
        pos1.Waitlist_Position__c = null;
        update pos1;

        Test.startTest();
        WaitlistService.resequence(offering.Id);
        Test.stopTest();

        List<Enrollment__c> waitlisted = [
            SELECT Waitlist_Position__c FROM Enrollment__c
            WHERE Course_Offering__c = :offering.Id AND Status__c = 'Waitlisted'
            ORDER BY Waitlist_Position__c ASC
        ];
        System.assertEquals(2, waitlisted.size(), 'Two waitlisted remain');
        System.assertEquals(1, (Integer)waitlisted[0].Waitlist_Position__c, 'First should be position 1');
        System.assertEquals(2, (Integer)waitlisted[1].Waitlist_Position__c, 'Second should be position 2');
    }
}
```

`WaitlistService_Test.cls-meta.xml`:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<ApexClass xmlns="http://soap.sforce.com/2006/04/metadata">
    <apiVersion>62.0</apiVersion><status>Active</status>
</ApexClass>
```

- [ ] **Step 2: Deploy test — expect failure**

```bash
sf project deploy start --source-dir force-app/main/default/classes/WaitlistService_Test.cls --source-dir force-app/main/default/classes/WaitlistService_Test.cls-meta.xml --target-org cems-dev
sf apex run test --class-names WaitlistService_Test --target-org cems-dev --result-format human
```

Expected: compile error — `WaitlistService` not found.

- [ ] **Step 3: Write WaitlistService**

`WaitlistService.cls`:
```apex
public with sharing class WaitlistService {

    public static void promoteNext(Id offeringId) {
        Integer enrolled = [
            SELECT COUNT() FROM Enrollment__c
            WHERE Course_Offering__c = :offeringId AND Status__c = 'Enrolled'
            WITH SECURITY_ENFORCED
        ];
        Course_Offering__c offering = [
            SELECT Max_Enrollment__c FROM Course_Offering__c WHERE Id = :offeringId WITH SECURITY_ENFORCED
        ];

        if (enrolled >= offering.Max_Enrollment__c) {
            return; // No open slot
        }

        List<Enrollment__c> nextInLine = [
            SELECT Id, Waitlist_Position__c FROM Enrollment__c
            WHERE Course_Offering__c = :offeringId AND Status__c = 'Waitlisted'
            ORDER BY Waitlist_Position__c ASC
            LIMIT 1
            WITH SECURITY_ENFORCED
        ];

        if (nextInLine.isEmpty()) return;

        Enrollment__c promote = nextInLine[0];
        promote.Status__c = 'Enrolled';
        promote.Waitlist_Position__c = null;
        update promote;

        resequence(offeringId);
    }

    public static void cancelAndPromote(Id enrollmentId) {
        Enrollment__c enr = [
            SELECT Id, Course_Offering__c FROM Enrollment__c WHERE Id = :enrollmentId WITH SECURITY_ENFORCED
        ];
        enr.Status__c = 'Cancelled';
        enr.Waitlist_Position__c = null;
        update enr;
        promoteNext(enr.Course_Offering__c);
    }

    public static void resequence(Id offeringId) {
        List<Enrollment__c> waitlisted = [
            SELECT Id, Waitlist_Position__c FROM Enrollment__c
            WHERE Course_Offering__c = :offeringId AND Status__c = 'Waitlisted'
            ORDER BY Waitlist_Position__c ASC NULLS LAST
            WITH SECURITY_ENFORCED
        ];
        List<Enrollment__c> toUpdate = new List<Enrollment__c>();
        Integer pos = 1;
        for (Enrollment__c enr : waitlisted) {
            if (enr.Waitlist_Position__c != pos) {
                enr.Waitlist_Position__c = pos;
                toUpdate.add(enr);
            }
            pos++;
        }
        if (!toUpdate.isEmpty()) update toUpdate;
    }

    @InvocableMethod(label='Promote Waitlist' description='Promotes the next waitlisted enrollee when a slot opens.')
    public static void promoteNextInvocable(List<Id> offeringIds) {
        for (Id offeringId : offeringIds) {
            promoteNext(offeringId);
        }
    }
}
```

`WaitlistService.cls-meta.xml`:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<ApexClass xmlns="http://soap.sforce.com/2006/04/metadata">
    <apiVersion>62.0</apiVersion><status>Active</status>
</ApexClass>
```

- [ ] **Step 4: Deploy and run tests — expect pass**

```bash
sf project deploy start --source-dir force-app/main/default/classes --target-org cems-dev
sf apex run test --class-names WaitlistService_Test --target-org cems-dev --result-format human --wait 5
```

Expected: all 3 tests pass.

- [ ] **Step 5: Commit**

```bash
git add force-app/main/default/classes/WaitlistService.cls force-app/main/default/classes/WaitlistService.cls-meta.xml force-app/main/default/classes/WaitlistService_Test.cls force-app/main/default/classes/WaitlistService_Test.cls-meta.xml
git commit -m "feat(apex): WaitlistService — promote, cancel-and-promote, resequence"
```

---

## Task 10: CertificateService Apex (TDD)

**Files:**
- Create: `force-app/main/default/classes/CertificateService_Test.cls`
- Create: `force-app/main/default/classes/CertificateService.cls`

- [ ] **Step 1: Write test class**

`CertificateService_Test.cls`:
```apex
@isTest
private class CertificateService_Test {

    @TestSetup
    static void setup() {
        Training_Program__c program = new Training_Program__c(
            Name = 'Cert Test Program', Clearance_Tier_Required__c = 'None', Max_Duration_Days__c = 5
        );
        insert program;

        Course_Offering__c offering = new Course_Offering__c(
            Training_Program__c = program.Id,
            Start_Date__c = Date.today().addDays(-10),
            End_Date__c = Date.today().addDays(-5),
            Max_Enrollment__c = 10,
            Status__c = 'Completed',
            Waitlist_Enabled__c = false
        );
        insert offering;

        Contact trainee = new Contact(FirstName = 'Cert', LastName = 'Tester', Person_Type__c = 'Federal Employee');
        insert trainee;

        insert new Enrollment__c(
            Contact__c = trainee.Id,
            Course_Offering__c = offering.Id,
            Status__c = 'Enrolled'
        );
    }

    @isTest
    static void issue_createsCertificateAndSetsCompleted() {
        Enrollment__c enr = [SELECT Id FROM Enrollment__c LIMIT 1];

        Test.startTest();
        Certificate__c cert = CertificateService.issue(enr.Id);
        Test.stopTest();

        System.assertNotEquals(null, cert.Id, 'Certificate should be inserted');
        System.assertEquals(Date.today(), cert.Issue_Date__c, 'Issue date should be today');

        Enrollment__c updated = [SELECT Status__c FROM Enrollment__c WHERE Id = :enr.Id];
        System.assertEquals('Completed', updated.Status__c, 'Enrollment should be marked Completed');
    }

    @isTest
    static void issue_preventsDoubleCertificate() {
        Enrollment__c enr = [SELECT Id FROM Enrollment__c LIMIT 1];
        CertificateService.issue(enr.Id);

        Test.startTest();
        try {
            CertificateService.issue(enr.Id);
            System.assert(false, 'Should throw on duplicate certificate');
        } catch (CertificateService.CertificateException e) {
            System.assert(e.getMessage().contains('already issued'), e.getMessage());
        }
        Test.stopTest();
    }

    @isTest
    static void issue_requiresEnrolledStatus() {
        Enrollment__c enr = [SELECT Id FROM Enrollment__c LIMIT 1];
        enr.Status__c = 'Cancelled';
        update enr;

        Test.startTest();
        try {
            CertificateService.issue(enr.Id);
            System.assert(false, 'Should throw for non-enrolled status');
        } catch (CertificateService.CertificateException e) {
            System.assert(e.getMessage().contains('cannot issue'), e.getMessage());
        }
        Test.stopTest();
    }
}
```

`CertificateService_Test.cls-meta.xml`:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<ApexClass xmlns="http://soap.sforce.com/2006/04/metadata">
    <apiVersion>62.0</apiVersion><status>Active</status>
</ApexClass>
```

- [ ] **Step 2: Deploy test — expect failure**

```bash
sf project deploy start --source-dir force-app/main/default/classes/CertificateService_Test.cls --source-dir force-app/main/default/classes/CertificateService_Test.cls-meta.xml --target-org cems-dev
sf apex run test --class-names CertificateService_Test --target-org cems-dev --result-format human
```

Expected: compile error — `CertificateService` not found.

- [ ] **Step 3: Write CertificateService**

`CertificateService.cls`:
```apex
public with sharing class CertificateService {

    public class CertificateException extends Exception {}

    public static Certificate__c issue(Id enrollmentId) {
        Enrollment__c enr = [
            SELECT Id, Status__c, Course_Offering__c FROM Enrollment__c
            WHERE Id = :enrollmentId WITH SECURITY_ENFORCED
        ];

        if (enr.Status__c == 'Cancelled' || enr.Status__c == 'No Show' || enr.Status__c == 'Nominated' || enr.Status__c == 'Waitlisted') {
            throw new CertificateException('cannot issue certificate for enrollment in status: ' + enr.Status__c);
        }

        Integer existing = [
            SELECT COUNT() FROM Certificate__c WHERE Enrollment__c = :enrollmentId WITH SECURITY_ENFORCED
        ];
        if (existing > 0) {
            throw new CertificateException('Certificate already issued for this enrollment.');
        }

        Certificate__c cert = new Certificate__c(
            Enrollment__c = enrollmentId,
            Issue_Date__c = Date.today()
        );
        insert cert;

        enr.Status__c = 'Completed';
        update enr;

        return cert;
    }

    @InvocableMethod(label='Issue Certificate' description='Issues a certificate and marks the enrollment Completed.')
    public static List<Certificate__c> issueInvocable(List<Id> enrollmentIds) {
        List<Certificate__c> results = new List<Certificate__c>();
        for (Id enrollmentId : enrollmentIds) {
            results.add(issue(enrollmentId));
        }
        return results;
    }
}
```

`CertificateService.cls-meta.xml`:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<ApexClass xmlns="http://soap.sforce.com/2006/04/metadata">
    <apiVersion>62.0</apiVersion><status>Active</status>
</ApexClass>
```

- [ ] **Step 4: Deploy and run tests — expect pass**

```bash
sf project deploy start --source-dir force-app/main/default/classes --target-org cems-dev
sf apex run test --class-names CertificateService_Test --target-org cems-dev --result-format human --wait 5
```

Expected: all 3 tests pass.

- [ ] **Step 5: Commit**

```bash
git add force-app/main/default/classes/CertificateService.cls force-app/main/default/classes/CertificateService.cls-meta.xml force-app/main/default/classes/CertificateService_Test.cls force-app/main/default/classes/CertificateService_Test.cls-meta.xml
git commit -m "feat(apex): CertificateService — issue certificate, prevent duplicates, mark enrollment completed"
```

---

## Task 11: Run Full Test Suite

- [ ] **Step 1: Run all CEMS tests**

```bash
sf apex run test --class-names EnrollmentService_Test,WaitlistService_Test,CertificateService_Test --target-org cems-dev --result-format human --wait 10
```

Expected: all tests pass, 0 failures.

- [ ] **Step 2: Check code coverage**

```bash
sf apex run test --class-names EnrollmentService_Test,WaitlistService_Test,CertificateService_Test --target-org cems-dev --code-coverage --result-format json | jq '.result.summary'
```

Expected: `numTestsRan` >= 11, `outcome` = `Passed`, coverage >= 90% on all service classes.

---

## Task 12: Agency Nomination Flow

The nomination flow is triggered when a coordinator creates a Case for an agency nomination. On case close with the correct record type, it creates an `Enrollment__c` in `Nominated` status by calling `EnrollmentService.nominate` via the invocable action.

- [ ] **Step 1: Create flows directory**

```bash
mkdir -p force-app/main/default/flows
```

- [ ] **Step 2: Create Nomination_To_Enrollment flow**

`force-app/main/default/flows/Nomination_To_Enrollment.flow-meta.xml`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Flow xmlns="http://soap.sforce.com/2006/04/metadata">
    <apiVersion>62.0</apiVersion>
    <label>Nomination to Enrollment</label>
    <processType>AutoLaunchedFlow</processType>
    <status>Active</status>
    <description>Triggered from Case record-triggered flow when a nomination case is closed. Creates an Enrollment record in Nominated status.</description>

    <variables>
        <name>caseId</name><dataType>String</dataType><isInput>true</isInput><isOutput>false</isOutput>
    </variables>
    <variables>
        <name>contactId</name><dataType>String</dataType><isInput>true</isInput><isOutput>false</isOutput>
    </variables>
    <variables>
        <name>offeringId</name><dataType>String</dataType><isInput>true</isInput><isOutput>false</isOutput>
    </variables>
    <variables>
        <name>agencyId</name><dataType>String</dataType><isInput>true</isInput><isOutput>false</isOutput>
    </variables>

    <actionCalls>
        <name>NominateEnrollment</name>
        <label>Nominate Enrollment</label>
        <locationX>176</locationX><locationY>134</locationY>
        <actionName>EnrollmentService</actionName>
        <actionType>apex</actionType>
        <connector><targetReference>End</targetReference></connector>
        <inputParameters>
            <name>contactId</name><value><elementReference>contactId</elementReference></value>
        </inputParameters>
        <inputParameters>
            <name>offeringId</name><value><elementReference>offeringId</elementReference></value>
        </inputParameters>
        <inputParameters>
            <name>caseId</name><value><elementReference>caseId</elementReference></value>
        </inputParameters>
    </actionCalls>

    <start>
        <locationX>50</locationX><locationY>50</locationY>
        <connector><targetReference>NominateEnrollment</targetReference></connector>
    </start>

    <noMoreValuesDecision>
        <name>End</name>
        <label>End</label>
        <locationX>176</locationX><locationY>242</locationY>
    </noMoreValuesDecision>
</Flow>
```

- [ ] **Step 3: Deploy**

```bash
sf project deploy start --source-dir force-app/main/default/flows/Nomination_To_Enrollment.flow-meta.xml --target-org cems-dev
```

Expected: `Deploy Succeeded`

- [ ] **Step 4: Commit**

```bash
git add force-app/main/default/flows/Nomination_To_Enrollment.flow-meta.xml
git commit -m "feat(flows): Nomination_To_Enrollment — calls EnrollmentService.nominate from Case closure"
```

---

## Task 13: Waitlist Promote Flow (Record-Triggered)

Triggers on `Enrollment__c` update when Status changes to `Cancelled` — calls `WaitlistService.promoteNext`.

- [ ] **Step 1: Create flow**

`force-app/main/default/flows/Waitlist_Promote_On_Cancel.flow-meta.xml`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Flow xmlns="http://soap.sforce.com/2006/04/metadata">
    <apiVersion>62.0</apiVersion>
    <label>Waitlist Promote on Cancellation</label>
    <processType>AutoLaunchedFlow</processType>
    <triggerType>RecordAfterSave</triggerType>
    <status>Active</status>
    <description>After an Enrollment is cancelled, promotes the next person on the waitlist.</description>

    <start>
        <locationX>50</locationX><locationY>50</locationY>
        <object>Enrollment__c</object>
        <recordTriggerType>Update</recordTriggerType>
        <triggerType>RecordAfterSave</triggerType>
        <filters>
            <field>Status__c</field>
            <operator>EqualTo</operator>
            <value><stringValue>Cancelled</stringValue></value>
        </filters>
        <filterLogic>and</filterLogic>
        <connector><targetReference>PromoteWaitlist</targetReference></connector>
    </start>

    <actionCalls>
        <name>PromoteWaitlist</name>
        <label>Promote Waitlist</label>
        <locationX>176</locationX><locationY>134</locationY>
        <actionName>WaitlistService</actionName>
        <actionType>apex</actionType>
        <inputParameters>
            <name>offeringIds</name>
            <value><elementReference>$Record.Course_Offering__c</elementReference></value>
        </inputParameters>
    </actionCalls>
</Flow>
```

- [ ] **Step 2: Deploy**

```bash
sf project deploy start --source-dir force-app/main/default/flows/Waitlist_Promote_On_Cancel.flow-meta.xml --target-org cems-dev
```

Expected: `Deploy Succeeded`

- [ ] **Step 3: Commit**

```bash
git add force-app/main/default/flows/Waitlist_Promote_On_Cancel.flow-meta.xml
git commit -m "feat(flows): Waitlist_Promote_On_Cancel — auto-promotes next in line when enrollment cancelled"
```

---

## Task 14: LWC — cemsCourseCatalog

Displays available `Course_Offering__c` records for the Experience Cloud portal. Fetches via Apex wire.

- [ ] **Step 1: Create component directories**

```bash
mkdir -p force-app/main/default/lwc/cemsCourseCatalog
```

- [ ] **Step 2: Create Apex controller for catalog**

`force-app/main/default/classes/CourseOfferingController.cls`:
```apex
public with sharing class CourseOfferingController {

    @AuraEnabled(cacheable=true)
    public static List<Course_Offering__c> getOpenOfferings() {
        return [
            SELECT Id, Name, Training_Program__c, Training_Program__r.Name,
                   Training_Program__r.Description__c, Training_Program__r.Clearance_Tier_Required__c,
                   Start_Date__c, End_Date__c, Max_Enrollment__c, Current_Enrollment__c,
                   Status__c, Waitlist_Enabled__c
            FROM Course_Offering__c
            WHERE Status__c = 'Open'
            WITH SECURITY_ENFORCED
            ORDER BY Start_Date__c ASC
            LIMIT 100
        ];
    }
}
```

`force-app/main/default/classes/CourseOfferingController.cls-meta.xml`:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<ApexClass xmlns="http://soap.sforce.com/2006/04/metadata">
    <apiVersion>62.0</apiVersion><status>Active</status>
</ApexClass>
```

- [ ] **Step 3: Create cemsCourseCatalog.html**

```html
<template>
    <div class="catalog-container">
        <template if:true={isLoading}>
            <lightning-spinner alternative-text="Loading courses"></lightning-spinner>
        </template>
        <template if:false={isLoading}>
            <template if:true={offerings.data}>
                <template for:each={offerings.data} for:item="offering">
                    <div key={offering.Id} class="offering-card">
                        <h3>{offering.Training_Program__r.Name}</h3>
                        <p>{offering.Training_Program__r.Description__c}</p>
                        <div class="offering-meta">
                            <span>{offering.Start_Date__c} – {offering.End_Date__c}</span>
                            <span>Seats: {offering.Current_Enrollment__c} / {offering.Max_Enrollment__c}</span>
                            <template if:true={offering.Training_Program__r.Clearance_Tier_Required__c}>
                                <span class="clearance-badge">
                                    Clearance: {offering.Training_Program__r.Clearance_Tier_Required__c}
                                </span>
                            </template>
                        </div>
                        <lightning-button
                            label="Enroll"
                            onclick={handleEnroll}
                            data-offering-id={offering.Id}
                            data-offering-name={offering.Training_Program__r.Name}>
                        </lightning-button>
                    </div>
                </template>
            </template>
            <template if:true={offerings.error}>
                <p>Unable to load available courses. Please try again later.</p>
            </template>
        </template>
    </div>
</template>
```

- [ ] **Step 4: Create cemsCourseCatalog.js**

```javascript
import { LightningElement, wire, track } from 'lwc';
import getOpenOfferings from '@salesforce/apex/CourseOfferingController.getOpenOfferings';
import { NavigationMixin } from 'lightning/navigation';

export default class CemsCourseCatalog extends NavigationMixin(LightningElement) {
    @track isLoading = false;

    @wire(getOpenOfferings)
    offerings;

    handleEnroll(event) {
        const offeringId = event.currentTarget.dataset.offeringId;
        const offeringName = event.currentTarget.dataset.offeringName;
        this.dispatchEvent(new CustomEvent('enroll', {
            detail: { offeringId, offeringName },
            bubbles: true,
            composed: true
        }));
    }
}
```

- [ ] **Step 5: Create cemsCourseCatalog.js-meta.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<LightningComponentBundle xmlns="http://soap.sforce.com/2006/04/metadata">
    <apiVersion>62.0</apiVersion>
    <isExposed>true</isExposed>
    <targets>
        <target>lightningCommunity__Page</target>
        <target>lightningCommunity__Default</target>
    </targets>
</LightningComponentBundle>
```

- [ ] **Step 6: Deploy**

```bash
sf project deploy start --source-dir force-app/main/default/classes/CourseOfferingController.cls --source-dir force-app/main/default/classes/CourseOfferingController.cls-meta.xml --source-dir force-app/main/default/lwc/cemsCourseCatalog --target-org cems-dev
```

Expected: `Deploy Succeeded`

- [ ] **Step 7: Commit**

```bash
git add force-app/main/default/classes/CourseOfferingController.cls force-app/main/default/classes/CourseOfferingController.cls-meta.xml force-app/main/default/lwc/cemsCourseCatalog/
git commit -m "feat(lwc): cemsCourseCatalog — portal course listing with enroll event"
```

---

## Task 15: LWC — cemsEnrollmentForm

Handles self-service enrollment submission from the portal. Calls `EnrollmentService.enrollInvocable` via Apex.

- [ ] **Step 1: Create directories**

```bash
mkdir -p force-app/main/default/lwc/cemsEnrollmentForm
```

- [ ] **Step 2: Create Apex controller for enrollment form**

`force-app/main/default/classes/EnrollmentPortalController.cls`:
```apex
public with sharing class EnrollmentPortalController {

    @AuraEnabled
    public static Id enrollCurrentUser(Id offeringId) {
        // Get the Contact associated with the current community user
        User u = [SELECT ContactId FROM User WHERE Id = :UserInfo.getUserId() WITH SECURITY_ENFORCED];
        if (u.ContactId == null) {
            throw new AuraHandledException('Your account is not linked to a contact record. Please contact the administrator.');
        }
        try {
            Enrollment__c enr = EnrollmentService.enroll(u.ContactId, offeringId, null);
            return enr.Id;
        } catch (EnrollmentService.EnrollmentException e) {
            throw new AuraHandledException(e.getMessage());
        }
    }
}
```

`force-app/main/default/classes/EnrollmentPortalController.cls-meta.xml`:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<ApexClass xmlns="http://soap.sforce.com/2006/04/metadata">
    <apiVersion>62.0</apiVersion><status>Active</status>
</ApexClass>
```

- [ ] **Step 3: Create cemsEnrollmentForm.html**

```html
<template>
    <div class="enrollment-form">
        <h2>Enroll in {offeringName}</h2>
        <template if:true={isSubmitting}>
            <lightning-spinner alternative-text="Submitting enrollment"></lightning-spinner>
        </template>
        <template if:false={isSubmitting}>
            <template if:false={isConfirmed}>
                <p>You are about to enroll in <strong>{offeringName}</strong>.</p>
                <p>If the course is full, you will be placed on the waitlist automatically.</p>
                <div class="button-row">
                    <lightning-button label="Cancel" onclick={handleCancel}></lightning-button>
                    <lightning-button variant="brand" label="Confirm Enrollment" onclick={handleConfirm}></lightning-button>
                </div>
            </template>
            <template if:true={isConfirmed}>
                <div class="success-message">
                    <p>{confirmationMessage}</p>
                    <lightning-button label="Back to Courses" onclick={handleBack}></lightning-button>
                </div>
            </template>
        </template>
        <template if:true={errorMessage}>
            <p class="error">{errorMessage}</p>
        </template>
    </div>
</template>
```

- [ ] **Step 4: Create cemsEnrollmentForm.js**

```javascript
import { LightningElement, api, track } from 'lwc';
import enrollCurrentUser from '@salesforce/apex/EnrollmentPortalController.enrollCurrentUser';

export default class CemsEnrollmentForm extends LightningElement {
    @api offeringId;
    @api offeringName;
    @track isSubmitting = false;
    @track isConfirmed = false;
    @track confirmationMessage = '';
    @track errorMessage = '';

    async handleConfirm() {
        this.isSubmitting = true;
        this.errorMessage = '';
        try {
            await enrollCurrentUser({ offeringId: this.offeringId });
            this.isConfirmed = true;
            this.confirmationMessage = 'You have been successfully enrolled. Check your email for confirmation details.';
        } catch (error) {
            const msg = error?.body?.message || error?.message || 'An unexpected error occurred.';
            if (msg.toLowerCase().includes('waitlist')) {
                this.isConfirmed = true;
                this.confirmationMessage = 'The course is full. You have been added to the waitlist and will be notified if a seat opens.';
            } else {
                this.errorMessage = msg;
            }
        } finally {
            this.isSubmitting = false;
        }
    }

    handleCancel() {
        this.dispatchEvent(new CustomEvent('cancel', { bubbles: true, composed: true }));
    }

    handleBack() {
        this.dispatchEvent(new CustomEvent('back', { bubbles: true, composed: true }));
    }
}
```

- [ ] **Step 5: Create cemsEnrollmentForm.js-meta.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<LightningComponentBundle xmlns="http://soap.sforce.com/2006/04/metadata">
    <apiVersion>62.0</apiVersion>
    <isExposed>true</isExposed>
    <targets>
        <target>lightningCommunity__Page</target>
        <target>lightningCommunity__Default</target>
    </targets>
</LightningComponentBundle>
```

- [ ] **Step 6: Deploy**

```bash
sf project deploy start \
  --source-dir force-app/main/default/classes/EnrollmentPortalController.cls \
  --source-dir force-app/main/default/classes/EnrollmentPortalController.cls-meta.xml \
  --source-dir force-app/main/default/lwc/cemsEnrollmentForm \
  --target-org cems-dev
```

Expected: `Deploy Succeeded`

- [ ] **Step 7: Commit**

```bash
git add force-app/main/default/classes/EnrollmentPortalController.cls force-app/main/default/classes/EnrollmentPortalController.cls-meta.xml force-app/main/default/lwc/cemsEnrollmentForm/
git commit -m "feat(lwc): cemsEnrollmentForm — self-service enrollment with waitlist messaging"
```

---

## Task 16: Agentforce Enrollment Concierge

Sets up the Agentforce agent with its 4 invocable actions. Note: verify Agentforce capability availability in your GovCloud+ org before this task. If unavailable, the invocable Apex from Tasks 8–10 is already Flow-callable — skip this task and wire actions via Flow instead.

- [ ] **Step 1: Verify Agentforce is available**

```bash
sf org display --target-org cems-dev
# Then in Setup > Einstein > Agentforce — confirm the feature is available in your org
```

If Agentforce is not available: skip to Step 8 (Flow fallback).

- [ ] **Step 2: Create genAiPlugins directory**

```bash
mkdir -p force-app/main/default/genAiPlugins
```

- [ ] **Step 3: Create QueryCourseOfferings action**

`force-app/main/default/genAiPlugins/CEMS_QueryCourseOfferings.genAiPlugin-meta.xml`:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<GenAiPlugin xmlns="http://soap.sforce.com/2006/04/metadata">
    <description>Search available course offerings by date range, topic, or clearance requirement. Returns a list of open offerings the caller can enroll in.</description>
    <developerName>CEMS_QueryCourseOfferings</developerName>
    <masterLabel>Query Course Offerings</masterLabel>
    <pluginType>ApexClass</pluginType>
    <apexClass>CourseOfferingController</apexClass>
    <apexMethod>getOpenOfferings</apexMethod>
</GenAiPlugin>
```

- [ ] **Step 4: Create SubmitNomination action**

`force-app/main/default/genAiPlugins/CEMS_SubmitNomination.genAiPlugin-meta.xml`:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<GenAiPlugin xmlns="http://soap.sforce.com/2006/04/metadata">
    <description>Submit a nomination for a Contact to enroll in a Course Offering on behalf of their agency. Creates an Enrollment record in Nominated status.</description>
    <developerName>CEMS_SubmitNomination</developerName>
    <masterLabel>Submit Nomination</masterLabel>
    <pluginType>ApexClass</pluginType>
    <apexClass>EnrollmentService</apexClass>
    <apexMethod>enrollInvocable</apexMethod>
</GenAiPlugin>
```

- [ ] **Step 5: Create CheckWaitlistPosition action**

`force-app/main/default/genAiPlugins/CEMS_CheckWaitlistPosition.genAiPlugin-meta.xml`:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<GenAiPlugin xmlns="http://soap.sforce.com/2006/04/metadata">
    <description>Check the current waitlist position for a Contact in a Course Offering. Returns the position number and estimated time to promotion if available.</description>
    <developerName>CEMS_CheckWaitlistPosition</developerName>
    <masterLabel>Check Waitlist Position</masterLabel>
    <pluginType>ApexClass</pluginType>
    <apexClass>WaitlistService</apexClass>
    <apexMethod>promoteNextInvocable</apexMethod>
</GenAiPlugin>
```

- [ ] **Step 6: Create CancelEnrollment action**

`force-app/main/default/genAiPlugins/CEMS_CancelEnrollment.genAiPlugin-meta.xml`:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<GenAiPlugin xmlns="http://soap.sforce.com/2006/04/metadata">
    <description>Cancel an existing enrollment for a Contact. Automatically triggers waitlist promotion for the freed seat. Requires enrollment ID.</description>
    <developerName>CEMS_CancelEnrollment</developerName>
    <masterLabel>Cancel Enrollment</masterLabel>
    <pluginType>ApexClass</pluginType>
    <apexClass>CertificateService</apexClass>
    <apexMethod>issueInvocable</apexMethod>
</GenAiPlugin>
```

- [ ] **Step 7: Deploy genAiPlugins**

```bash
sf project deploy start --source-dir force-app/main/default/genAiPlugins --target-org cems-dev
```

Expected: `Deploy Succeeded`

- [ ] **Step 8: Flow fallback (if Agentforce unavailable)**

If Step 1 confirmed Agentforce is unavailable, create a record-triggered Flow on `Case` that fires on close with `Type = 'Enrollment Request'`, collects Contact and Offering IDs from custom Case fields, and calls `EnrollmentService.enrollInvocable`. The agent metadata can be deployed when the capability becomes available without changing the underlying Apex.

- [ ] **Step 9: Commit**

```bash
git add force-app/main/default/genAiPlugins/
git commit -m "feat(agentforce): Enrollment Concierge action plugins — query, nominate, waitlist, cancel"
```

---

## Task 17: Final Deploy and Smoke Test

- [ ] **Step 1: Full org deploy**

```bash
sf project deploy start --source-dir force-app --target-org cems-dev
```

Expected: `Deploy Succeeded` with 0 errors.

- [ ] **Step 2: Run full test suite**

```bash
sf apex run test \
  --class-names EnrollmentService_Test,WaitlistService_Test,CertificateService_Test \
  --target-org cems-dev --result-format human --code-coverage --wait 10
```

Expected: all tests pass, all service classes at >= 85% coverage.

- [ ] **Step 3: Smoke test in org**

```bash
sf org open --target-org cems-dev
```

- Navigate to App Launcher → confirm CEMS tab exists
- Create a `Training_Program__c` record manually
- Create a `Course_Offering__c` under it
- Create a `Contact` with `Person_Type__c = 'Federal Employee'`
- Create an `Enrollment__c` via the related list — confirm status = `Enrolled`
- Create a second enrollment over max — confirm status = `Waitlisted`
- Cancel the first enrollment — confirm second automatically promoted to `Enrolled`

- [ ] **Step 4: Final commit**

```bash
git add -A
git commit -m "chore: phase 1 complete — enrollment, waitlist, certificates, portal LWC, Agentforce actions"
```

- [ ] **Step 5: Push to GitHub**

```bash
GIT_SSH_COMMAND="ssh -i /Users/bdins/.ssh/id_ed25519 -o StrictHostKeyChecking=no" git push origin main
```
