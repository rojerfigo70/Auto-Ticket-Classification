# Phase 1: Brainstorming and Ideation Document
## Project Title: Auto Ticket Classification using Flow Designer

---

### Executive Metadata
| Attribute | Specification |
| :--- | :--- |
| **Project Title** | Auto Ticket Classification using Flow Designer |
| **Target Platform** | ServiceNow Cloud Platform (Utah / Vancouver / Washington DC / Xanadu) |
| **Application Scope** | Global Scope (`global`) |
| **Primary Domain** | IT Service Management (ITSM) / Campus IT Operations |
| **Target Audience** | Academic Institutions, Schools, Colleges, and Enterprise IT Helpdesks |
| **Document Version** | 1.0.0 |
| **Status** | Approved & Baselined |

---

## 1. Introduction and Context

In modern academic environments—encompassing universities, colleges, and secondary schools—technology infrastructure is integral to daily pedagogical and administrative functions. The institutional IT Helpdesk receives hundreds of support requests daily from students, faculty, researchers, and administrative personnel. 

Historically, these tickets arrive as unstructured free-text submissions via email, self-service portals, or phone transcriptions. Without an automated ingestion and categorization engine, all incoming requests land in a generic, unassigned queue requiring manual evaluation by Tier-1 IT dispatchers. This creates a severe operational bottleneck that delays incident resolution and degrades end-user satisfaction.

This project, **"Auto Ticket Classification using Flow Designer"**, conceives, architects, and implements an event-driven, no-code/low-code automated classification engine within ServiceNow. By leveraging the visual workflow capabilities of **ServiceNow Flow Designer**, the system instantly inspects ticket metadata and descriptions upon creation, classifies the incident into appropriate **Category** and **Subcategory** taxonomies, and dispatches automated confirmation notifications to the end user.

---

## 2. Problem Statement and Operational Pain Points

### 2.1 The Current Operational Bottleneck
At typical campus IT helpdesks, incoming tickets are manually triaged by human operators during business hours (8:00 AM – 5:00 PM). Tickets submitted outside of these hours accumulate in a dormant queue. Even during operating hours, the manual review process introduces significant delays.

```
[User Submits Ticket]
        │
        ▼
[Generic Unassigned Queue] ─── (Waits 30-180 Mins for Human Review)
        │
        ▼
[Dispatcher Reads Text] ──── (Human Subjectivity / Inconsistency)
        │
        ▼
[Manual Category / Subcategory Selection]
        │
        ▼
[Manual Assignment Group Routing]
```

### 2.2 Critical Pain Points Identified
1. **Prolonged Triage Latency (Mean Time to Triage - MTTT):**
   - High volumes during morning lecture hours cause tickets to languish in the unassigned state for 45 to 120 minutes before any action is taken.
2. **High Misclassification Rate & "Ticket Ping-Pong":**
   - Human dispatchers under pressure frequently miscategorize tickets (e.g., misidentifying an online portal authentication error as a network connectivity fault). 
   - Misrouted tickets circulate through 2 to 3 different support queues before reaching the correct technical resolver group, inflating **Mean Time to Resolution (MTTR)** by over 300%.
3. **Disruption of Time-Critical Academic Activities:**
   - In-classroom incidents (such as multimedia projector failures or campus Wi-Fi drops during exams) require immediate, high-priority routing. A delay of 30 minutes in assigning a projector issue to the field-support technician can ruin an entire lecture or presentation.
4. **Lack of Immediate Caller Feedback:**
   - Callers submit tickets into a "black hole" with no automated feedback confirming how their issue was understood or categorized, generating repeated duplicate tickets and phone inquiries.
5. **Technical Debt and Maintenance Overhead of Legacy Scripts:**
   - Previous attempts to automate classification relied on monolithic, hardcoded JavaScript Business Rules (`sys_script`). These rules became brittle, difficult to debug, prone to recursion issues, and inaccessible to non-programmer IT administrators.

---

## 3. High-Frequency Incident Profiles

An audit of campus IT tickets revealed that over 75% of all daily incidents fall into four recurring operational clusters:

| Cluster | Incident Domain | Typical User Phrasing / Symptoms | Operational Impact |
| :--- | :--- | :--- | :--- |
| **Cluster 1** | **Network / Wi-Fi** | "WiFi dropping in campus library", "Cannot connect to eduroam", "Wireless signal weak in dorms", "Internet unavailable" | Critical: Halts online exams, research, and remote coursework. |
| **Cluster 2** | **Hardware / Projector** | "Projector won't turn on in Room 302", "HDMI display not detecting laptop", "Projector bulb blinking red", "No audio from display" | High: Halts in-person lectures and audiovisual presentations. |
| **Cluster 3** | **Access / Forgot Password** | "Forgot password for student portal", "Locked out of canvas account", "SSO login failure", "Need MFA reset" | High: Prevents students from submitting assignments and accessing grades. |
| **Cluster 4** | **Performance / Slow Computer** | "Computer lab workstation extremely slow", "PC freezing on login", "System lagging when running MATLAB", "High disk usage freeze" | Moderate to High: Slows down lab experiments and examinations. |

