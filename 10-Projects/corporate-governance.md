---
name: "Corporate Governance"
status: "active"
priority: "P1"
corporate_entity: "[[10-Projects/fortisyn/_corporate-hub|Fortisyn]]"
path: "/home/projects/corporate-governance/"
goals:
  - "Bank-ready corporate records for the three BC companies"
  - "Central securities registers, share certificates and directors' resolutions"
  - "US Letter print PDFs in each company's brand colours"
tags: [project, corporate-governance, fortisyn, innovatience, mercova]
created: "2026-09-01"
updated: "2026-09-01"
---

# Corporate Governance

Bank package for the three British Columbia companies — central securities registers, share
certificates, directors' resolutions and the Innovatience share transfer — under the
*Business Corporations Act*, S.B.C. 2002, c. 57.

## Ownership end-state (confirmed by Roy, 2026-09-01)

- **Fortisyn Holdings Ltd.** (BC1597227, inc. 2026-06-25) — Roy Hudlin sole shareholder,
  200 Class A Common (R01: 100 subscriber's shares, count fixed by Resolution No. 1 since
  no amount was specified at incorporation; R02: 100 consideration shares for the
  Innovatience exchange).
- **Innovatience Consulting Ltd.** (BC1555376, inc. 2025-09-03) — Fortisyn Holdings sole
  owner. Roy's 100 common shares transferred 2026-06-25 for 100 Fortisyn shares
  (s. 85(1) ITA rollover, Form T2057). Certificate C-001, issued at incorporation, recorded
  as cancelled on the transfer.
- **Mercova Retail Ltd.** (BC1597230, inc. 2026-06-25) — Fortisyn Holdings subscribed for
  the 100 Class A Common itself at incorporation; Roy was never a Mercova shareholder.

All shares are without par value; no issue prices. Records office: 309 - 951 Charland
Avenue, Coquitlam BC V3K 3K7.

## Deliverables — 15 final US Letter PDFs in `bank-package/`

- Registers (landscape), directors' resolutions and the transfer (official framed
  certificate-style layout; Fortisyn's two share resolutions are separate documents), banking
  resolutions for the Vancity accounts (Fortisyn and Mercova), four share certificates, and a
  print-list cover.
- PDFs are named `{Company} {Document}.pdf` (Roy's scheme, 2026-09-01), e.g.
  `Fortisyn Directors Resolution - Vancity Bank Account.pdf`.
- Each company's documents carry its colour logo and brand colours: Fortisyn bronze
  `#4A3210` + gold `#C9A227`; Innovatience charcoal `#1A1A1A` + silver `#8A8A8A`; Mercova
  blue `#0000A8` + green `#00A860`.
- Internal preparation notes and `[TO BE PROVIDED]` items stay in the working `.md`
  sources and are stripped at render — the PDFs contain the final records only.

## Open items

- Form T2057 filing date (CPA) — [[10-Projects/corporate-governance|tracker]] rows 4, 6.
- Certificate rights/restriction wording pending the actual articles (neutral wording used).
- Roy prints and signs: 6 resolution documents, numbered sequentially per company (Fortisyn:
  Nos. 1–3; Mercova: Nos. 1–2; Innovatience: No. 1), 1 transfer, 4 certificates.
- Banking resolutions: signing authority drafted as any one signatory (Roy Hudlin) — confirm
  against Vancity's requirements; printed dated 2026-09-01, correct by hand if executed
  another day.

## Regeneration

```bash
cd /home/projects/corporate-governance
python3 _templates/generate-certificates.py && python3 _templates/generate-pdfs.py
```
