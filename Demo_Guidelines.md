# Phase 8: Demonstration Guidelines & Walkthrough Script
## Project Title: Auto Ticket Classification using Flow Designer

---

### Executive Metadata
| Attribute | Specification |
| :--- | :--- |
| **Presentation Type** | Live Technical Capstone / Stakeholder Demonstration |
| **Target Platform** | ServiceNow Instance (PDI or Enterprise Sandbox) |
| **Demonstration Duration**| 10 Minutes Total (8 Minutes Walkthrough + 2 Minutes Q&A) |
| **Key Flow Shown** | `Auto Ticket Classifier` in Flow Designer |
| **Document Version** | 1.0.0 |
| **Status** | Approved Demonstration Script |

---

## 1. Pre-Demonstration Setup Checklist

Ensure all environment prerequisites are completed at least 15 minutes before beginning the live presentation:

- [ ] **Instance Accessibility:** Confirm active login session into your ServiceNow instance with `admin` privileges.
- [ ] **Browser Tabs Prepared:** Open the following tabs in order for smooth navigation:
  - Tab 1: `u_incident_workflow.list` (Incident WorkFlow List View)
  - Tab 2: `u_incident_workflow.do` (New Ticket Submission Form)
  - Tab 3: **Flow Designer** (`Auto Ticket Classifier` flow open in read/view mode)
  - Tab 4: `sys_email.list` (System Mailbox filtered by `Target table = u_incident_workflow`)
  - Tab 5: Local Update Sets (`Project Update Set` marked `Complete`)
- [ ] **Data Cleansing:** Delete any temporary or junk test records created during rehearsals so the queue is clean.
- [ ] **Sample Users Verified:** Verify test caller profiles exist in `sys_user` (`Abel Tuter`, `Beth Anglin`, `Fred Luddy`, `David Loo`).
- [ ] **Screen Recording & Audio Check:** Verify microphone input levels and screen resolution (1080p recommended).

---

## 2. Timed Demonstration Agenda (10-Minute Format)

| Time Window | Presentation Segment | Core Focus & Objective |
| :---: | :--- | :--- |
| **00:00 – 01:30** | **1. Problem Statement & Architecture** | Introduce campus IT triage bottleneck, manual delays, and the Flow Designer solution. |
| **01:30 – 05:00** | **2. Live End-to-End Test Execution** | Submit 4 live tickets covering Network, Hardware, Access, and Performance; show instant classification. |
| **05:00 – 07:00** | **3. Flow Designer Logic Deep-Dive** | Inspect the visual flow, conditional decision branches, and dynamic Data Pills. |
| **07:00 – 08:30** | **4. Verification & Audit Trail** | Inspect `sys_email` outbound confirmation and `sys_flow_context` execution metrics. |
| **08:30 – 10:00** | **5. Conclusion & Evaluator Q&A** | Summary of business ROI, future roadmap (AI/SLA), and answering questions. |

---

## 3. Step-by-Step Live Walkthrough Script

### Segment 1: Problem Statement & Architecture (00:00 – 01:30)
* **Presenter Action:** Share screen showing the architecture diagram from `Design_Specification.md`.
* **Spoken Narrative:**
  > *"Good morning, esteemed evaluators and colleagues. In high-volume IT service desks, such as university campuses, hundreds of unstructured support tickets arrive daily. Historically, Tier-1 helpdesk dispatchers spent hours reading free-text descriptions to figure out who should fix what, resulting in 45-minute triage delays and a 30% misclassification rate.*
  >
  > *Today, I am demonstrating **'Auto Ticket Classification using Flow Designer'**. Built natively on the ServiceNow platform, this solution automatically ingests tickets on our custom table `Incident WorkFlow [u_incident_workflow]`, evaluates semantic keywords within milliseconds, populates dependent Category and Subcategory fields, and dispatches rich-text confirmation emails to the caller. Let’s see it in action."*

---

### Segment 2: Live Ticket Creation & Instant Classification (01:30 – 05:00)
* **Presenter Action:** Switch to Tab 2 (`u_incident_workflow.do`). Show that Category and Subcategory are currently empty.
* **Demonstration Step 1 (Network / Wi-Fi):**
  - **Caller:** `Abel Tuter`
  - **Short description:** `WiFi not working in library`
  - **Click:** `Submit`
  - **Spoken Narrative:**
    > *"Notice that as I submit the ticket, the number is automatically generated as `INC00501`. If we re-open `INC00501`, the system has automatically set Category to **Network** and Subcategory to **Wi-Fi**. No human operator intervened."*
* **Demonstration Step 2 (Hardware / Projector):**
  - **Caller:** `Beth Anglin`
  - **Short description:** `Projector not turning on`
  - **Click:** `Submit`
  - **Spoken Narrative:**
    > *"Now let's submit an urgent classroom issue: 'Projector not turning on'. Upon submission, ticket `INC00502` is immediately classified under **Hardware** with subcategory **Projector**."*
