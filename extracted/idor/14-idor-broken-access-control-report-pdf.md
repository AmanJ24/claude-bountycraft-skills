# Sequential background-job IDs expose other users' exported PDF reports

Source completeness: partial (author states the draft was AI-assisted; endpoint names and some request details are screenshots not captured, and the sequential-versus-UUID claims are inconsistent)

- **Vuln type / sub-variant:** Predictable/enumerable object ID (background job ID, described as sequential though shown in UUID form) on an export-download path; direct object access to a stored file with no per-user authorization.
- **Target type:** Post-management web app with a "generate report" and a "export as PDF" feature; asynchronous job creates a PDF stored in a cloud storage bucket under a path keyed by job ID.
- **Recon signal:** Standard CRUD tests on posts were mostly protected, and the email-report feature sent only to the account owner. Clicking the PDF export and reviewing proxy history revealed two requests: one creating a job (response included a job ID with status CREATED) and one polling it that returned the download URL, so the job ID was the only thing keying the file.
- **Root cause:** The download location depends only on the job ID, with no signature, ownership check, or authorization on the storage path or the status endpoint; job IDs are enumerable by the account holder's own tooling per the author.
- **Exploitation steps:**
  1. Trigger a PDF export for own post and capture the job-creation and status requests.
  2. Note the job ID and the pattern of the resulting storage URL.
  3. Brute-force the ID space with Intruder (author states a numeric range up to millions) to gather valid job IDs belonging to other users.
  4. Insert each ID into the storage URL pattern and download the PDF.
- **Bypasses used:** None; the email-delivery protection was sidestepped by finding a second, direct-download export path.
- **Impact:** Read access to other users' private post reports, effectively all posts in the app.
- **Bounty:** Not stated in source.
- **Lesson / pattern:** Async export/job features create a second object type; check whether job or file IDs are guessable and whether the file store enforces ownership.
- **Source:**
  - Title: IDOR and Broken Access Control Risking Private Data Exposure
  - Author: Ahmed Qaramany (c0nqr0r)
  - URL: https://c0nqr0r.medium.com/idor-and-broken-access-control-risking-private-data-exposure-dd808412ed13
  - Date: 2024-10-13
  - Disclosed report: none
  - Raw file: `sources/idor/14-idor-broken-access-control-report-pdf.md`

Tags: uuid, enumeration, pii-disclosure, api
