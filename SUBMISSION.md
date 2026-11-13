# Invoicely Lite — PSE Support Lab Submission

- **Name:** Pakhi Mishra
- **Date submitted:** 11/14/2026
- **Repository link:** 
- **Total time spent (real time):** 2 hours

> All customer replies below are exercise drafts, not messages actually sent. Exercise clock: Monday, 16 November 2026, Asia/Dubai.

## Items fully investigated, and why I chose them

1. **Oasis Builders:** High business impact due to month-end processing, upload retries, and possible duplicate submissions. It also provided an opportunity to investigate the asynchronous import and ASP submission lifecycle.
2. **Al Noor Fuel:** Multiple invoices were rejected under a specific VAT rule. The workbook could be analysed and compared with the validator implementation.
3. **Ghaf Café:** The customer provided a reproducible date-format error, and the parser source allowed the cause and workaround to be verified.

Marina Bay and Sahara Medical were triaged and partially checked, but their historical incident evidence was insufficient for a defensible root-cause conclusion.

---

# Incident 1 — Oasis Builders

## Acknowledgement

- **Sent at (exercise clock):** 16 November 2026, 09:20 +0400
- **Reply:**

> Hello Rashid,
>
> Thank you for reporting this. I understand the urgency, especially with your November month-end close approaching. I’m checking the November upload and the individual invoice submission records to establish which invoices, if any, reached the ASP.
>
> Please do not retry the same file or manually resubmit invoices while we reconcile the existing records, as this could create duplicate submissions.
>
> I’ll share the initial findings and next steps by 10:00 UAE time.
>
> Regards,  
> Support Team

## Priority and justification

- **Priority:** P1 — provisional, pending confirmation of service-wide impact.
- **Why:** The customer reports an upload stuck in Processing, a subsequent error, and duplicate-number messages after retries. Month-end close increases business impact. P1 applies if no invoices are reaching the ASP for one or more customers. If only a subset is affected and other invoices continue reaching the ASP, P2 may be more appropriate.
- **SLA deadlines:** First response by 09:20–09:35; next update within 30–60 minutes; P1 resolution target four hours from receipt (13:05).

## Investigation log

| Time (exercise clock) | What I checked / did | What I found |
|---|---|---|
| 09:25 | Inspected the November workbook | 600 invoice lines across eight invoice numbers, OBS-2611-001 through OBS-2611-008; 75 lines per invoice. |
| 09:35 | Queried Oasis import records | Only the October import was present. No November import record was found. |
| 09:40 | Queried November invoice numbers | Zero matching November invoice records. |
| 09:45 | Checked October invoice history | All eight October invoices had Parsed, Validated, Submitted, and Accepted events. |
| 09:50 | Reviewed importer and submission source | Import is marked Completed before asynchronous ASP submission finishes. |
| 09:55 | Reviewed web proxy source | The proxy can return a 504 on timeout or 502 on another fetch error. A browser-side error does not establish that backend processing failed. |
| 10:00 | Checked available API logs and environment clock | Available API logs were dated 13 November. The container clock was also 13 November, earlier than the reported incident. No conclusion about the 16 November request could be drawn. |

## Root cause

- **What happened:** The November upload outcome cannot be established from the available database and historical logs. No November import or invoice records were found for Oasis.
- **Where:** `services/api/internal/importer/importer.go`, `services/api/internal/submit/submit.go`, and `web/app/api/[...path]/route.ts`.
- **Why it may happen:** The importer marks an import Completed before the asynchronous ASP submission queue has finished. The web proxy can return an error or timeout while backend processing may have progressed. This could explain why a customer sees an error and retries, but this is a hypothesis, not a confirmed explanation of this incident.
- **Evidence:** `evidence/database-findings.txt`, `evidence/importer-source.txt`, `evidence/asp-submission-source.txt`, and `evidence/operational-findings.txt`.
- **Confidence and open questions:** High confidence in the observed October records, missing November database records, and source-code behaviour. Low confidence in the actual November incident cause. Historical API logs, queue state, ASP records, and any November import request trace are still required.

## Customer resolution reply OR escalation note

- **Which one and why:** Engineering escalation is required because the November submission state cannot be reconciled from the available records.

