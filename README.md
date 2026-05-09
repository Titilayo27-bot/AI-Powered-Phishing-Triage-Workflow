# AI-Powered Phishing Triage Workflow

An automated email security triage system that ingests inbound emails, analyzes them using Google Gemini AI, and logs structured incident records to Google Sheets in real time.

---

## Overview

Security teams waste hours manually triaging phishing emails. This workflow automates the entire process — from ingestion to risk scoring to incident logging — using n8n, Google Gemini, and Google Sheets.

Every email that hits the monitored inbox is automatically analyzed and logged with a risk score, severity rating, extracted URLs, phishing verdict, and recommended action.

---

## Tools Used

| Tool | Purpose |
|------|---------|
| n8n | Workflow automation platform |
| AgentMail | Email ingestion via webhook |
| Google Gemini 2.5 Flash | AI phishing analysis engine |
| Google Sheets | Incident log database |
| JavaScript | JSON parsing and data cleaning |

---

## Workflow Architecture

```
Ingest Emails → Parse Fields → Analyze Email Content → Display Results → Append Row in Sheet
```

### Node Breakdown

**1. Ingest Emails (Webhook)**
Receives POST requests from AgentMail whenever a new email arrives in the monitored inbox.

**2. Parse Fields (Set Node)**
Extracts three key fields from the raw webhook payload:
- `email_sender`
- `email_subject`
- `email_body`

**3. Analyze Email Content (Google Gemini 2.5 Flash)**
Sends parsed email data to Gemini with a structured prompt that returns a JSON risk assessment including risk score, severity, extracted URLs, phishing verdict, summary, and recommended action.

**4. Display Results (Code Node)**
Parses and cleans the Gemini JSON response, handling edge cases like markdown code fences.

**5. Append Row in Sheet (Google Sheets)**
Logs the full incident record to a Google Sheet with a timestamp.

---

## Risk Scoring Scale

| Score | Severity |
|-------|----------|
| 75 – 100 | High |
| 40 – 74 | Medium |
| 0 – 39 | Low |

---

## Sample Output

```json
{
  "sender_email": "attacker@gmail.com",
  "subject": "Urgent: Verify Your Account",
  "urls": ["https://suspicious-login.com"],
  "sender_domain": "gmail.com",
  "risk_score": 87,
  "severity": "High",
  "phishing_likely": true,
  "summary": "Email uses urgency language and contains a suspicious login URL",
  "reasons": ["Urgency trigger", "Suspicious URL", "Public email provider"],
  "recommended_action": "Quarantine and block sender"
}
```

---

## Setup Instructions

### Prerequisites
- n8n instance (cloud or self-hosted)
- AgentMail account with a monitored inbox
- Google Gemini API key (from Google AI Studio)
- Google Cloud project with Sheets API and Drive API enabled
- Google OAuth2 credentials configured

### Steps

**1. Import the workflow**
- Open your n8n instance
- Go to **Workflows → Import**
- Upload `workflow.json`

**2. Configure the Webhook**
- Open the **Ingest Emails** node
- Copy the **Production URL**
- Add it as an endpoint in AgentMail under **Webhooks**

**3. Add Gemini API Credential**
- Go to **Settings → Credentials → New**
- Select **Google Gemini (PaLM) API**
- Paste your API key from [Google AI Studio](https://aistudio.google.com)
- Connect it to the **Analyze Email Content** node

**4. Add Google Sheets Credential**
- Go to [Google Cloud Console](https://console.cloud.google.com)
- Create a new project
- Enable **Google Sheets API** and **Google Drive API**
- Create an **OAuth 2.0 Client ID** (Web Application)
- Add your n8n OAuth redirect URL under Authorized Redirect URIs:
  `https://YOUR-N8N-INSTANCE/rest/oauth2-credential/callback`
- Add your Gmail as a test user under **Audience**
- Copy the **Client ID** and **Client Secret** into n8n

**5. Create the Google Sheet**
Create a new Google Sheet named `Phishing Incident Log` with these headers in Row 1:

| timestamp | sender_email | subject | risk-score | severity | phishing likely | urls | summary | recommended_action |
|-----------|-------------|---------|-----------|----------|----------------|------|---------|-------------------|

**6. Connect the Google Sheets Node**
- Open the **Append row in sheet** node
- Select your spreadsheet and sheet tab
- Map the columns to the output fields

**7. Publish**
- Click **Publish** in n8n
- The workflow will now run automatically on every incoming email

---

## Google Sheet Headers (Copy-Paste Ready)

```
timestamp	sender_email	subject	risk-score	severity	phishing likely	urls	summary	recommended_action
```

---

## Security Note

> ⚠️ Never commit real credentials to GitHub.
> The `workflow.json` in this repo has all credential IDs replaced with placeholder values.
> Re-add your own credentials after importing into n8n.

---

## License

MIT License — free to use, modify, and distribute.

---

## Author

Built by **Babatunde Usrah Titilayo**
