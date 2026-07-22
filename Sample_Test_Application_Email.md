# Sample Test Email — send this to the inbox your Gmail Trigger watches

**To:** (your workflow's connected Gmail inbox)
**Subject:** Application for AI Solutions Architect — Jane Doe
**Attachment:** Resume_Jane_Doe.pdf (attached separately)

---

**Body:**

Hi team,

I'm writing to apply for the AI Solutions Architect position. I've attached my resume for your review.

I have 6 years of experience building production ML systems and workflow automation platforms, most recently leading an LLM-based automation pipeline at Northwind Data Systems. I'd love the opportunity to discuss how my background fits this role.

Thank you for your time and consideration.

Best regards,
Jane Doe
jane.doe@example.com
+1 555 123 4567

---

### How to use this for testing
1. Send this email (with `Resume_Jane_Doe.pdf` attached) from any email account to the Gmail inbox your **Receive Resume** trigger is watching.
2. Upload `Job_Description_AI_Solutions_Architect.pdf` to your Google Drive, grab its file ID from the share link, and put that ID in the **Job Roles** sheet under `JD File ID`, with `Keywords` set to something like `AI Solutions Architect, AI Architect, Solutions Architect` — matching the subject line above.
3. Trigger a manual execution (or wait for the poll) and watch it flow through: Text Classifier → Upload to Drive → extraction → JD lookup → Recruiter Agent → Information Extractor → your DATA sheet.