**Escalation summary**

- **Priority:** P1 pending confirmation of impact.
- **Customer:** Oasis Builders Supply LLC.
- **Seller ID:** 2.
- **Reporter:** Rashid Al Mansoori.
- **Reported:** 16 November 2026, 09:05 UAE time.
- **Affected batch:** `oasis-builders-nov-2026.xlsx`; eight invoice numbers, 600 lines.
- **Symptoms:** Upload remained Processing, later showed an error, and retries produced already-exists messages.
- **Evidence:** No November import or invoice rows found. October batch was accepted. Available API logs and container time do not cover the reported incident.
- **Suspected cause:** Possible proxy timeout or response loss after backend processing, followed by customer retries. Unconfirmed.
- **Workaround:** Do not resend until invoice-level and ASP-level reconciliation is complete.
- **Exact ask:** Check historical import requests, worker queue activity, ASP submission records, and duplicate responses for the eight November invoice numbers. Confirm which invoices were accepted and which can safely be retried.
- **Next update:** 10:00 UAE time.

**Customer update**

> Hello Rashid,
>
> Our initial review confirms that the October batch was accepted, but the November batch is not visible in the database records currently available to Support. We are escalating this for a reconciliation against the historical submission and ASP records.
>
> Please continue to hold off on retrying the file. We will confirm the status of each invoice before recommending any resubmission.
>
> Regards,  
> Support Team

## Workaround offered

Do not retry the full workbook. First reconcile the eight invoice numbers against ASP records. Retry only invoices confirmed not to have been accepted, after duplicate risk is ruled out.

## Prevention ideas

- Make import completion distinguish between file processing completion and ASP delivery completion.
- Provide invoice-level submission status and ASP references in the portal.
- Add idempotency protection to retry operations.
- Provide a reconciliation report for partially completed imports.
- Improve proxy timeout and asynchronous job status visibility.

---

# Incident 2 — Marina Bay API restart alert

## Acknowledgement

- **Sent at (exercise clock):** 16 November 2026, 10:00 +0400
- **Sent to:** Priya Nair / Customer Success, with the support-on-call team included as appropriate.
- **Reply:**

> Hello Priya,
>
> Thank you for forwarding the alert and customer reports. We are treating the repeated API restart alert and intermittent 502 reports as a production incident while we establish the customer impact.
>
> We are checking API availability, affected uploads, and invoices remaining in Validated status. We will provide an update by 10:45 UAE time and confirm the next steps before the Marina Bay go-live call.
>
> Regards,  
> Support Team

## Priority and justification

- **Priority:** P1 provisional.
- **Why:** The alert reports three API restarts within 15 minutes and multiple customers report intermittent 502 errors and invoices stuck at Validated. Confirm whether any customers are unable to get invoices to the ASP. If only a subset is affected, reassess as P2.
- **SLA deadlines:** First response by 10:05–10:20; next update within 30–60 minutes; P1 resolution target 13:42 based on the alert receipt time of 09:42.

## Investigation log

Partial checks performed:

- Reviewed current Docker service status. API was healthy.
- Checked current API container restart count: zero.
- Reviewed available API logs, which were dated 13 November rather than the reported 16 November incident.
- Current health and restart count do not disprove the historical alert.

## Root cause

Not investigated — what I would check next:

- Historical Grafana API restart and availability metrics for 09:27–09:42 UAE time.
- Historical 502 request logs and traces.
- API container exit reason, OOM events, and host resource pressure.
- Database connectivity and ASP request latency.
- Invoice event history for invoices reportedly stuck at Validated.

## Customer resolution reply OR escalation note

Not investigated — what I would check next:

Escalate to Engineering/on-call with the critical alert, customer impact reports, exact incident time window, and request for historical restart and 502 traces. Keep Support as the customer communication owner.

## Workaround offered

Do not repeatedly retry affected uploads. Check individual invoice status before retrying and confirm whether invoices are already in the ASP.

## Prevention ideas

- Alert on restart count and sustained 5xx rate.
- Retain historical logs and traces with sufficient retention.
- Add queue-age and stuck-Validated alerts.
- Provide an incident dashboard with customer and invoice impact.