---

## 4. Ideation and Solution Evaluation

During the brainstorming phase, the technical team evaluated four distinct architectural strategies to achieve automated classification:

### 4.1 Comparative Architectural Evaluation Matrix

| Criteria | Strategy 1: Manual Triaging (Status Quo) | Strategy 2: Legacy Business Rules (`sys_script`) | Strategy 3: ServiceNow Flow Designer (Selected Solution) | Strategy 4: ServiceNow Predictive Intelligence (ML) |
| :--- | :--- | :--- | :--- | :--- |
| **Categorization Latency** | 30 – 180 Minutes | < 1 Second | < 1 Second | < 1.5 Seconds |
| **Implementation Complexity** | Zero (Existing) | Medium (Scripted JavaScript) | Low (Visual No-Code Builder) | High (Requires ML model training) |
| **Training Data Requirement** | None | None | None | 10,000+ historical records required |
| **Maintenance Accessibility** | None | Requires JS Developer | Any Helpdesk Admin / Analyst | Requires Data/ML Specialist |
| **Auditability & Traceability**| Poor (Scattered activity logs) | Medium (`gs.log` / Script debugger) | Superior (Visual Flow Execution History) | Black-box probabilistic scoring |
| **Notification Integration** | Manual Email Dispatch | Scripted `gs.eventQueue()` + Email Notifications | Native drag-and-drop "Send Email" action | Requires coupled Flow or Notification rule |
| **License Requirement** | Core ITSM | Core ITSM | Core Platform / Flow Designer (Included) | Enterprise ITSM Pro License (Additional Cost) |

### 4.2 Why Flow Designer is the Optimal Architectural Choice
1. **Declarative & Low-Code:** ServiceNow Flow Designer enables administrators and business analysts to construct, inspect, and update conditional routing rules without writing server-side JavaScript.
2. **Robust Execution Context:** Flow Designer provides a dedicated, visual execution engine (`sys_flow_context`) that inspects runtime state, data pills, step-by-step evaluations, and execution durations in real time.
3. **Decoupled Architecture:** Business logic is decoupled from table definitions, eliminating database deadlocks and recursive triggers frequently encountered in synchronous `before`/`after` Business Rules.
4. **Native Extensibility:** Easily extensible to include Slack/Teams webhooks, automated SMS gateways, or ServiceNow IntegrationHub spokes in future iterations without refactoring core logic.

---

## 5. Scope Definition

### 5.1 In-Scope Deliverables
- **Custom Workflow Table:** Creation of `u_incident_workflow` (`Incident WorkFlow`) in the Global application scope.
- **Controlled Taxonomies:** Implementation of Category (`u_choice_6`) and Subcategory (`u_choice_7`) fields with strict parent-child dependent value bindings.
- **Auto-Numbering Schema:** Implementation of auto-generated ticket numbers with prefix `INC`, starting at `500`, formatted across 5 digits (e.g., `INC00500`, `INC00501`).
- **Flow Designer Logic:** An event-driven flow triggered upon record creation (`Created`) evaluating keywords across `short_description` and `description`.
- **Classification Engine:**
  - `Network` $\rightarrow$ `Wi-Fi` (Triggered by: `wifi`, `wi-fi`, `wireless`, `internet`)
  - `Hardware` $\rightarrow$ `Projector` (Triggered by: `projector`, `display`, `hdmi`)
  - `Access` $\rightarrow$ `Forgot Password` (Triggered by: `password`, `portal`, `login`, `account`)
  - `Performance` $\rightarrow$ `Slow Computer` (Triggered by: `slow`, `freeze`, `lag`, `performance`)
  - Fallback handling for unclassified or ambiguous submissions.
- **Automated Communication:** Integrated `Send Email` action notifying the ticket caller (`Caller`) with ticket confirmation, assigned category, and next steps.
- **Enterprise Update Set:** Packaging all 48 configuration changes into `Project Update Set` for seamless portability across development, test, and production instances.

### 5.2 Out-of-Scope Items
- Optical Character Recognition (OCR) of screenshot attachments.
- Automatic password resets via Active Directory / LDAP spokes (reserved for Phase 2 integration).
- Multilingual Natural Language Processing (NLP) for non-English inquiries.
- Physical asset dispatch and hardware replacement procurement workflows.

