# M0 — account_invoice_import_llm Analysis

## Source

Apexive `odoo-llm`, module `account_invoice_import_llm`, Odoo 18.

## What the module already solves

The module provides a useful reference implementation for the first MVP:

- PDF/image invoice extraction
- Mistral OCR
- structured invoice extraction
- vendor name/VAT extraction
- invoice number and dates
- currency
- subtotal/tax/total
- invoice lines
- OCA `account_invoice_import` integration
- manual processing of a draft invoice
- fallback parsing when embedded XML is unavailable

The module uses an explicit structured schema and converts the extracted result into OCA Invoice Pivot Format.

## Important architectural observation

The existing module is **Odoo-centric**: the document reaches an Odoo invoice/import wizard first, then OCR and LLM extraction happen inside Odoo.

Our project is intentionally **n8n-centric**:

```text
Email / WhatsApp / Web
        ↓
       n8n
        ↓
Document Intake
        ↓
OCR / AI
        ↓
Normalized JSON
        ↓
Odoo lookup + deterministic validation
        ↓
Approval when required
        ↓
Odoo record
```

Therefore we should not simply copy the module into our project.

## Reuse decision

### REUSE AS REFERENCE

Reuse the following concepts:

1. Structured invoice JSON schema
2. OCR → structured extraction flow
3. OCA Invoice Pivot Format mapping
4. Provider abstraction
5. Fallback from direct document annotation to OCR text + LLM
6. Explicit validation/error handling

### POSSIBLE DIRECT DEPENDENCY

Evaluate using `account_invoice_import_llm` directly in an Odoo deployment where the normal OCA invoice-import workflow is desired.

For the n8nOdoo platform, keep the dependency optional rather than making the n8n workflow depend on an Odoo UI wizard.

### DO NOT COPY

Do not copy the entire Odoo-centric processing flow into n8nOdoo. In particular, the platform should not require users to first create a draft invoice and then click "Process with AI".

## Proposed n8nOdoo adaptation

```text
1. Email receives PDF
2. n8n creates intelligent.document
3. n8n stores source metadata/original file
4. OCR provider processes bytes
5. LLM returns normalized invoice JSON
6. n8n validates JSON schema
7. Odoo lookup finds candidate supplier
8. Deterministic rules validate supplier/PO/amount/tax/currency
9. Risk engine decides automatic vs approval
10. Odoo draft vendor bill is created only after validation/approval
11. Original PDF is archived in DMS
12. Audit event is recorded
```

## Key design improvement for our project

The existing module maps AI output to OCA Invoice Pivot Format. We should introduce an intermediate, provider-neutral contract instead:

```json
{
  "document_type": "vendor_invoice",
  "document_version": "1.0",
  "supplier": {
    "name": "...",
    "tax_id": "..."
  },
  "invoice": {
    "number": "...",
    "date": "...",
    "due_date": "...",
    "currency": "TRY",
    "subtotal": 0,
    "tax": 0,
    "total": 0
  },
  "lines": [],
  "source": {
    "ocr_provider": "...",
    "llm_provider": "..."
  }
}
```

Only the Odoo adapter should convert this contract into Odoo/OCA-specific structures.

## Critical validation rules

AI extraction is not business validation.

At minimum:

- supplier tax ID/name must be matched against Odoo
- invoice number must be checked for duplicates
- company must be determined from the receiving context
- currency must be validated
- PO should be matched when available
- quantities and prices should be compared with the PO
- tax rates should be validated against configured Odoo taxes
- totals should be recalculated and compared with extracted values
- suspicious supplier bank-account changes require human verification

## Fallback strategy

The Apexive implementation has a useful two-level strategy:

```text
Direct document annotation
        ↓ failure/unavailable
OCR text
        ↓
LLM structured extraction
```

We should preserve this concept at the workflow level.

## MVP conclusion

`account_invoice_import_llm` substantially reduces the amount of invoice-OCR research we need to do. It should be treated as a **reference implementation and optional Odoo-side component**, while n8nOdoo owns intake, routing, validation, approval, idempotency and audit orchestration.

Next M0 task: inspect the underlying OCA `account_invoice_import` contract and the exact Odoo models/fields used by the Apexive adapter before defining our normalized `invoice.schema.json`.
