# n8n Workflow Automation Portfolio

This repository contains production-pattern workflow automation projects built with n8n, demonstrating progressive complexity across data validation, routing, sub-workflow orchestration, and AI-assisted document processing.

---

## Project 1: Employee Onboarding Pipeline

### Overview

An automated HR onboarding pipeline that processes incoming employee submissions, validates data integrity, detects duplicate records, routes by department, and maintains a full audit trail — with error handling throughout.

### Architecture

The pipeline receives employee records via authenticated webhook, validates all required fields against a regex-checked schema, performs duplicate detection against existing Google Sheets records, routes valid submissions through department-specific sub-workflows, appends clean records to the master employee sheet, and logs every execution outcome to a separate audit sheet. Invalid submissions and system failures are caught and routed to Gmail alerts.

### Concepts Demonstrated

- Webhook authentication via custom header token
- Batch processing using Split Out node
- Field validation with regex email format checking
- Duplicate detection via JavaScript Code node comparing incoming records against existing sheet data
- Department-based routing using Switch node
- Sub-workflow orchestration across Engineering, HR, and Finance pipelines
- Parallel output handling — valid path and invalid path simultaneously active
- Audit logging with timestamp on every execution outcome
- Error workflow with Error Trigger node catching system-level failures

### Technical Challenges

The most significant challenge was correctly referencing data across nodes once the workflow grew beyond a linear structure. Early attempts used `.item` references which silently returned only the first record regardless of how many items were in the pipeline. This was resolved by switching to `.first()` and `.all()` depending on context, and by explicitly naming node references rather than relying on implicit input chaining.

A related issue was data nesting — incoming webhook data was structured as `item.json.body.Email` rather than `item.json.Email`, which caused the duplicate detection logic to silently fail. The fix required inspecting the raw JSON output at each node before writing any Code node logic, which became a consistent practice for the rest of the build.

Batch processing introduced a further complication: the Code node's `$input.all()` returned sheet rows rather than incoming employees because of how the node was positioned in the chain. This was resolved by explicitly referencing the upstream employee node via `$('Edit Fields').all()` while using `$input.all()` exclusively for the sheet comparison data.

### Stack

n8n Cloud, Google Sheets, Gmail, JavaScript

---

## Project 2: Intelligent Document Processing Pipeline

*In progress*

---

## Repository Structure

```
n8n-workflow-portfolio/
├── README.md
├── beginner/
│   └── employee-onboarding.json
└── advanced/
    └── document-processing-pipeline.json
```

## Importing Workflows

Each workflow is exported as a JSON file. To import into n8n: open your n8n instance, navigate to Workflows, select Import from File, and choose the relevant JSON file. Credentials will need to be reconfigured after import.