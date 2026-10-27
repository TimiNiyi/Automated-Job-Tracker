# n8n Job Application Updater

An AI-assisted workflow automation system that processes job-related emails from Gmail and maintains a structured job application tracker in Notion.

## Overview

Managing job applications can become tedious when application confirmations, interview invitations, rejections, and other updates arrive in separate emails. Manually reviewing these messages and updating a tracker takes time and makes it easier to overlook important information.

This project uses **n8n, Gmail, Google Gemini, and Notion** to automate parts of that process. It retrieves job-related emails, filters relevant messages, extracts structured information using AI, checks previously processed emails, and creates or updates records in a centralized tracking system.

The project consists of two complementary workflows:

1. **Historical Email Backfill** — processes existing job-related emails to establish an initial tracking dataset.
2. **Scheduled Job Tracker Updater** — periodically checks for newer emails and processes relevant messages to keep the tracker updated.

## Project Objectives

- Automate the collection of job application information from email.
- Use AI to extract relevant details from unstructured email content.
- Reduce repetitive manual data entry.
- Maintain a structured record of job applications and their statuses.
- Prevent the same email from being processed repeatedly.
- Connect email processing with a centralized Notion job tracker.

## Tools and Technologies

| Technology | Purpose |
|---|---|
| n8n | Workflow orchestration and automation |
| Gmail | Source of job application emails |
| Google Gemini | AI-powered information extraction |
| n8n Data Tables | Store processed email identifiers and extracted information |
| Notion | Centralized job application tracking |
| JSON | Workflow export and portability |

## Workflow Architecture

### 1. Historical Email Backfill

This workflow prepares an initial dataset from existing emails.

**Process:**

1. Manually trigger the workflow.
2. Retrieve Gmail messages from the selected historical period.
3. Filter messages using job-related keywords and conditions.
4. Pass relevant messages to an AI information extraction step.
5. Process messages in batches, with wait steps to help manage API request limits.
6. organize the extracted information into structured records in an n8n Data Table.
7. Create corresponding entries in the Notion job tracker.

This workflow is intended to reduce the effort required to populate the tracker with existing application history before relying on scheduled updates.

### 2. Scheduled Job Tracker Updater

This workflow handles ongoing email processing.

**Process:**

1. Start on a configured schedule.
2. Retrieve newer Gmail messages within the configured search window.
3. Filter messages to identify potentially relevant job-related emails.
4. Check the processed-email Data Table using the email identifier.
5. Pass eligible messages through AI extraction steps to identify relevant application information.
6. Use the extracted information to determine the appropriate record operation.
7. Insert or update records in the processed-email table.
8. Create a corresponding Notion job application page.

The schedule, Gmail search window, filters, credentials, and database references should be reviewed and configured for the environment in which the workflow is imported.

## Data Flow

```text
Gmail
  |
  v
Retrieve and Filter Emails
  |
  v
Check Processed Email Records
  |
  v
AI Information Extraction
  |
  v
Structure Application Information
  |
  v
Insert or Update Data Table Records
  |
  v
Create or Update Notion Tracking Records
```

The exact path through the workflow depends on the conditions and branches configured in n8n.

## Information Extracted

The AI extraction process is designed to turn unstructured email content into structured job application information. Depending on the email and configured extraction schema, relevant fields include:

- Company name
- Job title or role
- Job type, such as internship, full-time, or part-time
- Application stage or status
- Interview date, when available
- Email identifier and date
- Application-related information needed for tracking

Extracted values depend on the email content and the configured model schema. Missing or ambiguous details should be checked rather than assumed to be accurate.

## Duplicate Processing and Data Management

The processed-email Data Table acts as a reference for identifying messages that have already been handled.

The workflow uses email identifiers to check for previously processed messages before processing them again. This helps reduce duplicate processing of the same message.

**Important limitation:** Different emails about the same job application can have different email identifiers. For example, an application confirmation and a later interview invitation may come from the same company but represent different messages about the same application.

Email-ID deduplication alone does not reliably consolidate these messages into one application record. A more robust application-level matching mechanism—using normalized company names, job titles, and a stable application key—is a planned improvement.

## Repository Structure

