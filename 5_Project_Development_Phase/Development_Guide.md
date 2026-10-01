# Phase 5: Technical Development Guide
## Project Title: Auto Ticket Classification using Flow Designer

---

### Executive Metadata
| Attribute | Specification |
| :--- | :--- |
| **Development Instance**| ServiceNow Developer Instance (PDI) / Sub-Production Sandbox |
| **Application Scope** | `Global` (`sys_id: global`) |
| **Tracked Update Set** | `Project Update Set` |
| **Primary Artifacts** | `u_incident_workflow` Table, Dictionary Entries, Choice Lists, Flow Designer Workflow |
| **Document Version** | 1.0.0 |
| **Status** | Production-Ready Implementation Guide |

---

## 1. Overview of Development Lifecycle

This technical guide provides step-by-step instructions for reproducing, configuring, and verifying the **Auto Ticket Classification** solution within any ServiceNow instance. All platform modifications are tracked inside a dedicated Global Update Set named **`Project Update Set`**, ensuring full portability and zero manual script dependency.

```
Step 1: Initialize Update Set (`Project Update Set`)
   │
   ▼
Step 2: Create Table (`Incident WorkFlow` - `u_incident_workflow`) & Auto-Numbering
   │
   ▼
Step 3: Configure Form Layout & Mandatory Attributes
   │
   ▼
Step 4: Configure Data Dictionary & Dependent Choice Lists
   │
   ▼
Step 5: Build & Activate Flow Designer Logic (`Auto Ticket Classifier`)
   │
   ▼
Step 6: Mark Update Set 'Complete' & Export XML
```

---

## 2. Step 1: Update Set Initialization

Before performing any table or dictionary configurations, a dedicated Update Set must be initialized to capture all schema definitions and customer updates.

1. Navigate to **System Update Sets** $\rightarrow$ **Local Update Sets**.
2. Click **New** to create an update set record.
3. Configure the following values:
   - **Name:** `Project Update Set`
   - **Application:** `Global`
   - **State:** `In progress`
   - **Description:** `Captures table schema, dictionary entries, choice lists, form layout, and flow configurations for Auto Ticket Classification.`
4. Click **Submit and Make Current**.
5. Verify the Update Set picker in the ServiceNow header displays **`Project Update Set`** in the **Global** scope.

*(Note: Upon completing all steps, this update set is closed with state `Complete` as shown in the verification screenshot below).*

![Completed Project Update Set](../screenshots/update_set_completed.png)

---

## 3. Step 2: Custom Table Creation & Auto-Numbering

The custom table isolates the automated incident triage workflow while adopting standard ITSM conventions.

### 3.1 Creating the Table
1. Navigate to **System Definition** $\rightarrow$ **Tables**.
2. Click **New**.
3. Configure table attributes:
   - **Label:** `Incident WorkFlow`
   - **Name:** `u_incident_workflow` (auto-populated by system)
   - **Extends Table:** `-- None --`
   - **Application:** `Global`
4. In the **Controls** tab:
   - Check **Auto-number**.
   - **Prefix:** `INC`
   - **Starting number:** `500`
   - **Number of digits:** `5` (produces sequences: `INC00500`, `INC00501`, `INC00502`, ...)
   - Check **Create access controls (ACL)**.
   - User role: `snc_internal` / IT Helpdesk user role.
5. Click **Submit**.

---

## 4. Step 3: Form Layout and Form Designer Configuration

1. In the Application Navigator, type `u_incident_workflow.list` and press Enter.
2. Click **New** to open an empty record form.
3. Right-click the form header $\rightarrow$ **Configure** $\rightarrow$ **Form Design** (or **Form Layout**).
4. Arrange the fields in a balanced 2-column layout:

| Left Column | Right Column | Full Width (Section Below) |
| :--- | :--- | :--- |
| `u_number` (Number - Read Only) | `u_state` (State) | `u_description` (Description - Multiline) |
| `u_caller` (Caller - sys_user) | `u_choice_6` (Category) | |
| `u_short_description` (Short description - Mandatory) | `u_choice_7` (Subcategory) | |
| `u_assignment_group` (Assigned group) | `u_assigned_to` (Assigned to) | |

5. Click **Save** to commit the form view layout.

---

## 5. Step 4: Dictionary Entry Configuration for Choices & Dependencies

The heart of the taxonomic structure resides in the **Category** and **Subcategory** choice fields.

