# AI Procurement Agent

An end-to-end **AI-powered procurement automation workflow** built with **n8n Cloud**, **Groq Chat Model**, Google Drive, Excel (`.xlsx`) data, and human approval.

The agent helps automate procurement activities such as requirement understanding, inventory checking, procurement-policy validation, approved-supplier filtering, quotation comparison, purchase-order generation, and audit logging.

> **Important:** The AI Agent assists with procurement analysis and document generation. The final procurement approval remains with a human approver.

---

## 1. Project Overview

### Project Name
**AI Procurement Agent**

### Objective
To automate repetitive procurement activities while maintaining a **human-in-the-loop approval process** and a traceable audit trail.

### Business Problem

Traditional procurement processes can involve:

- Manual employee-request processing
- Spreadsheet-based inventory checks
- Manual procurement-policy verification
- Searching for approved suppliers
- Comparing supplier quotations
- Preparing purchase orders
- Maintaining audit records

These activities can be time-consuming and can lead to inconsistent processing or incomplete documentation.

### Solution

The AI Procurement Agent uses n8n to orchestrate the complete workflow:

```text
Employee Request
       ↓
AI Requirement Understanding
       ↓
Specification Extraction
       ↓
Inventory Check
       ↓
Procurement Policy Check
       ↓
Approved Supplier Search
       ↓
Quotation Collection
       ↓
Supplier Comparison
       ↓
AI Recommendation
       ↓
Human Approval
       ↓
     ┌───────────────┐
     │               │
   APPROVE          REJECT
     ↓               ↓
Generate PO     Rejection Audit
     ↓               ↓
Approved Audit ──────┘
       ↓
Existing Audit Log.xlsx
       ↓
Updated Audit Log
```

---

## 2. Key Features

- AI-based procurement requirement understanding
- Specification extraction from employee requests
- Inventory availability checking
- Procurement-policy validation
- Approved supplier filtering
- Supplier quotation comparison
- AI-generated procurement recommendation
- Human approval/rejection control
- Purchase Order generation
- Automated audit logging
- Existing Excel audit-log preservation
- Google Drive-based document storage
- Beginner-friendly n8n Cloud implementation

---

## 3. Technology Stack

| Technology | Purpose |
|---|---|
| **n8n Cloud** | Workflow orchestration and automation |
| **Groq Chat Model** | AI processing and generation |
| **Google Drive** | Storage and retrieval of Excel files |
| **Excel / XLSX** | Procurement input and reference data |
| **AI Agent** | Requirement understanding, comparison, and PO generation |
| **IF Node** | Business-rule branching |
| **Merge Node** | Combining records |
| **Extract From File** | Converting XLSX data into structured JSON |
| **Convert to File** | Converting processed records back to XLSX |

---

## 4. Project Architecture

### Main Workflow

```text
Employee Request
      ↓
Requirement Understanding
      ↓
Specification Extraction
      ↓
Inventory Check
      ↓
Procurement Policy Check
      ↓
Approved Supplier Search
      ↓
Quotation Collection
      ↓
Supplier Comparison
      ↓
Recommendation
      ↓
Human Approval
      ↓
Approval Decision
    ↙       ↘
Approve     Reject
  ↓           ↓
Generate PO  Rejection Audit
  ↓           ↓
Approved Audit
    ↘       ↙
      Audit Processing
           ↓
   Audit Log.xlsx
```

---

## 5. Input Files

The project uses Excel files stored in Google Drive.

### 5.1 Employee Requests

Suggested file:

```text
employee-requests.xlsx
```

Typical columns:

```text
request_id
employee_id
product
category
quantity
required_date
specifications
```

Example:

| request_id | employee_id | product | category | quantity |
|---|---|---|---|---:|
| R001 | E101 | Laptop | IT Equipment | 10 |

---

### 5.2 Inventory

Suggested file:

```text
inventory.xlsx
```

The inventory file should contain enough information to determine whether the requested item is available.

Typical fields:

```text
product
category
available_quantity
stock_status
location
```

---

### 5.3 Approved Supplier Matrix

Suggested file:

```text
approval_matrix(1).xlsx
```

Recommended columns:

```text
supplier_record_id
supplier_id
supplier_name
category
products
contact_email
phone
approved_status
supplier_rating
payment_terms
location
```

Example:

