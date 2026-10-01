# Phase 7: Operational User Manual
## Project Title: Auto Ticket Classification using Flow Designer

---

### Executive Metadata
| Attribute | Specification |
| :--- | :--- |
| **Document Target** | Campus Students, Faculty, IT Helpdesk Agents, and System Administrators |
| **System Module** | `Incident WorkFlow` (`u_incident_workflow`) |
| **Document Version** | 1.0.0 |
| **Status** | Official Operational Guide |

---

## 1. Introduction

This manual provides comprehensive instructions for interacting with the **Automated Incident Classification System**. The manual is divided into two distinct sections:
- **Part I: End-User Guide** — For students, teachers, and staff submitting IT support requests.
- **Part II: Helpdesk & Administrator Guide** — For IT technicians, dispatchers, and system administrators managing tickets and maintaining workflow logic.

---

# Part I: End-User Guide (Students & Faculty)

## 2. Submitting an Incident Ticket

### 2.1 Accessing the Incident Submission Form
1. Log in to the institutional ServiceNow portal using your university credentials (SSO / NetID).
2. In the Application Navigator or Self-Service catalog, select **Incident WorkFlow** $\rightarrow$ **Create New** (or navigate directly to `u_incident_workflow.do`).
3. You will be presented with the simplified Incident WorkFlow submission form:

```
+─────────────────────────────────────────────────────────────────────────────+
| Incident WorkFlow - New Record                                             |
+─────────────────────────────────────────────────────────────────────────────+
| Number:            [ INC005xx (Auto-Generated) ]                            |
| Caller:            [ Your Name (Auto-Populated) ]                           |
| Short description: [                                                      * ]|
| Description:                                                                |
| [                                                                         ] |
+─────────────────────────────────────────────────────────────────────────────+
|                                [ Submit Ticket ]                            |
+─────────────────────────────────────────────────────────────────────────────+
```

---

### 2.2 Tips for Optimal Automatic Classification
Our automated classification engine inspects your **Short description** to route your ticket to the right specialist immediately. To ensure your ticket is handled as quickly as possible, use clear, specific keywords:

| Issue Type | Recommended Phrasing Examples | What the System Does Automatically |
| :--- | :--- | :--- |
| **Wi-Fi / Internet** | *"WiFi dropping in library"*, *"Cannot connect to campus wireless"*, *"No internet in dorm room"* | Classifies as **Network / Wi-Fi** and routes directly to the Network Infrastructure team. |
| **Classroom Projector** | *"Projector not turning on in Hall 101"*, *"HDMI display cable broken"*, *"Projector lamp flickering"* | Classifies as **Hardware / Projector** and alerts on-duty AV technicians. |
| **Password / Login** | *"Forgot password for student portal"*, *"Canvas account locked out"*, *"Need login reset"* | Classifies as **Access / Forgot Password** and routes to Identity Services. |
| **Computer Performance**| *"Slow computer in lab 4"*, *"Workstation freezing on login screen"*, *"Computer extremely laggy"* | Classifies as **Performance / Slow Computer** and assigns to Desktop Support. |

> **Best Practice Tip:** Keep the **Short description** focused on the symptom and location (e.g., *"WiFi not working in 3rd floor study lounge"*). Place detailed steps or background history in the larger **Description** box.

---

### 2.3 What Happens After You Click Submit?
1. **Instant Classification (< 1 Second):** The system automatically populates the **Category** and **Subcategory** without human delay.
2. **Email Confirmation:** Within seconds, an automated confirmation email arrives in your university inbox containing your unique **Ticket Number** (e.g., `INC00501`).
3. **Technician Assignment:** Your ticket immediately appears in the dedicated queue of the responsible technical team.

---

# Part II: Helpdesk & Administrator Guide

## 3. Helpdesk Agent Ticket Management

IT support agents interact with auto-classified incidents through the ServiceNow native workspace or standard list views.

### 3.1 Viewing the Incident Queue
1. In the Application Navigator, enter `u_incident_workflow.list` and press Enter.
2. The list view displays all records with their auto-assigned attributes:

