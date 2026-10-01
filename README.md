# Invoice & Expense Processing Automation

**AI-assisted accounts payable for small businesses, built in n8n.**
Vendor invoices arrive by email or form → AI extracts the fields → code checks every number → duplicates are blocked → anything over $500 waits for a one-click approval → the PDF is filed in Google Drive and every outcome is logged to a Google Sheets register.

**Design rule: AI reads, code checks, humans approve.**

▶ **Case study page:** open [`index.html`](index.html) (or the GitHub Pages link, once enabled) to replay all 10 test invoices.
📄 **Docs:** [System Documentation (public edition)](docs/System-Documentation-Invoice-Processing.pdf) · [Owner SOP](docs/Owner-SOP-Invoice-Processing.pdf)

> Demo business: Brightside Creative Studio (fictional). All vendors, invoices and amounts are test data.

---

## Results

| | |
|---|---|
| Test invoices handled as expected | **10 / 10** on the final live run (1 Oct 2026) |
| Time per invoice, receipt to Register | **7–16 seconds** (measured, excluding approval wait) |
| AI cost per invoice | **~$0.0004** (estimate, gpt-4.1-mini via OpenRouter) |
| Staff time saved at 100 invoices/month | **~12 hours/month** (estimate; assumptions in the docs) |

## How it works

```
1. Intake        Gmail (subject has "invoice" + PDF)  or  upload form
2. Read          PDF text extracted            → no text?        → Needs Review
3. Extract       AI returns structured fields (null if not printed)
4. Validate      code checks fields, math, dates, AI vs PDF   → fails? → Needs Review
5. Duplicate     vendor + invoice # vs Register               → match? → Duplicate
6. Approve       total > $500 → Approve / Reject email (waits for the click)
7. File and log  PDF to Drive (Processed / Needs Review) → row in Invoice Register → email labelled
```

| Check | Rule |
|---|---|
| Readable PDF | Under 30 characters of text → manual review, AI skipped |
| AI output | Temperature 0, JSON only, `null` when not printed, fixed category list |
| Required fields | Vendor, invoice #, invoice date, total |
| Math | Subtotal + tax = total (±$0.01) |
| Dates | Valid, not in the future, not over a year old, due ≥ invoice date |
| Hallucination guard | AI's invoice # and total must appear in the PDF text |
| Duplicate | Vendor + invoice # vs live Register rows and the same run |
| Approval | Over $500 → Gmail Send-and-Wait (Approve / Reject), 3-day timeout |
| Errors | Retries ×3; separate workflow emails failed step + likely fix |

## Test cases

| # | Scenario | Expected → Actual |
|---|---|---|
| 1 | Clean, under limit | Approved (auto) ✅ |
| 2 | Over limit ($1,286.01) | Approved by owner ✅ |
| 3 | Emailed invoice | Approved (auto), email labelled ✅ |
| 4 | Over limit, $0 tax | Rejected by owner ✅ |
| 5 | Math error (+$20.00) | Needs Review ✅ |
| 6 | Missing invoice number | Needs Review, AI returned null ✅ |
| 7 | Resent duplicate of #1 | Duplicate ✅ |
| 8 | Future date | Needs Review ✅ |
| 9 | Exactly $500.00 | Approved (auto), no approval ✅ |
| 10 | Scanned image | Needs Review, AI skipped ✅ |

## Repository

```
index.html                 Case study page (interactive replay, FAQ, evidence)
docs/                      System Documentation (public) and Owner SOP, PDF
workflows/                 n8n exports, credentials and IDs removed
test-invoices/             The 10 test PDFs + answer key
screenshots/               Evidence screenshots (private details blurred)
```

## Import the workflows

1. In n8n: **Overview → Create → Import from file** → `workflows/invoice-processing-error-alerts.json`, then `invoice-processing-main.json`.
2. Create credentials: Gmail OAuth2, Google Sheets OAuth2, Google Drive OAuth2, OpenRouter. Assign them on each node.
3. Create a Google Sheet (the Register) whose first row has these 24 headers, in order:
   `record_id, received_at, source, sender, file_name, vendor_name, invoice_number, invoice_date, due_date, currency, subtotal, tax, total, category, duplicate_key, validation_status, validation_errors, approval_required, approval_status, approved_by, decided_at, drive_file_url, status, notes`
   Then create two Drive folders (Processed, Needs Review) and a Gmail label `invoice-processed`.
4. Replace the placeholders: `YOUR_INVOICE_REGISTER_SHEET_ID`, `YOUR_PROCESSED_FOLDER_ID`, `YOUR_NEEDS_REVIEW_FOLDER_ID`, `YOUR_INVOICE_PROCESSED_LABEL_ID`, `approver@yourcompany.com`.
5. In Main → **Settings → Error workflow**, choose *Invoice Processing – Error Alerts*. Publish both.

## Stack

n8n (self-hosted) · OpenRouter (gpt-4.1-mini) · Gmail · Google Sheets · Google Drive

## Limits in V1 / roadmap

Scanned invoices go to manual entry (OCR planned) · weekly Google re-sign-in while the Google app is in testing mode · one invoice email per poll · settings inside nodes (Config tab planned) · no QuickBooks/Xero sync yet.

---

Built by **Khael Mendoza** · KhaelWorks · Tech VA and workflow automation
Contact: [mjmendoza.workph@gmail.com](mailto:mjmendoza.workph@gmail.com)
