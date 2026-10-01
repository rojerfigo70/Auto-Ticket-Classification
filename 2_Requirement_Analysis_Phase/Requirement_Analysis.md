# Phase 2: Requirement Analysis Document
## Project Title: Auto Ticket Classification using Flow Designer

---

### Executive Metadata
| Attribute | Specification |
| :--- | :--- |
| **System Name** | Automated Incident Classification & Routing Subsystem |
| **Target Application** | Global Scope (`global`) |
| **Core Custom Table** | `u_incident_workflow` (Display Name: `Incident WorkFlow`) |
| **Document Version** | 1.0.0 |
| **Review Status** | Verified against ServiceNow Production Schema |

---

## 1. System Overview

The primary objective of the **Auto Ticket Classification** system is to transform raw, incoming incident requests submitted by campus stakeholders into accurately categorized, prioritized, and acknowledged service records. This document articulates the complete functional capabilities, non-functional constraints, and technical data schema underpinning the solution.

---

## 2. Functional Requirements (FR)

The functional requirements delineate the precise operational behaviors the ServiceNow instance must exhibit upon ticket submission.

| Requirement ID | Requirement Name | Description & Acceptance Criteria | Priority |
| :--- | :--- | :--- | :---: |
| **FR-01** | **Record Ingestion Trigger** | The system shall detect when a new record is inserted into the `u_incident_workflow` table (`Created` database trigger event) and immediately initiate the classification flow without requiring manual user dispatch. | Must Have |
| **FR-02** | **Keyword Analysis Engine** | The system shall parse the `short_description` (and `description` if provided) using case-insensitive substring evaluation against preconfigured keyword matrices representing four core campus operational domains: Network, Hardware, Access, and Performance. | Must Have |
| **FR-03** | **Parent Category Assignment** | Upon matching a designated keyword group, the flow shall update the ticket's `Category` field (`u_choice_6`) with the corresponding choice value (`network`, `hardware`, `access`, or `performance`). | Must Have |
| **FR-04** | **Dependent Subcategory Binding** | The system shall enforce and populate the dependent `Subcategory` field (`u_choice_7`) such that: <br>• `wi-fi` is only assignable when Category is `network`<br>• `projector` is only assignable when Category is `hardware`<br>• `forgot password` is only assignable when Category is `access`<br>• `slow computer` is only assignable when Category is `performance`. | Must Have |
| **FR-05** | **Default / Fallback Handling** | If an incoming record does not match any recognized keywords, the system shall default the Category to `hardware` (or general triage), leave the Subcategory blank, and flag the record for Tier-1 supervisory review. | Should Have |
| **FR-06** | **Automated Caller Notification** | Immediately following classification, the system shall generate an outbound HTML notification via `Send Email` dispatched to the `Caller` (`sys_user`) displaying the auto-generated ticket Number, Short description, assigned Category, Subcategory, and Current State. | Must Have |
| **FR-07** | **State Lifecycle Management** | Records shall initialize in the `New` state (`1`). The system shall allow support technicians to transition the state to `In progress` (`2`), `On hold` (`3`), `Resolved` (`6`), and `Closed` (`7`). | Must Have |
| **FR-08** | **Auto-Numbering Generation** | Every newly created record shall automatically be assigned an alphanumeric sequence starting with prefix `INC`, beginning at integer `500`, formatted with a 5-digit zero-padded sequence (e.g., `INC00500`, `INC00501`). | Must Have |

---

## 3. Non-Functional Requirements (NFR)

The non-functional requirements define the quality attributes, performance envelopes, maintainability, and security postures required of the system.

| Requirement ID | Quality Attribute | Specification & Threshold |
| :--- | :--- | :--- |
| **NFR-01** | **No-Code / Low-Code Maintainability** | The workflow logic must be 100% maintainable via the ServiceNow Flow Designer visual interface. Helpdesk administrators must be able to add, modify, or remove keyword triggers without modifying server-side script files or compiling code. |
| **NFR-02** | **Execution Latency & Performance** | The execution time from database insert (`sys_created_on`) to classification update and email queuing must not exceed **1,500 milliseconds (1.5 seconds)** under standard instance concurrency. |
| **NFR-03** | **Data Integrity & Referential Constraints**| All user references (`Caller`, `Assigned to`) must validate against active records in `sys_user`. Group assignments must validate against active groups in `sys_user_group`. Choice values must strictly align with dictionary definitions (`sys_choice`). |
| **NFR-04** | **Platform Upgrade Compatibility** | All customizations must adhere to ServiceNow standard configuration best practices, built within the Global application scope using native Flow Designer actions, ensuring zero regression across platform upgrades (Vancouver, Washington DC, Xanadu). |
| **NFR-05** | **Auditability and Execution Logging** | Every automated classification transaction must maintain an immutable execution audit record in `sys_flow_context`, allowing administrators to inspect trigger inputs, conditional branch evaluations, and step durations. |
| **NFR-06** | **Email Gateway Delivery Integrity** | Email generation must output valid RFC-compliant MIME emails written to the instance `sys_email` table, ready for SMTP dispatch by the ServiceNow mail delivery scheduler. |

---

## 4. Custom Table Schema Specification: `u_incident_workflow`

To isolate this automated workflow and preserve clean separation from core incident records, the solution deploys a specialized table: **`Incident WorkFlow [u_incident_workflow]`**.