| Number | Caller | Short Description | Category | Subcategory | State | Assignment Group |
| :---: | :--- | :--- | :---: | :---: | :---: | :--- |
| `INC00501` | Abel Tuter | WiFi not working in library | Network | Wi-Fi | New | Network Operations |
| `INC00502` | Beth Anglin | Projector not turning on | Hardware | Projector | New | AV Support |
| `INC00503` | Fred Luddy | Forgot my password for student portal | Access | Forgot Password | In progress | Identity & Access |
| `INC00504` | David Loo | Slow computer in computer lab | Performance | Slow Computer | New | Desktop Support |

---

### 3.2 Updating Ticket State and Resolving Issues
1. Open the target ticket record.
2. As work begins, change the **State** from `New` (`1`) to **`In Progress`** (`2`).
3. If waiting on user feedback or external vendor parts, set **State** to **`On Hold`** (`3`).
4. Once the technical resolution is confirmed:
   - Set **State** to **`Resolved`** (`6`).
   - Add resolution work notes detailing the corrective action taken.
5. After user acceptance or the standard inactivity window (5 business days), set **State** to **`Closed`** (`7`).

---

## 4. System Administrator Maintenance & Monitoring

System administrators are responsible for monitoring workflow health and extending the keyword dictionary.

### 4.1 Auditing Flow Executions (`sys_flow_context`)
If a ticket fails to categorize as expected or an email is delayed, inspect the execution context:
1. Navigate to **Process Automation** $\rightarrow$ **Flow Designer**.
2. Click on the **Executions** tab in the top navigation bar.
3. Locate the flow name: **`Auto Ticket Classifier`**.
4. Click on the execution instance timestamp corresponding to the ticket number.
5. Review the visual execution timeline:
   - Green checkmarks indicate successfully evaluated steps.
   - Expand the conditional branches to see which keyword condition evaluated to `true`.
   - Inspect the **Send Email** step to confirm delivery payload and recipient address.

---

### 4.2 Adding or Modifying Routing Keywords
To expand the keyword list as new campus technologies emerge:
1. In Flow Designer, open the **`Auto Ticket Classifier`** flow.
2. Click **Checkout** (if working in an active update set).
3. Select the target conditional branch (e.g., Condition 1: Network).
4. Click the **+ OR** button to append additional synonyms:
   - *Example:* Add `Trigger -> Short description contains "vpn"` or `"eduroam"`.
5. Click **Save** in the top-right header.
6. Click **Activate** to publish the changes into the live production runtime.

---

### 4.3 Managing Dependent Choice Values
To introduce a new subcategory:
1. Navigate to **System Definition** $\rightarrow$ **Choice Lists** (or right-click the Subcategory field on the form $\rightarrow$ **Configure Dictionary**).
2. Filter for Table: `u_incident_workflow` and Element: `u_choice_7`.
3. Click **New**.
4. Provide the **Label**, **Value**, and crucially, set the **Dependent value** matching the internal value of the desired parent Category (`u_choice_6`):
   - *Example:* Label: `Printer`, Value: `printer`, Dependent value: `hardware`.
5. Click **Submit**. The new subcategory will now dynamically appear only when `Hardware` is selected.

---

## 5. Frequently Asked Questions (FAQ)

**Q1: What happens if a user submits a ticket with no matching keywords?**  
*A:* The flow enters the default `Else` branch, setting Category to `Hardware` and Subcategory to empty. The ticket is immediately routed to the Tier-1 Helpdesk Triage queue for manual inspection.

**Q2: Can an IT agent manually override the automated Category and Subcategory?**  
*A:* Yes. Field agents have full read/write access to update `Category` and `Subcategory` if a ticket's true root cause diverges from the initial user description.

**Q3: Does the flow re-run if an agent edits the Short Description later?**  
*A:* No. The flow trigger is strictly configured for `Record Created`. Subsequent database updates do not trigger re-classification, preserving technician edits and preventing race conditions.