---

# Incident 3 — Al Noor Fuel BR-CO-15

## Acknowledgement

- **Sent at (exercise clock):** 16 November 2026, 10:30 +0400
- **Reply:**

> Hello Fatima,
>
> Thank you for sharing the workbook and the rejection details. I understand the importance of providing a clear explanation for your finance team and auditor.
>
> We are comparing the affected invoice totals and checking how the VAT validation is handled by the portal and ASP. We will share the invoice-level findings and recommended correction by 12:00 UAE time.
>
> In the meantime, please avoid changing quantities or unit prices until the discrepancy has been identified.
>
> Regards,  
> Support Team

## Priority and justification

- **Priority:** P2.
- **Why:** A subset of invoices has been rejected while other invoices in the same batch were accepted. The customer also has an external audit dependency.
- **SLA deadlines:** First response by 11:15; next update within 2–4 hours; resolution target eight hours from receipt (18:15), subject to business-hours clock rules.

## Investigation log

| Time (exercise clock) | What I checked / did | What I found |
|---|---|---|
| 10:40 | Inspected the workbook | Six invoice numbers, ANF-2611-0041 through ANF-2611-0046. |
| 10:50 | Calculated net and VAT totals | Invoices 0044 and 0046 show a 0.01 AED difference between summed line-rounded VAT and invoice-level rounded VAT. |
| 11:00 | Reviewed `validate.go` | Portal validator checks line net amount but does not implement the BR-CO-15 invoice-level total comparison. |
| 11:10 | Reviewed `submit.go` | HTTP 422 responses are stored as ASP rejections, including rule and message details when provided. |
| 11:15 | Queried Al Noor database records | Only the October import and accepted October invoices were present. No November import or invoice records were available. |

## Root cause

- **What happened:** The workbook calculation identifies a possible one-fils VAT rounding discrepancy in invoices ANF-2611-0044 and ANF-2611-0046.
- **Where:** `services/api/internal/validate/validate.go` and `services/api/internal/submit/submit.go`.
- **Why it may happen:** The portal validator does not implement the invoice-level BR-CO-15 calculation. The ASP can therefore reject a payload that passes the portal's existing line-level checks.
- **Evidence:** `evidence/validation-source.txt`, `evidence/asp-submission-source.txt`, and `evidence/database-findings.txt`.
- **Confidence and open questions:** The two one-fils differences are confirmed from workbook calculations. The actual ASP 422 response payload for the November invoices is unavailable, so the rounding discrepancy is a likely explanation, not a confirmed root cause for the rejection.

## Customer resolution reply OR escalation note

- **Which one and why:** Escalate for confirmation of the actual ASP rejection payload and expected rounding calculation before advising a financial correction.

**Escalation summary**

- **Priority:** P2.
- **Customer:** Al Noor Fuel & Lubricants LLC.
- **Seller ID:** 4.
- **Affected invoices:** ANF-2611-0041 through ANF-2611-0046.
- **Symptoms:** BR-CO-15 rejection on November invoices.
- **Evidence:** Workbook calculation shows 0.01 AED difference for 0044 and 0046. Portal validator does not implement BR-CO-15. November records are absent from the current database.
- **Exact ask:** Retrieve the ASP 422 response bodies and confirm the expected invoice-level VAT rounding method and affected invoice totals.
- **Workaround:** Do not change quantities or prices without confirmation. Reconcile invoice net, VAT, and gross totals with the ASP's expected calculation.
- **Next update:** 12:00 UAE time.

**Customer update**

> Hello Fatima,
>
> Our workbook review identified a one-fils VAT rounding difference in invoices ANF-2611-0044 and ANF-2611-0046. The portal's current validation code checks line amounts but does not perform the same invoice-level BR-CO-15 comparison reported in the rejection.
>
> We are checking the ASP rejection details to confirm the exact expected calculation before recommending a correction. Please retain the original quantities and prices for now.
>
> We will provide a further update by 12:00 UAE time.
>
> Regards,  
> Support Team

## Workaround offered

Reconcile the invoice-level net amount, VAT amount, and gross total with the ASP's expected rounding method. Do not change quantities or unit prices merely to force the totals to match.