| supplier_id | supplier_name | category | products | approved_status | supplier_rating |
|---|---|---|---|---|---:|
| S001 | ABC Technologies | IT Equipment | Laptop/Monitor | Approved | 4.5 |
| S002 | XYZ Systems | IT Equipment | Laptop/Keyboard | Approved | 4.2 |
| S003 | Tech Solutions | Office Equipment | Printer/Scanner | Not Approved | 3.9 |

---

### 5.4 Supplier Quotations

Suggested file:

```text
supplier-quotations.xlsx
```

Recommended columns:

```text
quotation_id
request_id
supplier_id
product
quantity
unit_price
total_price
delivery_days
warranty_years
quotation_date
valid_until
payment_terms
quotation_status
```

---

### 5.5 Audit Log

The project uses an existing:

```text
Audit Log.xlsx
```

The audit log contains exactly these columns:

```text
audit_id
request_id
timestamp
agent_step
action
input_data
output_data
decision
performed_by
status
```

---

## 6. n8n Workflow Stages

### Stage 1 — Employee Request

The workflow starts by reading the employee request Excel file.

Typical nodes:

```text
Manual Trigger
     ↓
Google Drive - Download Employee Requests
     ↓
Extract From File
```

The XLSX file must be extracted into JSON before being passed to an AI Agent.

---

### Stage 2 — AI Requirement Understanding

The AI Agent interprets the employee's request and identifies:

- Product
- Category
- Quantity
- Specifications
- Required date
- Other procurement requirements

The AI should use only the supplied request information and should not invent missing values.

---

### Stage 3 — Inventory Check

The workflow checks whether the requested item is already available.

Example:

```text
Requested Quantity = 10
Available Quantity = 15
```

Result:

```text
Inventory Status = Available
```

If inventory is insufficient, the workflow can route the request for procurement or further review depending on the business rule.

---

### Stage 4 — Procurement Policy Check

The workflow verifies the request against procurement policy.

Possible output:

```text
policy_status = Approved
```

or:

```text
policy_status = Rejected
```

An IF node can then route the request accordingly.

Example:

```text
Value 1:
{{$json.policy_status}}

Operation:
is equal to

Value 2:
Approved
```

---

### Stage 5 — Approved Supplier Search

The supplier master is downloaded and extracted from Excel.

The workflow filters suppliers using:

```text
approved_status = Approved
```

and checks whether the supplier's:

- Category
- Products

match the employee requirement.

Only relevant approved suppliers should continue to quotation comparison.

---

### Stage 6 — Quotation Processing

Quotation records are filtered using fields such as:

```text
request_id
supplier_id
product
```

The workflow collects relevant quotation information including:

- Total price
- Unit price
- Delivery days
- Warranty
- Payment terms
- Supplier rating

---

## 7. Supplier Comparison AI Agent

The Supplier Comparison AI Agent evaluates the available quotations.

### Recommended comparison criteria

- Total price
- Delivery time
- Warranty
- Supplier rating
- Payment terms

### Recommended System Prompt

```text
You are an AI Procurement Supplier Comparison Agent.

Compare only the quotations and supplier information provided in the input.

Consider:
- Total price
- Delivery days
- Warranty
- Supplier rating
- Payment terms

Do not invent missing information.

Provide a transparent comparison and recommendation.

The recommendation is not the final purchasing decision.
A human approver must make the final procurement decision.
```

If multiple supplier records arrive separately, aggregate them before the AI Agent so that all relevant quotations are available to the model.

---

## 8. Human Approval

The human approval step is the main decision-control point.

Recommended fields:

```text
procurement_decision
approval_comments
```

Possible decision values:

```text
Approve
Reject
```

After the human approval node, use an IF node called:

```text
Approval Decision
```

Example:

```text
Value 1:
{{$json.procurement_decision}}

Operation:
is equal to

Value 2:
Approve
```

### TRUE

```text
TRUE
 ↓
AI Agent - Generate PO
```

### FALSE

```text
FALSE
 ↓
Prepare Rejected Audit
```

> Always inspect the Human Approval node's Output → JSON and use the actual field name produced by the node.

---

## 9. Purchase Order Generation

After human approval, the TRUE branch goes to:

```text
AI Agent - Generate PO
```

The AI should generate the PO using only approved procurement information.

Important information can include:

```text
Request ID
Supplier ID
Supplier Name
Product
Quantity
Unit Price
Total Price
Delivery Days
Warranty Years
Payment Terms
Purchase Order Number
```

