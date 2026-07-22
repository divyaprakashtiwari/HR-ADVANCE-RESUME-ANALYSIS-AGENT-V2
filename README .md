# HR Advance Resume Analysis Agent

An n8n workflow that watches a Gmail inbox for job applications, screens each resume against the correct job description using Gemini, logs structured candidate data to Google Sheets, and replies to the candidate automatically — with duplicate detection, OCR fallback, and score-based routing built in.

---

## What it does

1. **Watches Gmail** for new emails (polls every 5 minutes).
2. **Classifies** each email — is this a job application, or not? (Gemini)
3. **Non-applications** get logged separately and labeled, instead of silently dropped.
4. **Extracts attachments**, uploads them to Google Drive, and converts Word/PDF/plain-text resumes into raw text (with OCR fallback via Gemini for scanned/image-only PDFs).
5. **Matches the email to the correct open role** using a "Job Roles" lookup sheet, so multiple job postings can be screened correctly at the same time — not just one hardcoded JD.
6. **Scores the candidate** against that role's job description: strengths, weaknesses, risk factor, reward factor, overall fit rating, and justification (Gemini).
7. **Extracts structured candidate data**: name, email, phone, LinkedIn, years of experience, key skills, education, location.
8. **Checks for duplicate applications** by email — updates the existing row instead of creating a second one.
9. **Routes by score**: fit ≥ 8 → logged to a dedicated "High Fit" tab for fast follow-up; fit ≤ 3 → logged to "Low Fit Review"; everything else → main data log.
10. **Replies to the candidate** automatically, including a disclosure that AI is used in initial screening.
11. **Marks the email as read** so it isn't reprocessed.

A human always makes the final call — nothing here auto-rejects a candidate.

---

## Files in this project

| File | Purpose |
|---|---|
| `HR_ADVANCE_RESUME_ANALYSIS_AGENT_v2.json` | Main n8n workflow — import this into n8n |
| `HR_Agent_Error_Alerts_workflow.json` | Small companion workflow that emails you if the main workflow fails |
| `Job_Roles_Template.xlsx` | Template for the sheet that maps job titles → job description file IDs |
| `HR_Advance_Agent_Data_Template.xlsx` | Template for the main data log (DATA / Non-Applications / Low Fit Review tabs) |
| `Resume_Jane_Doe.pdf` | Sample resume for test runs |
| `Job_Description_AI_Solutions_Architect.pdf` | Sample job description for test runs |
| `Sample_Test_Application_Email.md` | Sample email subject/body to send for testing |

---

## Setup

### 1. Credentials
Connect these in n8n before importing (or reconnect them after import — credential references are account-specific and won't transfer):
- **Gmail** (OAuth2) — for the trigger, sending replies, marking as read, and labeling
- **Google Drive** — for storing uploaded resumes
- **Google Sheets** — for all logging/lookup steps
- **Google Gemini (PaLM API)** — for classification, extraction, scoring, and OCR

> ⚠️ Google frequently deprecates Gemini model versions. If a Gemini node throws a `404 model not found` error, open it and switch to the current generally-available model (e.g. `gemini-3.5-flash` or the `gemini-flash-latest` alias) — check both Gemini Chat Model nodes in this workflow, not just one.

### 2. Spreadsheets
Import both `.xlsx` templates into Google Sheets (File → Import), or recreate their tabs/headers manually in your own spreadsheet:

**Job Roles sheet** — one row per keyword/synonym per role (lookup is an exact match):
| Job Title | Keywords | JD File ID |
|---|---|---|
| AI Solutions Architect | AI Solutions Architect | `<drive file id>` |
| AI Solutions Architect | AI Architect | `<drive file id>` |

**Main data spreadsheet**, with tabs:
- `DATA` — every screened candidate
- `Non-Applications` — emails the classifier ruled out
- `Low Fit Review` — candidates scoring ≤ 3
- `High Fit` — candidates scoring ≥ 8

### 3. Update node references
After import, open these nodes and point them at *your* spreadsheet/Drive IDs (they currently reference placeholder/example IDs):
- `Get Job Description`
- `Lookup Job Role`
- `Lookup Existing Candidate`
- `Update Existing Row`
- `Append Data1`
- `Log Non-Application`
- `Log Low Fit Review`
- `Log High Fit`
- `Label Non-Application` (needs a real Gmail label ID)

### 4. Error alerting
Import `HR_Agent_Error_Alerts_workflow.json`, set your own email in the **Alert Me** node, then in the main workflow go to **Settings → Error Workflow** and select it.

### 5. Test it
Upload `Job_Description_AI_Solutions_Architect.pdf` to Drive, add its file ID + keywords to the Job Roles sheet, then send `Sample_Test_Application_Email.md`'s content (with `Resume_Jane_Doe.pdf` attached) to your watched inbox. Run the workflow manually and step through each node to confirm the full path works.

---

## Known limitations

- **Multi-attachment handling** uses a small Code node (`Split Attachments`) — there's no built-in n8n node that fans out multiple email attachments into separate items.
- **OCR fallback** is wired up on the direct-PDF branch only; the Word→PDF branch would need the same pattern copied over if scanned Word-derived PDFs become common.
- **Lookup nodes use exact-match filters** — the Job Roles / duplicate-candidate lookups only match if the cell value is identical to the search value (see the Job Roles table format above).
- Sheet/Drive/credential IDs in the shipped JSON are placeholders or examples — every deployment needs its own.

---

## Compliance note

The Recruiter Agent's system prompt explicitly instructs it not to factor in protected characteristics, and the auto-reply discloses AI-assisted screening. Depending on your jurisdiction, additional disclosures or human-review requirements may apply to automated hiring tools (e.g. NYC Local Law 144, EU AI Act) — confirm with your legal/HR team before using this in production.