## Prevention ideas

- Implement BR-CO-15 in portal-side validation.
- Use decimal arithmetic and a clearly documented rounding policy.
- Add regression tests for line-level versus invoice-level VAT rounding.
- Surface ASP rejection payloads and rule details in the portal.
- Provide an invoice-level calculation preview before submission.

---

# Incident 4 — Bluepeak / Sahara Medical

## Acknowledgement

- **Sent at (exercise clock):** 16 November 2026, 11:45 +0400
- **Reply:**

> Hello Daniel,
>
> Thank you for providing the connector details and the affected batch information. I understand the urgency, particularly with the hospital payment run approaching.
>
> We are checking the import and invoice submission records against the ASP references. Please do not resend the 40 invoices until we have reconciled their existing submission status, to avoid potential duplicates.
>
> We will provide an update by 13:00 UAE time.
>
> Regards,  
> Support Team

## Priority and justification

- **Priority:** P2, pending confirmation of wider impact.
- **Why:** A subset of invoices appears to have reached the buyer while others are missing, and many lack ASP references. The customer reports a payment-run dependency. Reassess as P1 if evidence shows no invoices are reaching the ASP across affected customers.
- **SLA deadlines:** First response by 12:30; next update within 2–4 hours; resolution target eight hours from receipt (19:30), subject to business-hours clock rules.

## Investigation log

Partial checks performed:

- Queried Sahara import and invoice records.
- Found the October batch containing 12 invoices, all Accepted with ASP message IDs.
- Queried November invoice numbers matching the reported batch identifier; zero rows returned.
- The current database snapshot does not contain the November 16 batch.

## Root cause

Not investigated — what I would check next:

- Locate the 40-invoice November import using the connector request ID and HTTP 201 response timestamp.
- Reconcile every invoice against the ASP using invoice number and sender ID.
- Inspect queue events, ASP responses, and message IDs.
- Compare the ERP connector's batch payload with the portal's import record.

## Customer resolution reply OR escalation note

Not investigated — what I would check next:

Escalate to Engineering and the integration team for reconciliation of the connector's HTTP 201 response against the missing import and ASP records. Request the original request ID, timestamp, and invoice number list.

## Workaround offered

Do not resend the batch until each invoice's ASP status has been confirmed. Resubmit only invoices confirmed not to have been accepted, after duplicate risk is ruled out.

## Prevention ideas

- Return a durable import ID and per-invoice status to the connector.
- Expose ASP message IDs in the integration response or status endpoint.
- Add reconciliation tooling for partial batch delivery.
- Alert on invoices remaining Submitted without an ASP reference.

---

# Incident 5 — Ghaf Café

## Acknowledgement

- **Sent at (exercise clock):** 16 November 2026, 12:30 +0400
- **Reply:**

> Hello Hessa,
>
> Thank you for sharing the workbook. I understand that the dates are correct from your perspective and that the rejection is frustrating.
>
> I have checked the date format expected by our upload system. It accepts dates in YYYY-MM-DD format or valid Excel date cells, whereas the affected cells in your file are text in DD/MM/YYYY format.
>
> You can retain the same intended dates and reformat the cells before uploading again. I can guide you through the steps if needed.
>
> Regards,  
> Support Team

## Priority and justification

- **Priority:** P3.
- **Why:** The issue affects one customer's upload and has a straightforward formatting workaround. It does not currently indicate a platform-wide outage.
- **SLA deadlines:** First response by 16:10; daily updates; resolution target three days.

## Investigation log

| Time (exercise clock) | What I checked / did | What I found |
|---|---|---|
| 12:35 | Inspected the workbook | Issue and due dates are stored as text in DD/MM/YYYY format. |
| 12:40 | Reviewed the date parser | `ParseDate()` accepts valid ISO dates and Excel serial values. |
| 12:45 | Compared the error with the code | DD/MM/YYYY text does not match either accepted format and is rejected. |

## Root cause

