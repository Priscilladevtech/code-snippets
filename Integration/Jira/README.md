# Jira to ServiceNow Integration via IntegrationHub and Jira Spoke

Category: Integration/Jira  
Contributor: priscilladevtech  
  

## What This Solves
This code snippet enables bidirectional synchronization between Jira and ServiceNow, automating issue tracking. It creates ServiceNow incidents from Jira issues via webhooks, then pushes ServiceNow Story records to Jira as issues, streamlining collaboration between IT and development teams.

## Where to Use
- Organizations using Jira for agile development and ServiceNow for IT Service Management (ITSM).
- IT teams requiring real-time incident creation from Jira issues.
- Agile teams syncing ServiceNow Stories to Jira for task management.
- Environments with ServiceNow’s Agile Development module and Jira Spoke.

## How It Works
- Jira to ServiceNow: A Jira webhook sends issue data to ServiceNow, creating an incident with the issue’s summary and description using Jira Spoke’s webhook processing.
- ServiceNow to Jira: A Flow Designer flow triggers on Story creation, uses the Jira Spoke to create a Jira issue, and updates the Story with Jira’s ID, Key, and URL.
- Validation: A business rule ensures Story records have short_description and description before syncing to Jira, preventing errors (note: business rule is untested but designed for validation).

## Prerequisites
- Jira API token (Jira Settings > Security > API Tokens).
- ServiceNow plugins: Integration Hub, Jira Spoke, Agile Development.
- Personal Developer Instance (PDI, with a Jira Details section on the Story form (rm_story), including fields: Key (String), ID (String), URL (URL).
- Admin role in PDI for configuration.
- Jira connection configured in ServiceNow.

## Configure
### 1. Jira Configuration
- Create an API token in Jira (Settings > Security > API Tokens).
- Configure a webhook in Jira:
  - Go to Settings > System > Webhooks.
  - Name: ServiceNow Webhook.
  - URL: ServiceNow’s webhook endpoint (provided by Jira Spoke, e.g., https://your-instance.service-now.com/api/sn_jira_spoke/jira_webhook).
  - Events: Select Issue Created and optionally Issue Updated.
  - Test the webhook to ensure connectivity.

### 2. ServiceNow Configuration
- Install Plugins:
  - Install Integration Hub, Jira Spoke, and Agile Development via the ServiceNow Store.
- Create Jira Details Section:
  - On the rm_story form, add a section "Jira Details" with fields: Key (String), ID (String), URL (URL).
- Create Connection & Credential Alias:
  - Go to All > Connection & Credential Aliases.
  - Click New.
  - Name: Jira Spoke Connection.
  - Credential: Basic Auth (Jira email, API token).
  - Connection URL: Your Jira URL (e.g., https://your-jira.atlassian.net).
  - Save.
- Create Jira Webhook Connection:
  - Go to All > Jira Spoke > Webhook Registries.
  - Click New.
  - Name: Your Jira URL (e.g., https://your-jira.atlassian.net).
  - Token: Click the search icon to open token verification.
  - Token Verification Details:
    - Name: Jira URL Token.
    - Click Generate Token and save.
  - Save the webhook registry.
- Create Business Rule:
  - Go to System Definition > Business Rules.
  - Click New.
  - Name: Validate Jira Story.
  - Table: Story [rm_story].
  - When: Before, check Insert and Update.
  - Script: Paste from validate_jira_story.js (untested but designed to validate required fields).
  - Save.
- Create Flow Designer Flow:
  - Go to Flow Designer.
  - Click New > Flow.
  - Name: Jira Issue Creation.
  - Trigger: Record Created, Table Story [rm_story].
  - Action 1: Jira Spoke -> Create Issue.
    - Project ID: Your Jira project ID (e.g., PROJ-123).
    - Issue Type: Story.
    - Summary: Trigger -> Story Record -> Short Description.
    - Description: Trigger -> Story Record -> Description.
  - Action 2: Update Record.
    - Record: Trigger -> Story Record.
    - Table: Story [rm_story].
    - Fields:
      - ID: Create Issue -> ID.
      - Key: Create Issue -> Key.
      - URL: Create Issue -> URL.
  - Save and activate.

## Files
- jira_flow_setup.txt: Flow Designer configuration for ServiceNow-to-Jira sync.
- validate_jira_story.js: Business rule to validate Story records (untested).
- README.md: This documentation.

## Test Steps
1. In ServiceNow, create a Story (rm_story) with short_description and description filled.
2. Verify the Flow Designer flow creates a Jira issue and updates the Story with Jira’s ID, Key, and URL.
3. In Jira, create an issue and confirm a ServiceNow incident is created via the webhook.
4. (Optional) Test the business rule by creating a Story without a short description or description to ensure it blocks submission with an error.

## Troubleshooting
- Webhook Not Creating Incidents:
  - Ensure the Jira webhook URL matches the ServiceNow endpoint (https://your-instance.service-now.com/api/sn_jira_spoke/jira_webhook).
  - Verify the webhook registry token in Jira Spoke > Webhook Registries.
  - Check System Logs > Transactions (filter for /jira_webhook) for errors.
- Flow Not Triggering:
  - Confirm the flow is active in Flow Designer.
  - Ensure rm_story fields (short_description, description) are populated.
- Permission Issues:
  - Add admin role: Go to User Administration > Users, edit your user, and add the admin role.
- PDI Accessibility:
  - Ensure PDI is online at [developer.servicenow.com](https://developer.servicenow.com).
