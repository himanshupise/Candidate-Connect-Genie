# Candidate Connect Genie : Workato Genie
 
A Workato package that automates candidate outreach, rejection, follow-up and reply classification for recruiting teams. Recruiters work through a Slack-based Genie, and the package handles Gmail delivery, follow-up scheduling and AI-powered reply classification in the background.
 
> **Guide version:** 1.0 (2026) | **Audience:** client administrators and operations teams
 
---

 
## 1. Project Overview
 
### Purpose
 
This package delivers a ready-to-import Workato solution that handles candidate communication from first outreach through to response tracking. It contains recipes, skill recipes, a Genie, data tables and connection shells, so it can be deployed to a customer workspace and running end to end with minimal setup.
 
### Business problem solved
 
Recruiters spend significant time on repetitive candidate emails: sending outreach, sending rejections, chasing non-responders and reading replies. This package removes that manual work:
 
- **Outreach and rejection emails** are sent when a recruiter approves or rejects a candidate, using a Slack conversation with the Genie.
- **Follow-up emails** go out automatically to candidates who have not replied (maximum of three follow-ups).
- **Candidate replies** are detected, classified by AI and stored, so recruiters can see candidate status without reading every inbox thread.
 
### Systems integrated
 
| System | Role in the solution |
| --- | --- |
| **Gmail** | Sends outreach, reject and follow-up emails, and detects candidate replies |
| **OpenAI** | Classifies candidate replies (model used by the recipes: `gpt-5.6-luna`) |
| **Slack** | Chat interface through which recruiters use the Candidate Connect Genie Genie |
| **Workato** | Hosts the recipes, Genie, data tables and connections |
 
---
 
## 2. Features and Workflows
 
### Recipes (trigger-based, run automatically once activated)
 
| Recipe | Trigger | What it does |
| --- | --- | --- |
| `[CECG] REC │ 001 │ Monitor Candidate Response with openai` | A candidate replies by email | Classifies the reply using OpenAI and stores the result in `candidate_outreach_and_follow_up_mail_details` |
| `[CECG] REC │ 002 │ Follow Up Candidate Email Trigger` | Scheduled, every 1 day at 1:00 AM | Fetches candidates with fewer than 3 follow-ups and sends follow-up emails, updating the follow-up count |
 
**Follow-up rules:** a candidate receives at most 3 follow-ups, spaced at roughly 3 days, 7 days and up to 14 days, until the candidate responds.
 
### Skill recipes (callable by the Genie)
 
| Skill recipe | When it runs | What it does |
| --- | --- | --- |
| `[CECG] SKILL │ 001 │ Send out-reach email` | A recruiter approves a candidate | Sends the outreach email to the candidate and records it in `candidate_outreach_and_follow_up_mail_details`. If the recruiter's name and email are given, the recruiter also receives a notification copy |
| `[CECG] SKILL │ 002 │ Send Reject email` | A recruiter rejects a candidate | Sends the rejection email to the candidate, with the same optional recruiter notification copy |
 
Recruiter name and email are **optional** for both skills. Without them, only the candidate receives the email, and no recruiter record needs to exist in any data table.
 
### Genie
 
- **Candidate Connect Genie :** the Slack entry point for recruiters. It has both skill recipes registered and does not send an email unless a candidate name, email and role are provided.
 
### Data tables
 
- `candidate_outreach_and_follow_up_mail_details`: each email sent, follow-up count and classified candidate response.
- `openai_audit_log`: a record of each OpenAI call.
 
### Connections
 
- Gmail account, OpenAI account and Slack connection (shells only; credentials are never included in the package).
 
---
 
## 3. Package Contents
 
| Field | Value |
| --- | --- |
| **Package name** | Candidate Connect Genie (CCE) |
| **Deployment guide version** | 1.0 (2026) |
| **Root project folder** | `Candidate Connect Genie` |
 
### Included assets
 
| Asset type | Count | Name(s) | Purpose |
| --- | --- | --- | --- |
| Recipes | 2 | `[CECG] REC │ 001 │ Monitor Candidate Response with openai`<br>`[CECG] REC │ 002 │ Follow Up Candidate Email Trigger` | Reply classification and scheduled follow-ups |
| Skill recipes | 2 | `[CECG] SKILL │ 001 │ Send out-reach email`<br>`[CECG] SKILL │ 002 │ Send Reject email` | Outreach and rejection emails triggered through the Genie |
| Genie | 1 | Candidate Connect Genie | Conversational Slack interface for recruiters |
| Data tables | 2 | `candidate_outreach_and_follow_up_mail_details`<br>`openai_audit_log` | Candidate email tracking and OpenAI audit trail |
| Connections | 3 | Gmail, OpenAI, Slack | External system access (re-authorized after import) |
 
The import should show **exactly** these assets.
 
### Folder layout
 
| Folder | Purpose |
| --- | --- |
| `Candidate Connect Genie` | Root project folder |
| `Recipes` | Trigger recipes |
| `Skill Recipes` | Outreach and reject skills |
| `Data tables` | Working data tables |
| `Connections` | Gmail, OpenAI and Slack connections |
 
---
 
## 4. Installation and Deployment

### 4.1 Import into Workato
 
1. Log in to the customer's Workato workspace.
2. Create a folder named **Candidate Connect Genie** (it must not already exist, so the import has a clean destination).
3. Go to **Tools → Recipe Lifecycle Management**.
4. Select **Import Package**, choose the **Candidate Connect Genie** folder as the destination, and upload the package zip.
5. Confirm the import finishes without errors and that the expected assets appear (2 recipes, 2 skill recipes, 1 Genie, 2 data tables, 3 connections).
 
