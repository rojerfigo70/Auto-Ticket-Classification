# Auto Ticket Classification using Flow Designer
### ServiceNow IT Service Management (ITSM) Automation Repository

[![ServiceNow](https://img.shields.io/badge/Platform-ServiceNow-81B5A1?style=for-the-badge&logo=servicenow&logoColor=white)](https://www.servicenow.com/)
[![Application Scope](https://img.shields.io/badge/Scope-Global-032D60?style=for-the-badge)](https://developer.servicenow.com/)
[![Flow Designer](https://img.shields.io/badge/Automation-Flow%20Designer-293E40?style=for-the-badge)](https://docs.servicenow.com/)
[![Update Set](https://img.shields.io/badge/Update%20Set-Complete%20(48%20Updates)-success?style=for-the-badge)](#setup-and-deployment-guide)
[![Academic Status](https://img.shields.io/badge/Submission-Phase%201--8%20Complete-brightgreen?style=for-the-badge)](#project-repository-structure)

---

## Executive Abstract

In high-volume IT support environments—such as educational institutions, universities, and enterprise helpdesks—manual ticket triaging represents an expensive, error-prone operational bottleneck. Dispatchers spend hours manually reading free-text descriptions to route tickets to appropriate resolver teams, resulting in 45+ minute triage delays, 30% misclassification rates, and ticket ping-pong.

**Auto Ticket Classification using Flow Designer** is a complete, enterprise-grade IT Service Management automation system built natively on the **ServiceNow** cloud platform. Designed around the custom workflow table **`Incident WorkFlow [u_incident_workflow]`**, the system intercepts newly created incident tickets, extracts semantic intent from the caller's short description via an ordered keyword decision tree, instantly sets normalized **Category** and **Subcategory** choice fields, and triggers automated rich-text confirmation emails to the caller.

All 48 platform configuration artifacts (table definitions, auto-numbering, dictionary entries, dependent choice lists, and Flow Designer actions) are fully packaged into a deployable ServiceNow Update Set (**`Project_Update_Set.xml`**).

---

## Key Features & Highlights

- **Near-Zero Triage Latency (< 800 ms):** Completely eliminates human dispatcher wait times by evaluating keywords and classifying records at the database creation event.
- **Strict Data Normalization:** Enforces dependent choice list integrity between parent Category (`u_choice_6`) and child Subcategory (`u_choice_7`).
- **Zero-Code / Low-Code Maintainability:** 100% constructed using ServiceNow Flow Designer visual logic; no fragile JavaScript Business Rules or technical debt.
- **Dynamic Email Notifications:** Generates real-time, personalized HTML confirmation emails dispatched directly to the caller via ServiceNow's native `sys_email` engine.
- **Deterministic Keyword Routing:** Structured decision tree handling four critical campus incident domains (Wi-Fi/Network, Projectors/Hardware, Passwords/Access, and Workstation Performance) with graceful fallback.
- **Turnkey Update Set:** Self-contained, exportable, and importable via `Project_Update_Set.xml` for instant migration between ServiceNow environments.

---

## High-Level System Architecture

The following diagram illustrates the end-to-end data and execution flow across the ServiceNow architecture:

```mermaid
flowchart TD
    subgraph UI ["Presentation Layer"]
        A[Caller / Student / Faculty] -->|Submits Ticket| B[Incident WorkFlow Form / Portal]
    end

    subgraph DB ["Data Persistence Layer (MariaDB)"]
        B -->|INSERT Record| C[(u_incident_workflow Table<br>Auto-Number: INC005xx<br>State: New)]
    end

    subgraph FD ["Automation Layer (Flow Designer Engine)"]
        C -->|Trigger: Record Created| D{Keyword Decision Engine}
        
        D -->|'wifi', 'wireless', 'internet'| E1[Set Category: Network<br>Set Subcategory: Wi-Fi]
        D -->|'projector', 'display', 'hdmi'| E2[Set Category: Hardware<br>Set Subcategory: Projector]
        D -->|'password', 'portal', 'login'| E3[Set Category: Access<br>Set Subcategory: Forgot Password]
        D -->|'slow', 'freeze', 'lag'| E4[Set Category: Performance<br>Set Subcategory: Slow Computer]
        D -->|No Match (Fallback)| E5[Set Category: Hardware<br>Set Subcategory: -- None --]
        
        E1 --> F[Action: Update Record<br>Commit Choice Values to DB]
        E2 --> F
        E3 --> F
        E4 --> F
        E5 --> F
    end

    subgraph Notify ["Notification Subsystem"]
        F --> G[Action: Send Email<br>Generate MIME Record]
        G --> H[(sys_email Outbox)]
        H -->|SMTP Dispatch| I[Caller University Inbox]
    end

    classDef primary fill:#032d60,stroke:#fff,stroke-width:2px,color:#fff;
    classDef action fill:#2e7d32,stroke:#fff,stroke-width:2px,color:#fff;
    classDef decision fill:#e65100,stroke:#fff,stroke-width:2px,color:#fff;
    class A,B,C,H,I primary;
    class F,G action;
    class D decision;
```

---

## Project Repository Structure (8-Phase Architecture)

This repository strictly adheres to the standard 8-phase academic and enterprise delivery lifecycle:

| Phase Folder | Primary Artifacts | Description & Deliverables |
| :--- | :--- | :--- |
| [📁 `1_Brainstorming_and_Ideation_Phase/`](./1_Brainstorming_and_Ideation_Phase/) | [`Brainstorming_Document.md`](./1_Brainstorming_and_Ideation_Phase/Brainstorming_Document.md) | Problem statement, comparative strategy analysis (Flow Designer vs. Business Rules vs. AI), business ROI, stakeholder analysis, and RACI matrix. |
| [📁 `2_Requirement_Analysis_Phase/`](./2_Requirement_Analysis_Phase/) | [`Requirement_Analysis.md`](./2_Requirement_Analysis_Phase/Requirement_Analysis.md) | Functional requirements (FR-01 to FR-08), non-functional constraints (NFR-01 to NFR-06), and complete schema specification for `u_incident_workflow`. |
| [📁 `3_Project_Design_Phase/`](./3_Project_Design_Phase/) | [`Design_Specification.md`](./3_Project_Design_Phase/Design_Specification.md) | Multi-tier architecture blueprint, complete Mermaid process flowchart, UML sequence diagram, and deterministic keyword routing matrix. |
| [📁 `4_Project_Planning_Phase/`](./4_Project_Planning_Phase/) | [`Project_Plan.md`](./4_Project_Planning_Phase/Project_Plan.md) | 6-Milestone Work Breakdown Structure (WBS), Mermaid Gantt chart schedule, resource allocation, and risk management matrix. |
| [📁 `5_Project_Development_Phase/`](./5_Project_Development_Phase/) | [`Development_Guide.md`](./5_Project_Development_Phase/Development_Guide.md)<br>[`Project_Update_Set.xml`](./5_Project_Development_Phase/update_set/Project_Update_Set.xml) | Step-by-step configuration manual, form design layout, dictionary entry setup, flow action logic, and the complete exported ServiceNow Update Set XML. |
| [📁 `6_Project_Testing_Phase/`](./6_Project_Testing_Phase/) | [`Test_Cases_and_Results.md`](./6_Project_Testing_Phase/Test_Cases_and_Results.md) | Execution results for 6 test scenarios (Wi-Fi, Projector, Password, Slow Computer, Fallback, Edge-Case), email verification logs, and `sys_flow_context` audit. |
| [📁 `7_Project_Documentation_Phase/`](./7_Project_Documentation_Phase/) | [`Final_Project_Report.md`](./7_Project_Documentation_Phase/Final_Project_Report.md)<br>[`User_Manual.md`](./7_Project_Documentation_Phase/User_Manual.md) | Executive project report, technical challenges & solutions, quantitative metrics, future scalability roadmap (AI/SLA), and comprehensive user manual. |
| [📁 `8_Project_Demonstration_Phase/`](./8_Project_Demonstration_Phase/) | [`Demo_Guidelines.md`](./8_Project_Demonstration_Phase/Demo_Guidelines.md)<br>[`Demo_Video_Link.txt`](./8_Project_Demonstration_Phase/Demo_Video_Link.txt) | Timed 10-minute presentation agenda, step-by-step live walkthrough script, evaluator Q&A defense answers, and demo video metadata template. |

---

## Schema and Controlled Taxonomies

### Core Table: `u_incident_workflow` (`Incident WorkFlow`)
- **Auto-Numbering:** `INC` + 5 digits starting at `500` (e.g. `INC00500`, `INC00501`)
- **Caller Field:** Reference $\rightarrow$ `sys_user`
- **Short description:** String (255, Mandatory)
- **Description:** String (4000)
- **State:** Integer Choice (`1`=New, `2`=In progress, `3`=On hold, `6`=Resolved, `7`=Closed)

### Category & Dependent Subcategory Choice Structure

| Category Label | Category DB Value (`u_choice_6`) | Subcategory Label | Subcategory DB Value (`u_choice_7`) | Dependent Parent Value | Matching User Keywords |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Network** | `network` | **Wi-Fi** | `wi-fi` | `network` | `wifi`, `wi-fi`, `wireless`, `internet` |
| **Hardware** | `hardware` | **Projector** | `projector` | `hardware` | `projector`, `display`, `hdmi` |
| **Access** | `access` | **Forgot Password** | `forgot password` | `access` | `password`, `portal`, `login`, `account` |
| **Performance** | `performance` | **Slow Computer** | `slow computer` | `performance` | `slow`, `freeze`, `lag`, `performance` |

---

## Instance Implementation Verification

The system is configured and tested in a live ServiceNow instance. Below are visual evidence captures of key configuration records:

### 1. Category Dictionary Entry (`u_choice_6`)
*Shows the 4 configured categories (`network`, `hardware`, `access`, `performance`) on `u_incident_workflow`:*
![Category Dictionary Configuration](./screenshots/dictionary_entry_category.png)

### 2. Subcategory Dictionary Entry (`u_choice_7`)
*Shows the 4 dependent choices with `dependent=u_choice_6`:*
![Subcategory Dictionary Configuration](./screenshots/dictionary_entry_subcategory.png)

### 3. Completed Update Set (`Project Update Set`)
*Shows the completed state with 48 customer updates captured:*
![Update Set Completed](./screenshots/update_set_completed.png)

---

## Setup and Deployment Guide

Follow these steps to deploy this solution into any ServiceNow instance (Personal Developer Instance or Sub-Production environment):

### Step 1: Import the Update Set XML
1. Log in to your ServiceNow instance as an administrator (`admin`).
2. In the Application Navigator, search for **System Update Sets** $\rightarrow$ **Retrieved Update Sets**.
3. Under the Related Links at the bottom of the list, click **Import Update Set from XML**.
4. Click **Choose File** and select:
   ```
   Auto_Ticket_Classification_ServiceNow/5_Project_Development_Phase/update_set/Project_Update_Set.xml
   ```
5. Click **Upload**.

### Step 2: Preview and Commit the Update Set
1. Open the imported record named **`Project Update Set`** (State will show as `Loaded`).
2. Click **Preview Update Set** in the header.
3. Verify that the preview finishes with **0 Errors / 0 Collisions**. (If any collisions occur, click into the update and choose *Accept Remote Update*).
4. Click **Commit Update Set**.
5. Once committed, the custom table `u_incident_workflow`, auto-numbering, dictionary entries, choice lists, and form views are instantiated.

### Step 3: Verify and Activate Flow Designer
1. Navigate to **Process Automation** $\rightarrow$ **Flow Designer**.
2. Open the flow named **`Auto Ticket Classifier`**.
3. Review the trigger (Table: `u_incident_workflow`, Created) and the conditional branches.
4. If not already active, click **Activate** in the top right corner.

### Step 4: Run Smoke Test
1. In the navigation bar, type `u_incident_workflow.do` and hit Enter.
2. Select any active caller (e.g. `Abel Tuter`).
3. Enter `WiFi not working in library` in the **Short description**.
4. Leave Category and Subcategory blank and click **Submit**.
5. Re-open the record and verify that Category is automatically populated as **Network** and Subcategory as **Wi-Fi**.
6. Check `sys_email.list` to confirm the outbound confirmation email was generated.

---

## Project Metadata & Credits

- **Project Title:** Auto Ticket Classification using Flow Designer
- **Platform Version Tested:** ServiceNow Washington DC / Xanadu / Utah
- **Target Audience:** IT Service Desk Managers, Enterprise IT Architects, Academic Capstone Evaluators
- **Repository Location:** `C:\Auto_Ticket_Classification_ServiceNow\`
- **Update Set Artifact:** [`5_Project_Development_Phase/update_set/Project_Update_Set.xml`](./5_Project_Development_Phase/update_set/Project_Update_Set.xml)
- **Status:** Completed, Verified, and Ready for Submission
"# Auto_Ticket_Classification_ServiceNow" 