- **What happened:** The workbook stores dates as text in DD/MM/YYYY format, which the parser does not accept.
- **Where:** `services/api/internal/sheet/sheet.go`, `ParseDate()` and the date validation in `services/api/internal/validate/validate.go`.
- **Why it happens:** The parser accepts `YYYY-MM-DD` strings and Excel date serial numbers only.
- **Evidence:** `evidence/date-parser-source.txt` and the Ghaf workbook inspection.
- **Confidence and open questions:** High confidence. The parser implementation and workbook values directly reproduce the reported error.

## Customer resolution reply OR escalation note

- **Which one and why:** Customer resolution reply; the cause and workaround are confirmed.

> Hello Hessa,
>
> We identified the formatting issue. Your workbook currently contains dates such as `13/11/2026` as text. Our upload system accepts dates in `YYYY-MM-DD` format or valid Excel date cells.
>
> Please change the issue date to `2026-11-13` and the corresponding due date to `2026-12-13`, preserving the intended dates. Apply the same format to the other affected rows and upload the corrected file.
>
> If you prefer, you can send us a sample row and we can guide you through the formatting.
>
> Regards,  
> Support Team

## Workaround offered

Convert the issue-date and due-date cells to ISO date strings (`YYYY-MM-DD`) or valid Excel date cells. Preserve the original intended calendar dates.

## Prevention ideas

- Accept common regional date formats where unambiguous.
- Validate and preview date values before upload.
- Provide a clear template example and date-format guidance.
- Return an error message that includes an accepted-format example.

---

# Overall

## Prioritisation

I prioritised the cases according to business impact and the possibility that invoices were not reaching the ASP.

1. Oasis Builders — month-end urgency, repeated upload attempts, and potential duplicate risk.
2. Marina Bay — critical restart alert and multiple reports of intermittent 502 errors.
3. Al Noor Fuel — rejected invoices and an external audit dependency.
4. Sahara Medical — missing buyer deliveries and absent ASP references.
5. Ghaf Café — a single-customer date-format issue with a clear workaround.

The priorities remain subject to impact confirmation. In particular, Oasis and Marina Bay should be reassessed if evidence shows only a subset of invoices is affected.

## What I would do differently

I would first establish a reliable incident timeline and preserve the request IDs, import IDs, invoice numbers, and ASP references before attempting retries. I would also distinguish between import completion and final ASP acceptance from the start.

I would request historical observability data earlier when the current environment does not cover the incident timestamp. This would avoid treating current service health as evidence about a past incident.

I would also use a consistent invoice-level reconciliation checklist for batch incidents so that customer communication is based on confirmed invoice states rather than assumptions.

## What I learned / what I would need to learn

During this exercise I investigated:

- The difference between synchronous API responses and asynchronous background processing.
- How import records, invoice records, and invoice event histories relate.
- How ASP HTTP responses map to Accepted, Rejected, and Failed invoice states.
- How a proxy timeout can differ from the backend processing outcome.
- How date parsing and regional formats affect invoice validation.
- How VAT rounding differences can occur between line-level and invoice-level calculations.
- How to use Docker Compose, PostgreSQL queries, application logs, and source-code inspection during support triage.

I would next study distributed-system idempotency, queue observability, retry strategies, structured logging, tracing, and decimal arithmetic for financial calculations.

## Time log (real time)

| Date | From–to | What I worked on |
|---|---|---|
| [Actual date] | [Start–end] | Environment setup and application startup |
| [Actual date] | [Start–end] | Incident review and Oasis investigation |
| [Actual date] | [Start–end] | Marina Bay, Al Noor, and Sahara investigation |
| [Actual date] | [Start–end] | Ghaf investigation and evidence collection |
| [Actual date] | [Start–end] | Report preparation and final review |

**Total real time:** [Enter actual total. Do not use the exercise-clock duration.]

## AI usage disclosure

I used ChatGPT as an assistance tool for understanding the assignment, planning investigation steps, interpreting selected source-code sections and SQL outputs, and drafting customer communications and this report.

I ran the commands and queries in the provided local environment and reviewed the resulting output. I checked the date-parser, validator, importer, and ASP submission source files directly. I have distinguished observed results from hypotheses and have documented unavailable historical evidence rather than presenting an unverified root cause as confirmed.

The final investigation conclusions and submitted content were reviewed by me.