```text
n8n-job-application-automation/
├── README.md
├── workflows/
│   ├── 01-historical-email-backfill.sanitized.json
│   └── 02-scheduled-job-tracker-updater.sanitized.json
├── docs/
│   ├── architecture.md
│   └── pre-publish-checklist.md
├── screenshots/
│   └── README.md
├── .gitignore
└── LICENSE
```

The workflow files are exported JSON configurations intended for inspection and adaptation. They may require additional configuration before they can run in another n8n instance.

## Setup and Configuration

### Prerequisites

- An n8n instance with permission to create and execute workflows.
- A Gmail account containing job application emails.
- A Google Gemini API credential configured in n8n.
- A Notion integration authorized to access the intended job application database.
- An n8n Data Table for storing processed-email records.

### Importing the Workflows

1. Clone or download this repository.
2. Open your n8n instance.
3. Import the historical backfill workflow JSON.
4. Import the scheduled updater workflow JSON.
5. Reconnect the required credentials.
6. Select or recreate the required Data Table and Notion database references.
7. Review the Gmail search queries, filtering rules, AI extraction schemas, and schedule.
8. Test the workflows with a small set of representative emails before processing larger volumes.

The sanitized exports do not include working credentials. Resource references may also need to be selected again after import.

### Recommended Configuration Checks

Before running either workflow, verify:

- Gmail access and search queries.
- Google Gemini model availability and API quota.
- AI extraction fields and expected output structure.
- Data Table columns and matching conditions.
- Notion database properties and permissions.
- Batch sizes and wait intervals.
- Schedule and email search window.
- Error handling and duplicate-processing behavior.

Keep the workflows inactive while configuring and testing them. Enable scheduled execution only after confirming the intended behavior in your own environment.

## Testing and Validation

A practical test plan should cover:

1. **Relevant email:** Confirm that a genuine application-related message passes the filters.
2. **Irrelevant email:** Confirm that unrelated messages are excluded.
3. **Information extraction:** Compare AI output against the original email.
4. **Previously processed email:** Confirm that the same email identifier is not processed again.
5. **Different emails, same application:** Check whether an application confirmation and interview email create separate records. This is an important test for the current application-level matching limitation.
6. **Missing information:** Verify that unavailable interview dates or job details are not fabricated.
7. **API limits:** Confirm that wait steps and batch processing behave as expected when multiple messages are processed.
8. **Notion output:** Verify that the resulting records contain the expected properties.

## Security and Privacy

This project interacts with email and job application information, which may contain personal data.

- Never commit API keys, passwords, OAuth tokens, or private credentials.
- Reconnect credentials within your own n8n instance.
- Review exported workflow JSON files before publishing them.
- Avoid publishing real email contents, personal contact details, or private application records.
- Use sanitized examples or screenshots when documenting execution.
- Apply the principle of least privilege when authorizing Gmail and Notion access.

The public repository should contain workflow configurations and documentation—not private email data or credentials.

## Current Limitations

- AI extraction can produce incomplete or incorrect fields and requires validation.
- API quotas can constrain processing speed.
- Email-ID checks identify repeated messages but do not fully solve application-level deduplication.
- Imported workflows require environment-specific credentials and resource configuration.
- The Notion integration must be tested against the target database's actual schema.

## Future Improvements

- Introduce a stable application key to match different emails belonging to the same application.
- Improve company and job-title normalization.
- Update existing Notion application pages instead of always creating new pages where appropriate.
- Add stronger error handling, retries, and execution logging.
- Improve AI extraction validation and confidence checks.
- Add notifications for interview invitations and significant application status changes.
- Expand test coverage for duplicate messages, ambiguous emails, and missing fields.
- Add sanitized workflow screenshots and a demonstration of the end-to-end process.

## What This Project Demonstrates

This project brings together workflow automation, API integration, AI-assisted data extraction, data organization, and productivity tooling in a practical use case.

It demonstrates an approach to reducing repetitive administrative work by connecting existing services and structuring information that would otherwise remain scattered across email conversations.

## License

This project is licensed under the MIT License. See [LICENSE](LICENSE) for details.

## Disclaimer

This repository is provided for learning, demonstration, and adaptation. The workflows require configuration and testing before use. AI-generated information should be validated, and the workflows should not be assumed to be production-ready without appropriate testing and error handling.
