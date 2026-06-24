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


Intermediate: Daily News Digest Aggregator
What it does:

Every morning at 8am, pulls news from 3 different public APIs, merges the results, deduplicates by title, ranks by relevance score, and emails you a clean digest.
New concepts:

Schedule Trigger — no webhook, runs automatically
HTTP Request node — hitting real public APIs
Merge node — combining data from multiple parallel branches
Code node for ranking — scoring articles by keyword match
Deduplication — removing duplicate stories across sources
HTML email formatting — sending a styled digest not plain text

Flow:

Schedule Trigger — 8am daily
Three parallel HTTP Request nodes — fetch from 3 news APIs simultaneously (NewsAPI, HackerNews, Reddit RSS)
Merge node — combines all three into one stream
Code node — deduplicate by title, score each article by keywords you care about (AI, fintech, logistics)
Sort — highest score first
Code node — build HTML email body
Gmail — send digest

What makes it different:

Parallel branches, real external APIs, data merging, scoring logic, HTML output. None of which you touched in the beginner project.

Advanced: Sales Pipeline Automation
Webhook receives lead data → authenticates → validates → duplicate check in Postgres → enriches lead data via HTTP Request to a public company API → scores the lead with a Code node (based on company size, department, role) → routes high-score leads to one sub-workflow (sends personalized email + creates a follow-up task logged in Postgres) and low-score leads to another (sends generic email) → scheduled workflow runs every 24 hours pulling all pending leads from Postgres, aggregates stats, and emails a daily summary report → full audit trail → error handling throughout.
New concepts added: API enrichment, lead scoring logic, scheduled + webhook combo, aggregated reporting.
