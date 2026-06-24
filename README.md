n8n Workflow Automation Portfolio
This repository contains production-pattern workflow automation projects built with n8n, demonstrating progressive complexity across data validation, sub-workflow orchestration, parallel data processing, and analytical reporting pipelines.

Project 1: Employee Onboarding Pipeline
Overview
An automated HR onboarding pipeline that processes incoming employee submissions, validates data integrity, detects duplicate records, routes by department, and maintains a full audit trail with error handling throughout.
Architecture
The pipeline receives employee records via authenticated webhook, validates all required fields against a regex-checked schema, performs duplicate detection against existing Google Sheets records, routes valid submissions through department-specific sub-workflows, appends clean records to the master employee sheet, and logs every execution outcome to a separate audit sheet. Invalid submissions and system failures are caught and routed to Gmail alerts.
Concepts Demonstrated

Webhook authentication via custom header token
Batch processing using Split Out node
Field validation with regex email format checking
Duplicate detection via JavaScript Code node comparing incoming records against existing sheet data
Department-based routing using Switch node
Sub-workflow orchestration across Engineering, HR, and Finance pipelines
Parallel output handling across valid and invalid paths simultaneously
Audit logging with timestamp on every execution outcome
Error workflow with Error Trigger node catching system-level failures

Technical Challenges
The most significant challenge was correctly referencing data across nodes once the workflow grew beyond a linear structure. Early attempts used .item references which silently returned only the first record regardless of how many items were in the pipeline. This was resolved by switching to .first() and .all() depending on context, and by explicitly naming node references rather than relying on implicit input chaining.
A related issue was data nesting — incoming webhook data was structured as item.json.body.Email rather than item.json.Email, which caused the duplicate detection logic to silently fail. The fix required inspecting the raw JSON output at each node before writing any Code node logic, which became a consistent practice for the rest of the build.
Batch processing introduced a further complication: the Code node's $input.all() returned sheet rows rather than incoming employees because of how the node was positioned in the chain. This was resolved by explicitly referencing the upstream employee node via $('Edit Fields').all() while using $input.all() exclusively for the sheet comparison data.
Stack
n8n Cloud, Google Sheets, Gmail, JavaScript

Project 2: Sales Performance Reporting Pipeline
Overview
An automated weekly sales reporting pipeline that pulls data from multiple Google Sheets sources, merges and joins them, calculates KPIs per sales representative, detects week-over-week performance anomalies, and delivers a formatted HTML executive summary via email on a scheduled trigger.
Architecture
The pipeline runs every Monday morning via a Schedule Trigger. Two parallel Google Sheets reads pull current sales data and weekly targets simultaneously. A Merge node joins both datasets by representative name. The first Code node performs a join operation — matching each representative's actual revenue against their target, calculating achievement percentage, and computing week-over-week change. The second Code node applies anomaly detection logic, flagging any representative whose revenue dropped more than 20% compared to the previous week. The IF node routes flagged records to an immediate manager alert and all records to the Summarize node, which aggregates totals and averages across the full team. A final Code node constructs an HTML report body which is delivered via Gmail.
Concepts Demonstrated

Schedule Trigger for fully automated execution without manual intervention
Parallel data reads from multiple sheets merged into a single stream
Merge node combining two datasets on a common key
KPI calculation in JavaScript: revenue achievement percentage, week-over-week growth rate
Anomaly detection logic using percentage drop threshold
Summarize node for team-level aggregation: total revenue, average revenue, representative count
HTML email construction from structured data
Dual output routing: anomaly alert path and executive summary path running simultaneously

Technical Challenges
The primary difficulty was in the analytical code layer — transforming raw row-level data into reduced, meaningful metrics. The core problem was understanding that each item flowing through the pipeline represents one record, and meaningful analysis requires either reducing multiple records into one (aggregation) or enriching each record with derived values (KPI calculation) before routing decisions can be made.
The join operation between sales data and targets required matching records by representative name across two datasets, which meant iterating over one dataset inside a loop over the other — a nested loop pattern that was unintuitive coming from a node-based drag-and-drop mental model.
Anomaly detection required understanding percentage change as a formula rather than an absolute comparison — (previous - current) / previous * 100 — and applying it consistently across all records before any routing decision was made.
Stack
n8n Cloud, Google Sheets, Gmail, JavaScript

Repository Structure
n8n-workflow-portfolio/
├── README.md
├── beginner/
│   └── employee-onboarding.json
├── intermediate/
│   └── sales-reporting-pipeline.json
└── advanced/
    └── (in progress)
Importing Workflows
Each workflow is exported as a JSON file. To import into n8n: open your n8n instance, navigate to Workflows, select Import from File, and choose the relevant JSON file. Credentials will need to be reconfigured after import.