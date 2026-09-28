# n8nOdoo

Open-source intelligent document and business-process automation platform for Odoo 18, orchestrated by n8n.

> **AI reads. Odoo knows. Rules validate. Human approves when necessary. n8n orchestrates. DMS stores. Audit proves.**

## Vision

Connect Email, WhatsApp, Web and API inputs to Odoo through a secure, auditable workflow engine. AI is used for classification, OCR and structured extraction; deterministic rules and Odoo master data remain authoritative.

## First production slice

**Email → Supplier Invoice → OCR/AI → Odoo Partner/PO validation → Approval → Draft Vendor Bill → DMS → Audit**

## Principles

- n8n is the orchestrator, not the accounting/business-rule authority.
- Odoo is the source of truth for partners, products, orders, invoices and company data.
- AI extracts and interprets; it must not invent business facts.
- Validation rules are deterministic and independently testable.
- High-risk actions require human approval.
- Original documents are retained in DMS.
- Every processing step is auditable and idempotent.
- Document content must never be treated as system instructions.

## Planned components

- Odoo 18 custom module: `intelligent_document`
- n8n workflows and reusable sub-workflows
- OCA DMS for document storage/classification
- Pluggable OCR/LLM providers
- JSON schemas for normalized document data
- Rule and approval engine
- Audit and idempotency layer

## Status

The repository starts with architecture and research. Implementation begins after the reuse/fork/rewrite assessment in M0.

## License

License will be finalized before the first implementation release after dependency/license compatibility review.