The AI must not invent supplier, price, quantity, quotation, or approval information.

---

## 10. Approved Audit Record

After the PO-generation AI Agent, add an **Edit Fields** node:

```text
Prepare Approved Audit
```

Configure:

| Field | Example |
|---|---|
| audit_id | `AUD-{{$now.toMillis()}}` |
| request_id | Actual request ID |
| timestamp | `{{$now}}` |
| agent_step | `Approval Decision` |
| action | `Procurement Request Approved` |
| input_data | Request/product/quantity summary |
| output_data | PO generation result |
| decision | `Approved` |
| performed_by | `Human` |
| status | `Completed` |

Turn on:

```text
Include Other Input Fields
```

when you need to preserve useful incoming information.

---

## 11. Rejected Audit Record

The FALSE branch uses:

```text
Prepare Rejected Audit
```

Configure:

| Field | Example |
|---|---|
| audit_id | `AUD-{{$now.toMillis()}}` |
| request_id | Actual request ID |
| timestamp | `{{$now}}` |
| agent_step | `Approval Decision` |
| action | `Procurement Request Rejected` |
| input_data | Request/product/quantity summary |
| output_data | `Procurement request rejected by human approval` |
| decision | `Rejected` |
| performed_by | `Human` |
| status | `Completed` |

---

## 12. Audit Log Processing

The existing Audit Log is an XLSX file in Google Drive.

The purpose is to preserve existing records and append the new audit record.

### Architecture

```text
Approved Audit
      +
Rejected Audit
      ↓
Merge #1
Mode: Append
      ↓
New Audit Record
      +
Existing Audit Log.xlsx
      ↓
Google Drive - Download
      ↓
Extract From File
      ↓
Merge #2
Mode: Append
      ↓
Convert to XLSX
      ↓
Google Drive - Update
```

### Merge #1

Use:

```text
Mode = Append
```

This keeps the audit records flowing from the approval outcome.

### Download Existing Audit Log

Configure:

```text
Resource = File
Operation = Download
File = Audit Log.xlsx
```

The downloaded XLSX is usually stored in a binary property such as:

```text
data
```

### Extract Audit Log

Use:

```text
Extract From File
```

Choose the XLSX extraction option available in your n8n version.

Set the input binary field to the property produced by the Google Drive Download node, commonly:

```text
data
```

### Merge #2

Connect:

```text
Existing Audit Records → Input 1
New Audit Record       → Input 2
```

Set:

```text
Mode = Append
```

If the existing file contains 100 audit records and one new record is added, Merge #2 should output 101 items.

### Convert Back to XLSX

After Merge #2:

```text
Convert to File
```

Convert the JSON items back into XLSX format.

### Google Drive Update

The final Google Drive node should update the existing Audit Log.xlsx when the available operation supports updating an existing file.

Avoid blindly choosing Upload, because Upload may create:

```text
Audit Log (1).xlsx
```

instead of replacing the original file.

---

## 13. Important n8n Beginner Rules

### Rule 1 — Inspect JSON

Whenever you need an expression:

```text
{{$json.some_field}}
```

first open:

```text
Previous Node
→ Output
→ JSON
```

and confirm the field name.

### Rule 2 — XLSX is binary until extracted

Use:

```text
Google Drive Download
        ↓
Extract From File
        ↓
JSON
```

Do not send the raw binary XLSX directly to an AI Agent.

### Rule 3 — Use Append for audit history

For audit-log consolidation, use:

```text
Merge
Mode = Append
```

when the goal is to retain every record.

### Rule 4 — Human approval is final

The AI recommendation should not be treated as authorization.

### Rule 5 — Test each node

Use:

```text
Execute Step
→ Output
→ JSON
```

before moving to the next node.

---

## 14. Testing

Test the following scenarios:

### Test 1 — Approved Request

Expected:

```text
Employee Request
 ↓
Checks
 ↓
Supplier Comparison
 ↓
Human Approval = Approve
 ↓
Generate PO
 ↓
Approved Audit
 ↓
Audit Log
```

### Test 2 — Rejected Request

Expected:

```text
Employee Request
 ↓
Checks
 ↓
Supplier Comparison
 ↓
Human Approval = Reject
 ↓
Rejected Audit
 ↓
Audit Log
```

No purchase order should be generated on the rejection path.

### Test 3 — Unapproved Supplier

Expected:

```text
Supplier approved_status = Not Approved
        ↓
Supplier excluded
```