### 4.1 Table Technical Metadata
| Parameter | Value | Reference / Notes |
| :--- | :--- | :--- |
| **Label** | `Incident WorkFlow` | Display label shown in Application Navigator |
| **Name** | `u_incident_workflow` | Database physical table name |
| **Extends Table** | `None` (Base Table) | Standalone custom table |
| **Application** | `Global` | Native Global scope (`sys_id: global`) |
| **Live Update Set** | `Project Update Set` | Captured under customer updates |

---

### 4.2 Data Dictionary Specification

The table consists of 9 core operational fields configured as follows:

| Field Label | Database Column Name | Data Type | Max Length | Mandatory | Reference Table / Choices | Default Value / Formula |
| :--- | :--- | :--- | :---: | :---: | :--- | :--- |
| **Number** | `u_number` | String | 40 | No (Read Only) | Auto-numbered sequence | Prefix: `INC`, Number: `500`, Digits: `5` |
| **Caller** | `u_caller` | Reference | 32 | Yes | `sys_user` | `javascript:gs.getUserID()` (Default logged-in user) |
| **Category** | `u_choice_6` | String (Choice) | 40 | No | Dropdown with `-- None --`<br>• `network`<br>• `hardware`<br>• `access`<br>• `performance` | `-- None --` |
| **Subcategory** | `u_choice_7` | String (Choice) | 40 | No | Dropdown with `-- None --`<br>Dependent on: `u_choice_6`<br>• `wi-fi`<br>• `projector`<br>• `forgot password`<br>• `slow computer` | `-- None --` |
| **Short description** | `u_short_description` | String | 255 | **Yes** | Text input | None |
| **Description** | `u_description` | String | 4000 | No | Multiline text | None |
| **State** | `u_state` | Integer (Choice) | 40 | No | Choice Dropdown:<br>• `1` (New)<br>• `2` (In progress)<br>• `3` (On hold)<br>• `6` (Resolved)<br>• `7` (Closed) | `1` (New) |
| **Assigned group** | `u_assignment_group` | Reference | 32 | No | `sys_user_group` | None |
| **Assigned to** | `u_assigned_to` | Reference | 32 | No | `sys_user` | None (Filtered by Assigned group) |

---

## 5. Controlled Choice Lists and Dependent Mappings

### 5.1 Category Choice List (`u_choice_6`)
The Category field serves as the top-level taxonomic classifier.

| Sequence | Label | Value (Stored in DB) | Inactive | Language |
| :---: | :--- | :--- | :---: | :---: |
| 0 | **Network** | `network` | false | `en` |
| 1 | **Hardware** | `hardware` | false | `en` |
| 2 | **Access** | `access` | false | `en` |
| 3 | **Performance** | `performance` | false | `en` |

### 5.2 Subcategory Choice List (`u_choice_7`)
The Subcategory field is strictly configured with a **Dependent Field** reference pointing to `u_choice_6` (`Category`). The choice values are filtered dynamically based on the active selection in the parent field.

| Sequence | Label | Value (Stored in DB) | Inactive | Dependent Value (`u_choice_6`) | Language |
| :---: | :--- | :--- | :---: | :--- | :---: |
| 0 | **Wi-Fi** | `wi-fi` | false | `network` | `en` |
| 1 | **Projector** | `projector` | false | `hardware` | `en` |
| 2 | **Forgot Password** | `forgot password` | false | `access` | `en` |
| 3 | **Slow Computer** | `slow computer` | false | `performance` | `en` |

---

## 6. Instance Configuration Evidence & Screenshot Verification

The system specifications documented above have been faithfully implemented and verified in the live ServiceNow instance. The figures below document the actual Dictionary Entry configuration records:

### 6.1 Category Dictionary Entry (`u_choice_6`)
- **Table:** `Incident WorkFlow [u_incident_workflow]`
- **Column label:** `Category`
- **Column name:** `u_choice_6`
- **Choices (4):** `network`, `performance`, `hardware`, `access`
- **Application:** `Global`

![Category Dictionary Entry](../screenshots/dictionary_entry_category.png)

### 6.2 Subcategory Dictionary Entry (`u_choice_7`)
- **Table:** `Incident WorkFlow [u_incident_workflow]`
- **Column label:** `Subcategory`
- **Column name:** `u_choice_7`
- **Choices (4):** `wi-fi` (dep: `network`), `projector` (dep: `hardware`), `forgot password` (dep: `access`), `slow computer` (dep: `performance`)
- **Attributes:** `edge_encryption_enabled=true`

![Subcategory Dictionary Entry](../screenshots/dictionary_entry_subcategory.png)

---

## 7. Form Layout and User Experience Design

The form design for `Incident WorkFlow` adopts a standard 2-column balanced layout optimized for desktop and mobile self-service:

```
+------------------------------------------------------------------------------------+
| Incident WorkFlow - [ INC00500 ]                                                  |
+------------------------------------------------------------------------------------+
| Number:            [ INC00500 (Read Only) ] | State:             [ New (1)       v ] |
| Caller:            [ Abel Tuter        (i) ] | Category:          [ Network       v ] |
| Short description: [ WiFi not working in lib ] | Subcategory:       [ Wi-Fi         v ] |
+------------------------------------------------------------------------------------+
| Assignment Group:  [ Network Support   (i) ] | Assigned To:       [ David Loo     (i) ] |
+------------------------------------------------------------------------------------+
| Description:                                                                       |
| [ The wireless signal drops continuously on the 2nd floor library reading desk.  ] |
+------------------------------------------------------------------------------------+
```

---

## 8. Summary of Approval

The requirements and schema specifications detailed in this document establish the formal technical baseline for **Phase 3: Project Design** and **Phase 5: Development**. Any modification to field definitions or dependent choices must undergo formal Change Control review.
