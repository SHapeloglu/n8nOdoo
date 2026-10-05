# SESSION.md — n8nOdoo

Newest entry on top: what was done, decisions, open issues, next step.

---

## 2026-10-05

- Replaced the template-generated `CLAUDE.md`, `BACKLOG.md` and `SESSION.md` with content derived from the repo (README, ARCHITECTURE, PROJECT_PLAN, ROADMAP, docs/, schema, workflow).
- Open: no `.gitignore` (add before any code/venv/.env appears); no tests yet; the M1 partner-lookup workflow has not been run against a real Odoo 18 instance.
- Next: run `001_odoo_partner_lookup.json` against the Odoo test DB with a read-only API user and record results for the acceptance cases in `docs/M1-ODOO-PARTNER-MATCHING-API.md` (known VAT, customer-only, supplier-only, both, ambiguous, missing VAT, inactive, multi-company, auth failure, timeout).

---

## 2026-09-29

- Partner matching and document routing rules (`docs/M0-PARTNER-MATCHING.md`).
- Odoo partner matching API design (`docs/M1-ODOO-PARTNER-MATCHING-API.md`).
- First n8n workflow: read-only Odoo partner lookup by VAT.

## 2026-09-28

- Project documentation initialized (README, ARCHITECTURE, PROJECT_PLAN, ROADMAP, TASKS).
- M0 reuse/dependency assessment and invoice-LLM reference analysis.
- Normalized supplier invoice JSON schema.