### Test 4 — Policy Failure

Expected:

```text
policy_status = Rejected
        ↓
Request does not continue to purchasing
```

### Test 5 — Audit Preservation

If the audit file contains:

```text
100 records
```

and one new audit record is created:

```text
Final file = 101 records
```

The original 100 records must remain.

---

## 15. Troubleshooting

### Binary data error

**Problem:**

The AI Agent reports that binary data is not supported.

**Solution:**

Use:

```text
Google Drive Download
        ↓
Extract From File
        ↓
AI Agent
```

---

### Empty expression

**Problem:**

```text
{{$json.request_id}}
```

returns empty.

**Solution:**

Open the previous node:

```text
Output → JSON
```

and identify the actual field name.

---

### Merge gives unexpected number of items

Check:

```text
Merge Mode = Append
```

and verify:

```text
Input 1
Input 2
```

connections.

---

### Old audit records disappear

This usually means only the new record was converted back to XLSX.

Correct flow:

```text
Existing Audit Log
        ↓
Extract
        ↓
Merge #2 ← New Audit Record
        ↓
Convert to XLSX
```

---

### Duplicate Audit Log files

If Google Drive creates:

```text
Audit Log (1).xlsx
```

instead of changing the original file, review the Google Drive operation.

Use an existing-file update operation when supported.

---

### AI invents information

Strengthen the prompt:

```text
Use only information supplied in the input.
Do not invent missing values.
If information is missing, explicitly state that it is unavailable.
```

---

## 16. Security and Governance

Recommended controls:

- Keep API credentials inside n8n credentials.
- Do not hard-code API keys.
- Use test data during development.
- Restrict Google Drive permissions.
- Keep human approval before purchase authorization.
- Do not allow AI to invent prices, supplier status, or policy rules.
- Keep an audit record for important actions.
- Avoid putting unnecessary sensitive information into audit fields.
- Back up the audit workbook before changing the update workflow.
- Review the workflow whenever procurement policy changes.

---

## 17. Production Checklist

Before production:

- [ ] Employee request file is accessible.
- [ ] Inventory file is accessible.
- [ ] Approved supplier file is accessible.
- [ ] Quotation file is accessible.
- [ ] Procurement policy document is accessible.
- [ ] Audit Log.xlsx is accessible.
- [ ] Google Drive credentials work.
- [ ] Groq credentials work.
- [ ] AI Agent prompts are tested.
- [ ] All expressions match actual JSON fields.
- [ ] Inventory check works.
- [ ] Policy check works.
- [ ] Supplier filter works.
- [ ] Supplier comparison works.
- [ ] Human approval works.
- [ ] Approved path generates a PO.
- [ ] Rejected path does not generate a PO.
- [ ] Audit record is created for both outcomes.
- [ ] Existing audit history is preserved.
- [ ] Updated XLSX is stored in the correct Google Drive file.
- [ ] Error handling has been tested.

---

## 18. Final Workflow Summary

```text
Employee Request
       ↓
AI Requirement Understanding
       ↓
Specification Extraction
       ↓
Inventory Check
       ↓
Procurement Policy Check
       ↓
Approved Supplier Search
       ↓
Quotation Collection
       ↓
Supplier Comparison
       ↓
AI Recommendation
       ↓
Human Approval
       ↓
Approval Decision
    ↙          ↘
Approve        Reject
  ↓              ↓
Generate PO    Rejected Audit
  ↓              ↓
Approved Audit ───┘
       ↓
Merge / Audit Processing
       ↓
Existing Audit Log.xlsx
       ↓
Updated Audit Log.xlsx
```

---

## 19. Project Completion Criteria

The AI Procurement Agent can be considered complete when a test procurement request can travel from request intake through:

1. Requirement understanding
2. Specification extraction
3. Inventory checking
4. Procurement-policy validation
5. Approved supplier filtering
6. Quotation processing
7. Supplier comparison
8. AI recommendation
9. Human approval
10. PO generation for approval
11. Rejection handling
12. Audit logging
13. Preservation of previous audit history

without losing required data or bypassing the human approval control.

---

## Author / Project Documentation

**Project:** AI Procurement Agent  
**Platform:** n8n Cloud  
**AI:** Groq Chat Model  
**Storage:** Google Drive  
**Data Format:** Excel / XLSX  
**Control Model:** Human-in-the-loop procurement approval