### 5.1 Category Field (`u_choice_6`)
1. On the `Incident WorkFlow` form, right-click the **Category** label $\rightarrow$ **Configure Dictionary**.
2. Ensure the following attributes are configured:
   - **Table:** `Incident WorkFlow [u_incident_workflow]`
   - **Type:** `String`
   - **Column label:** `Category`
   - **Column name:** `u_choice_6`
   - **Max length:** `40`
   - **Application:** `Global`
   - **Active:** Checked
   - **Choice:** `Dropdown with -- None --`
3. Under the **Choices** related list, click **New** to insert each of the 4 primary operational categories:

| Sequence | Label | Value | Inactive |
| :---: | :--- | :--- | :---: |
| 0 | **Network** | `network` | false |
| 1 | **Hardware** | `hardware` | false |
| 2 | **Access** | `access` | false |
| 3 | **Performance** | `performance` | false |

4. Click **Update** to save the Dictionary Entry.

*Instance Verification:*
![Category Dictionary Configuration](../screenshots/dictionary_entry_category.png)

---

### 5.2 Subcategory Field (`u_choice_7`) with Dependent Values
1. On the `Incident WorkFlow` form, right-click the **Subcategory** label $\rightarrow$ **Configure Dictionary**.
2. Switch to **Advanced view** via the Related Links.
3. Configure the following attributes:
   - **Table:** `Incident WorkFlow [u_incident_workflow]`
   - **Type:** `String`
   - **Column label:** `Subcategory`
   - **Column name:** `u_choice_7`
   - **Max length:** `40`
   - **Choice:** `Dropdown with -- None --`
   - **Choice table:** `-- None --`
   - **Attributes:** `edge_encryption_enabled=true`
4. Switch to the **Dependent Field** tab:
   - Check **Use dependent field**.
   - **Dependent field:** `Category` (`u_choice_6`).
5. In the **Choices** related list, configure each subcategory choice with its corresponding parent **Dependent value**:

| Sequence | Label | Value | Dependent value (`u_choice_6`) | Language | Inactive |
| :---: | :--- | :--- | :--- | :---: | :---: |
| 0 | **Wi-Fi** | `wi-fi` | `network` | `en` | false |
| 1 | **Projector** | `projector` | `hardware` | `en` | false |
| 2 | **Forgot Password** | `forgot password` | `access` | `en` | false |
| 3 | **Slow Computer** | `slow computer` | `performance` | `en` | false |

6. Click **Update** to persist changes.

*Instance Verification:*
![Subcategory Dictionary Configuration](../screenshots/dictionary_entry_subcategory.png)

---

## 6. Step 5: Flow Designer Automation Pipeline Construction

Flow Designer automates the evaluation, assignment, and email dispatch upon record creation.

### 6.1 Creating the Flow
1. Navigate to **Process Automation** $\rightarrow$ **Flow Designer**.
2. Click **Create Flow**.
3. Set properties:
   - **Flow name:** `Auto Ticket Classifier`
   - **Application:** `Global`
   - **Run As:** `System User` (ensures consistent execution privileges regardless of submitting caller permissions)
4. Click **Submit**.

---

### 6.2 Trigger Configuration
- **Trigger Type:** `Record` $\rightarrow$ `Created`
- **Table:** `Incident WorkFlow [u_incident_workflow]`
- **Condition:** `-- None --` (Evaluates every incoming ticket upon insert)

---

### 6.3 Flow Actions & Conditional Routing Rules

The flow executes conditional decision branches evaluating the contents of `short_description` and `description`:

#### Action 1: Evaluate Network Branch
- **Flow Logic:** `If`
- **Condition Label:** `If text relates to Wi-Fi / Network`
- **Condition Builder:**
  - `Trigger -> Incident Record -> Short description` *contains* `wifi` **OR**
  - `Trigger -> Incident Record -> Short description` *contains* `wi-fi` **OR**
  - `Trigger -> Incident Record -> Short description` *contains* `wireless` **OR**
  - `Trigger -> Incident Record -> Short description` *contains* `internet`
- **Sub-Action (Within If):** `Update Record`
  - **Record:** `Trigger -> Incident Record`
  - **Fields to Update:**
    - `Category` (`u_choice_6`) = `network`
    - `Subcategory` (`u_choice_7`) = `wi-fi`

---

#### Action 2: Evaluate Hardware Branch
- **Flow Logic:** `Else If`
- **Condition Label:** `If text relates to Projector / Hardware`
- **Condition Builder:**
  - `Trigger -> Incident Record -> Short description` *contains* `projector` **OR**
  - `Trigger -> Incident Record -> Short description` *contains* `display` **OR**
  - `Trigger -> Incident Record -> Short description` *contains* `hdmi`
- **Sub-Action (Within Else If):** `Update Record`
  - **Record:** `Trigger -> Incident Record`
  - **Fields to Update:**
    - `Category` (`u_choice_6`) = `hardware`
    - `Subcategory` (`u_choice_7`) = `projector`