> Recipes are imported **inactive and unconfigured**.
 
### 4.2 Configure connections and environment properties
 
1. Open the imported project.
2. Set the environment property: `companyName` = your organization name.
3. Re-authorize all three connections with the customer's own credentials (see [Section 5](#5-configuration-and-prerequisites)).
4. Double-check every property and connection before activating anything. A missed value here is the most common cause of failure in later steps.
 
### 4.3 Activate and verify (in this order)
 
**Step 1: Activate the skill recipes.** No manual run is needed; they run when the Genie calls them. For each skill, run it once through the Genie with a test candidate, check Job history, and confirm the expected data table rows. Move on only after a clean run.
 
**Step 2: Activate the trigger recipes.**
 
1. `[CECG] REC │ 002 │ Follow Up Candidate Email Trigger`: confirm the follow-up count updates after the nightly run.
2. `[CECG] REC │ 001 │ Monitor Candidate Response with openai`: send a test reply and confirm it is classified and stored.
 
**Step 3: Activate the Genie and register skills.** Open the Candidate Connect Genie and confirm both skills are registered and active. Test with a real recruiter prompt in Slack (for example, approving a test candidate).
 
**Step 4: Configure Slack.**
 
1. Authenticate the Slack connection via OAuth from a Slack Admin account (**Tools → Connections**).
2. In the Genie's **End user access**, create an end-user group (for example, "Test") and add the intended recruiters.
3. In the Genie's **Triggers**, click **+ Add** and add Slack as a "Chat with this genie in" interface. Confirm it shows Active (green dot).
4. Verify access is limited to the intended users or groups, not the whole "Everyone" system group, unless that is intentional.

---
 
## 5. Configuration and Prerequisites
 
### Workato
 
- **Recipe Lifecycle Management** available in the customer workspace (Tools → Recipe Lifecycle Management).
- Workspace plan and task capacity assigned. Capacity must cover both the nightly follow-up recipe and the reply-monitoring recipe, which runs on every reply.
 
### Customer accounts and credentials
 
| Application | Requirement |
| --- | --- |
| **Gmail** | The customer's Gmail account that sends outreach, reject and follow-up emails and receives candidate replies. Must be authorized in Workato with permission to **send and read** mail |
| **OpenAI** | A customer API key with access to the model used by the recipes (`gpt-5.6-luna`) and enough quota for the expected volume of replies |
| **Slack** | The customer's Slack workspace, with a Slack Admin available to approve the OAuth connection. The account must be able to post and read in the channel(s) the Genie uses |
 
### Environment properties
 
| Property | Value |
| --- | --- |
| `companyName` | Organization name |
 
### What customers may and may not change
 
**May change:** connection credentials, the schedule time of the follow-up trigger (within agreed windows), starting/stopping recipes, and the Genie's skill active/inactive toggles.
 
**Should not change:** core logic of the outreach, reject, follow-up and monitoring recipes; the follow-up rules (maximum of three follow-ups, spaced over 14 days) and response categories, unless agreed with the solution owner; anything in another customer's workspace.
 
---
 
## 6. Security, Testing and Support
 
### Credential handling
 
- **Credentials never travel with the package.** Connections must be re-authorized after every import into a new workspace.
- **Never hard-code secrets** (OpenAI keys, Gmail tokens, Slack tokens) in recipe steps. Always use Connections.
- Keep customer-specific values (such as `companyName`) in environment properties, not in recipe steps.
- Do not commit credentials, API keys or exported connection details to this repository.
- Each customer gets a **separate workspace**. Invite users only into their own workspace, and do not add customer users to the parent Development workspace unless they are co-builders under NDA.
- Document who owns **credential rotation** for OpenAI, Google/Gmail and Slack.
- Restrict Genie access to the intended end-user group rather than "Everyone".
 
### Validation and go-live checklist
 
**Package import**
 
- [ ] Package version noted and zip stored in the release location
- [ ] `companyName` environment property filled in
- [ ] Both trigger recipes present
- [ ] Both skill recipes present
- [ ] Both data tables present
- [ ] Gmail, OpenAI and Slack connections connected
- [ ] Genie present, with both skills attached and active


### Troubleshooting
 
| Symptom | Things to check |
| --- | --- |
| Recipe fails on first run | A connection or the `companyName` property was missed. Re-check every connection and property |
| Emails not sent or replies not detected | Gmail authorization must include permission to send **and** read mail |
| Replies not classified | OpenAI key validity, model access (`gpt-5.6-luna`) and quota. Check `openai_audit_log` and Job history |
| Genie not visible or not responding in Slack | Slack OAuth authorized by a Slack Admin, Slack trigger shows Active, and the user is in the end-user group |
| Duplicate follow-ups after an upgrade | Rows in `candidate_outreach_and_follow_up_mail_details` were cleared. Restore from backup and retain rows on future imports |
| Follow-ups running at the wrong time | Re-check the REC 002 schedule (every 1 day at 1:00 AM) after import |
 
For any failure, start with **Job history** in Workato and inspect the failed step.
 
### Ownership and support
 
| Item | Detail |
| --- | --- |
| **Product owner** | Harbinger Group |
 
Changes to core recipe logic, follow-up rules or response categories should be agreed with the solution owner and shipped as a new versioned package.
 
---
 
© 2026 Harbinger Group