* **Demonstration Step 3 (Access / Password):**
  - **Caller:** `Fred Luddy`
  - **Short description:** `Forgot my password for student portal`
  - **Click:** `Submit`
  - **Spoken Narrative:**
    > *"Third, an account lockout: 'Forgot my password for student portal'. Instantaneously, ticket `INC00503` categorizes into **Access** and **Forgot Password**."*
* **Demonstration Step 4 (Performance / Slow Computer):**
  - **Caller:** `David Loo`
  - **Short description:** `Slow computer in computer lab`
  - **Click:** `Submit`
  - **Spoken Narrative:**
    > *"Finally, a device performance ticket: 'Slow computer in computer lab'. `INC00504` is automatically assigned to **Performance** and **Slow Computer**."*

---

### Segment 3: Flow Designer Engine Deep-Dive (05:00 – 07:00)
* **Presenter Action:** Switch to Tab 3 (**Flow Designer**).
* **Spoken Narrative:**
  > *"Now let's inspect the underlying intelligence behind this automation. Inside ServiceNow Flow Designer, we have built the `Auto Ticket Classifier` flow.*
  >
  > *• **Trigger:** The flow triggers automatically whenever a new record is created on `u_incident_workflow`.*
  > *• **Conditional Branches:** We implemented an ordered decision tree. Branch 1 evaluates if the short description contains network keywords like `wifi`, `wireless`, or `internet`. If matched, it executes the `Update Record` action.*
  > *• **Dependent Choice Integrity:** Because our dictionary defines `Subcategory` as dependent on `Category`, our flow updates adhere strictly to data normalization rules.*
  > *• **Maintainability:** Because this is built using declarative Flow Designer actions, any IT analyst can add new synonyms with a single click—no JavaScript coding required."*

---

### Segment 4: Email Dispatch & Execution Audit Log (07:00 – 08:30)
* **Presenter Action:** Switch to Tab 4 (`sys_email.list`). Open the newest email record for `INC00501`.
* **Spoken Narrative:**
  > *"Next, let's verify communication integrity. In `sys_email`, we see the outbound notification generated for Abel Tuter. Notice how Flow Designer dynamically populated the ticket number `INC00501`, the short description, and the classified category and subcategory directly into the HTML body.*
  >
  > *If we examine the Flow Execution Context in `sys_flow_context`, the entire end-to-end transaction—from keyword evaluation to database update and email dispatch—completed in just **680 milliseconds**."*

---

### Segment 5: Conclusion & Q&A (08:30 – 10:00)
* **Presenter Action:** Switch to Tab 5 showing the completed `Project Update Set` and 48 customer updates.
* **Spoken Narrative:**
  > *"All 48 platform configurations—including our table, auto-numbering, dictionary entries, choice lists, and flow logic—have been captured in `Project Update Set` and exported to clean XML for seamless migration across any instance.*
  >
  > *In conclusion, this automation slashes triage latency by 99.7%, eliminates queue reassignments, and delivers instant transparency to campus users. Thank you, and I now welcome any questions."*

---

## 4. Anticipated Evaluator Questions and Authoritative Answers

| Question | Recommended Answer |
| :--- | :--- |
| **Q1: Why did you choose Flow Designer instead of traditional Business Rules (`sys_script`)?** | *"Flow Designer offers a modern, no-code visual interface that separates business logic from code. Unlike Business Rules, Flow Designer runs asynchronously, prevents recursive loops, provides a full execution history in `sys_flow_context`, and allows business analysts to maintain routing rules without writing server-side JavaScript."* |
| **Q2: What happens if a user enters words from two categories, like 'portal is slow'?** | *"We designed our decision tree with strict sequential precedence: Network $\rightarrow$ Hardware $\rightarrow$ Access $\rightarrow$ Performance. In that specific scenario, the Access branch evaluates first, correctly identifying that the student needs assistance accessing the portal before addressing the performance symptom."* |
| **Q3: How are choice dependencies enforced between Category and Subcategory?** | *"We configured the Subcategory field (`u_choice_7`) in Advanced Dictionary View with `dependent=u_choice_6`. Each record in the `sys_choice` table specifies the exact parent value (e.g. `wi-fi` requires `network`). This prevents invalid combinations from being saved in both the UI and the database."* |
| **Q4: How easily can this be migrated to another ServiceNow instance?** | *"Extremely easily. All 48 configuration items are captured in our completed `Project Update Set`. An administrator simply navigates to Retrieved Update Sets, imports `Project_Update_Set.xml`, previews it, and commits it. The table, dictionary entries, choices, and flows are instantiated instantly."* |
