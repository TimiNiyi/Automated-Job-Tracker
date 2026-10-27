# Workflow Architecture

## Overview

The Job Application Updater consists of two n8n workflows that use Gmail as the source of job-related emails, Google Gemini for AI-assisted information extraction, n8n Data Tables for processed-email tracking, and Notion for job application management.

## Workflow 1: Historical Email Backfill

1. Retrieve historical Gmail messages.
2. Filter messages using job-related keywords and conditions.
3. Extract structured information using an AI information extraction node.
4. Process messages in batches, with wait steps to help manage API request limits.
5. Store processed-email information in an n8n Data Table.
6. Create corresponding records in Notion.

## Workflow 2: Scheduled Job Tracker Updater

1. Start on a configured schedule.
2. Retrieve newer Gmail messages.
3. Filter potentially relevant job-related emails.
4. Check the processed-email table using the email identifier.
5. Extract application details using AI.
6. Insert or update processed-email records according to the workflow logic.
7. Create corresponding Notion tracking records.

## Data Flow

Gmail → Email Filtering → Processed-Email Check → AI Extraction → Data Table → Notion

The actual path depends on the conditions and branches configured in the exported workflows.

## Data Integrity Considerations

Checking email identifiers helps prevent the same message from being processed repeatedly. However, separate messages about the same application have different identifiers. Application-level matching using normalized company names, job titles, and a stable application key is a potential improvement.

## Configuration

Imported workflows require valid Gmail, Google Gemini, and Notion credentials, as well as correctly configured Data Table and Notion database references. Test the workflows with representative messages before enabling scheduled execution.