---

#### Action 3: Evaluate Access Branch
- **Flow Logic:** `Else If`
- **Condition Label:** `If text relates to Password / Account Access`
- **Condition Builder:**
  - `Trigger -> Incident Record -> Short description` *contains* `password` **OR**
  - `Trigger -> Incident Record -> Short description` *contains* `portal` **OR**
  - `Trigger -> Incident Record -> Short description` *contains* `login` **OR**
  - `Trigger -> Incident Record -> Short description` *contains* `account`
- **Sub-Action (Within Else If):** `Update Record`
  - **Record:** `Trigger -> Incident Record`
  - **Fields to Update:**
    - `Category` (`u_choice_6`) = `access`
    - `Subcategory` (`u_choice_7`) = `forgot password`

---

#### Action 4: Evaluate Performance Branch
- **Flow Logic:** `Else If`
- **Condition Label:** `If text relates to Slow Computer / Performance`
- **Condition Builder:**
  - `Trigger -> Incident Record -> Short description` *contains* `slow` **OR**
  - `Trigger -> Incident Record -> Short description` *contains* `freeze` **OR**
  - `Trigger -> Incident Record -> Short description` *contains* `lag` **OR**
  - `Trigger -> Incident Record -> Short description` *contains* `performance`
- **Sub-Action (Within Else If):** `Update Record`
  - **Record:** `Trigger -> Incident Record`
  - **Fields to Update:**
    - `Category` (`u_choice_6`) = `performance`
    - `Subcategory` (`u_choice_7`) = `slow computer`

---

#### Action 5: Default Fallback Branch
- **Flow Logic:** `Else`
- **Sub-Action (Within Else):** `Update Record`
  - **Record:** `Trigger -> Incident Record`
  - **Fields to Update:**
    - `Category` (`u_choice_6`) = `hardware`
    - `Subcategory` (`u_choice_7`) = `-- None --`

---

### 6.4 Action 6: Send Outbound Notification Email
Immediately after the conditional branches complete:
- **Action:** `ServiceNow Core` $\rightarrow$ `Send Email`
- **Target Record:** `Trigger -> Incident Record`
- **To:** Data Pill $\rightarrow$ `Trigger -> Incident Record -> Caller`
- **Subject:** `Ticket Created - [Number]: [Short description]`
- **Body (HTML):**
```html
<p>Hello <strong>{{Trigger.u_caller.name}}</strong>,</p>
<p>Your support ticket has been received and automatically categorized:</p>
<ul>
  <li><strong>Ticket Number:</strong> {{Trigger.u_number}}</li>
  <li><strong>Summary:</strong> {{Trigger.u_short_description}}</li>
  <li><strong>Category:</strong> {{Trigger.u_choice_6}}</li>
  <li><strong>Subcategory:</strong> {{Trigger.u_choice_7}}</li>
  <li><strong>Current State:</strong> New</li>
</ul>
<p>Our technical support team has been assigned to your ticket. Thank you for contacting Campus IT Support.</p>
```
- Click **Save** in Flow Designer.
- Click **Activate** to make the flow live in the instance runtime.

---

## 7. Step 6: Completing and Exporting the Update Set

Once all configurations and tests pass:
1. Navigate to **System Update Sets** $\rightarrow$ **Local Update Sets**.
2. Open **`Project Update Set`**.
3. Verify that **Customer Updates (48)** records are captured (including table definitions, fields, choices, and views).
4. Change the **State** field from `In progress` to **`Complete`**.
5. Click **Update** to save the record.
6. Under **Related Links**, click **Export to XML**.
7. The browser downloads **`Project_Update_Set.xml`**.
8. Save this file into the repository subfolder `5_Project_Development_Phase/update_set/Project_Update_Set.xml`.

*Instance Verification:*
![Update Set Completed with 48 Updates](../screenshots/update_set_completed.png)

---

## 8. Summary of Development Deliverables

| Artifact | Location in Repository | Verification Method |
| :--- | :--- | :--- |
| **Development Guide** | `5_Project_Development_Phase/Development_Guide.md` | Markdown Specification |
| **Exported Update Set XML** | `5_Project_Development_Phase/update_set/Project_Update_Set.xml` | XML Schema Validation |
| **Category Dictionary Screenshot** | `screenshots/dictionary_entry_category.png` | Visual Evidence |
| **Subcategory Dictionary Screenshot**| `screenshots/dictionary_entry_subcategory.png` | Visual Evidence |
| **Update Set Overview Screenshot** | `screenshots/update_set_completed.png` | Visual Evidence |
