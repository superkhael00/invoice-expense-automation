# Test Invoice Answer Key
Demo business: **Brightside Creative Studio** (fictional) · Currency: **USD** · Approval rule: **total > $500.00** · Sales tax: 8.25%

All vendors, emails (`.example` domain) and amounts are fictional.

| # | File | Vendor | Invoice # | Date (as printed) | Subtotal | Tax | Total (printed) | Tests | Expected result |
|---|------|--------|-----------|-------------------|---------:|----:|----------------:|-------|-----------------|
| 1 | 01_summit_office_clean_under | Summit Office Supply Co. | INV-4471 | 2026-09-02 | 175.40 | 14.47 | 189.87 | Happy path, under threshold | Auto-approved → Drive → Sheet. Category: Office Supplies |
| 2 | 02_clouddesk_software_over_threshold | CloudDesk Software LLC | CD-2026-0917 | September 12, 2026 | 1,188.00 | 98.01 | 1,286.01 | Over threshold | Approval email sent; logged after decision. Category: Software & Subscriptions |
| 3 | 03_metro_fiber_clean_under | Metro Fiber Internet | MF-883120 | 09/15/2026 | 104.99 | 8.66 | 113.65 | Happy path, US date format | Auto-approved. Category: Utilities / Internet |
| 4 | 04_rivera_web_design_over_threshold_no_tax | Rivera Web Design | RWD-0042 | September 18, 2026 | 2,550.00 | 0.00 | 2,550.00 | Over threshold, $0 tax | Approval email. Tax = 0 must NOT fail validation. Category: Contractors / Professional Services |
| 5 | 05_greenleaf_catering_MATH_ERROR | Greenleaf Catering | GLC-10385 | 2026-09-19 | 420.00 | 34.65 | **474.65** | Math check | **Validation fail**: 420.00 + 34.65 = 454.65, not 474.65 (off by $20.00) → flag for review |
| 6 | 06_apex_print_MISSING_INVOICE_NUMBER | Apex Print & Sign | *(none)* | 09/21/2026 | 285.00 | 23.51 | 308.51 | Required field | **Validation fail**: invoice_number missing → flag for review (AI must return null, not invent one) |
| 7 | 07_summit_office_DUPLICATE_of_01 | Summit Office Supply Co. | INV-4471 | 2026-09-02 | 175.40 | 14.47 | 189.87 | Duplicate check | **Duplicate**: same vendor + invoice # as #1 → not logged twice. Run AFTER #1. |
| 8 | 08_harbor_courier_FUTURE_DATE | Harbor Courier Services | HCS-55219 | January 15, 2027 | 135.00 | 11.14 | 146.14 | Date sanity | **Validation fail**: invoice date is in the future → flag for review |
| 9 | 09_pixel_pine_EXACTLY_500_boundary | Pixel & Pine Stock Media | PP-7730 | 09/24/2026 | 461.89 | 38.11 | 500.00 | Boundary (> vs >=) | **No approval** (500.00 is not > 500). Auto-approved. Category: Marketing / Creative Assets |
| 10 | 10_lakeside_janitorial_SCANNED_IMAGE | Lakeside Janitorial | LJ-2291 | 2026-09-26 | 320.00 | 26.40 | 346.40 | Scanned / image-only PDF | **No text layer** → Extract from File returns empty text → flag "manual review (scanned)". Do NOT send to AI. |

## Built-in variation (on purpose)
- **3 layouts / label styles**: "Invoice #" vs "Invoice No." vs "Invoice Number"; "TOTAL DUE" vs "Amount Due (USD)" vs "Balance Due".
- **3 date formats**: ISO `2026-09-02`, long `September 12, 2026`, US `09/15/2026`. The AI must normalize all to `YYYY-MM-DD`.
- #7 carries a "REMINDER - RESENT" stamp: realistic reason a vendor sends the same invoice twice.

## Suggested run order
1, 2, 3, 4, 9 (happy paths) → 5, 6, 8 (validation) → 7 (duplicate, needs #1 already logged) → 10 (scanned).
