"# n8n-workflow-portfolio" 
Write what problem it solves, what nodes are used, and how to test it


Project: Employee Onboarding + HR Automation System
What it does end to end:

A complete HR pipeline that handles new employee submissions, validates them, detects duplicates, stores records, notifies relevant parties, handles errors, and maintains a full audit trail.
Full workflow:

Webhook — receives new employee JSON (Name, Email, Age, Department, Role)
Header Auth — rejects unauthorized requests
IF: Field Validation — checks Name, Email, Department all exist and are non-null
False path → Gmail alert "Invalid submission" → Stop and Error
True path → Edit Fields — clean and structure data, add timestamp
Get rows in sheet — read existing employee records
Code node — duplicate email check, throw error if found
IF: Department Router — routes to different sub-workflows based on Department value
Sub-workflow: Engineering — sends Engineering-specific welcome email
Sub-workflow: HR — sends HR-specific welcome email
Sub-workflow: Finance — sends Finance-specific welcome email
Append row in sheet — write to main Employee Records sheet
Audit Log append — write to separate Audit sheet: timestamp, name, email, department, status
Error workflow — catches any crash, sends Gmail alert with error details, logs to Audit sheet as "system error"

Test cases to verify:

Valid new employee → goes through, emails sent, logged
Missing Email field → rejected, alert sent, logged
Duplicate email → rejected, alert sent, logged
Wrong auth header → 403, workflow never starts
Valid employee in each of three departments → correct sub-workflow fires


Intermediate: Customer Support Ticket System
Webhook receives support ticket (Name, Email, Issue, Priority) → validates data → checks for duplicate tickets from same email in Postgres database → routes based on Priority (High/Medium/Low) to different response sub-workflows → stores ticket in Postgres → sends acknowledgement email to customer → logs to audit sheet → error workflow catches failures.
New concepts added: Postgres node, database queries, priority-based routing.

Advanced: Sales Pipeline Automation
Webhook receives lead data → authenticates → validates → duplicate check in Postgres → enriches lead data via HTTP Request to a public company API → scores the lead with a Code node (based on company size, department, role) → routes high-score leads to one sub-workflow (sends personalized email + creates a follow-up task logged in Postgres) and low-score leads to another (sends generic email) → scheduled workflow runs every 24 hours pulling all pending leads from Postgres, aggregates stats, and emails a daily summary report → full audit trail → error handling throughout.
New concepts added: API enrichment, lead scoring logic, scheduled + webhook combo, aggregated reporting.