---

## 6. Business Value and Quantifiable ROI

```
[Traditional Manual Model]
Triage Latency: 45 - 90 Minutes  |  Misclassification: 35%  |  Manual Effort: ~15 hrs/week
                                    │
                                    ▼ (Flow Designer Automation)
[Automated Classification Engine]
Triage Latency: < 1.0 Second    |  Misclassification: < 2%  |  Manual Effort: 0 hrs/week
```

### 6.1 Key Performance Indicators (KPIs)
| KPI Metric | Baseline (Manual) | Projected Target (Flow Designer) | Expected Impact |
| :--- | :--- | :--- | :--- |
| **Mean Time to Triage (MTTT)** | 45 minutes | Instantaneous (< 1.5 seconds) | **99.9% reduction** |
| **Misclassification Rate** | 32% | < 3% (Exact keyword match) | **90% reduction in ticket bounce** |
| **Mean Time to Resolution (MTTR)** | 18.5 hours | 6.2 hours | **66% faster issue resolution** |
| **Helpdesk Tier-1 Time Saved** | 0% | 15–20 hours/week per operator | Staff redeployed to hands-on support |
| **First-Contact Caller Satisfaction (CSAT)**| 64% | > 92% | Callers receive immediate, transparent categorization |

---

## 7. Stakeholder Analysis

| Stakeholder Group | Role in Project | Key Needs & Expectations | Impact of Automation |
| :--- | :--- | :--- | :--- |
| **Students & Faculty (End Users)** | Consumers / Callers | Fast acknowledgment, clear visibility into issue routing, minimal classroom downtime. | Immediate confirmation email; zero delay before technician dispatch. |
| **Tier-1 Helpdesk Dispatchers** | Operational Operators | Relief from repetitive ticket reading and manual data entry; fewer escalations. | Eradicates manual triage queue; allows focus on complex support calls. |
| **Field Technicians / Resolvers** | Technical Resolvers | Accurately categorized tickets in their team queue without misrouted spam. | Receives tickets with clear category/subcategory taxonomy and complete context. |
| **IT Helpdesk Manager** | Operational Governance | Accurate reporting on incident frequencies across Wi-Fi, hardware, and account access. | High data integrity; pristine analytics dashboards without unclassified noise. |
| **ServiceNow System Administrator** | Platform Custodian | Low-maintenance solution that does not degrade instance performance or break during upgrades. | Declarative, upgrade-safe Flow Designer workflow packaged in an XML update set. |

---

## 8. Project Team Roles and Responsibilities (RACI Matrix)

| Role | Primary Responsibilities | Core Deliverables |
| :--- | :--- | :--- |
| **Solution Architect** | System design, architectural compliance, scoping, schema governance | Architecture Specification, Data Model, Risk Analysis |
| **ServiceNow Developer** | Table creation, dictionary configuration, Flow Designer building, Update Set management | `u_incident_workflow`, Flows, `Project_Update_Set.xml` |
| **QA / Test Engineer** | Test case design, unit testing, regression testing, email queue validation | Test Execution Matrix, Verification Logs (`sys_email`) |
| **Business Analyst / Tech Writer** | Requirements elicitation, user guide creation, demo walkthrough, final reporting | User Manual, Final Project Report, Demonstration Guidelines |

### RACI Matrix
| Activity / Phase | Solution Architect | ServiceNow Developer | QA Engineer | Business Analyst |
| :--- | :---: | :---: | :---: | :---: |
| **Brainstorming & Scoping** | **A** | C | C | **R** |
| **Requirement Specification** | **A** | C | C | **R** |
| **Table & Choice Configuration** | C | **R / A** | I | I |
| **Flow Designer Logic Construction** | C | **R / A** | I | I |
| **System & Integration Testing** | I | C | **R / A** | I |
| **Update Set Packaging & Export** | C | **R / A** | I | I |
| **User Manual & Final Report** | I | I | C | **R / A** |
| **Demo & Presentation Walkthrough** | C | **R** | I | **A** |

*Legend: R = Responsible, A = Accountable, C = Consulted, I = Informed*

---

## 9. Conclusion and Next Steps

The brainstorming and ideation phase conclusively validates that automating ticket classification through ServiceNow Flow Designer addresses the foundational pain points of campus IT service operations. By eliminating manual intervention for Wi-Fi, projector, account, and performance incidents, the helpdesk can achieve near-zero triage latency, drastically cut MTTR, and improve user satisfaction.

The project proceeds immediately into **Phase 2: Requirement Analysis**, where functional specifications, non-functional thresholds, and exact schema definitions for `u_incident_workflow` are codified.
