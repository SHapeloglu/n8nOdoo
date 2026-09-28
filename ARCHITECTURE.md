# Architecture

## System flow

```
Email / WhatsApp / Web / API
            |
            v
           n8n
            |
            v
      Document Intake
            |
            v
        OCR / Vision
            |
            v
       AI Classification
            |
            v
      Structured JSON
            |
            v
       Odoo Lookup
            |
      +-----+-----+
      |           |
   Partner      PO/Order
      |           |
      +-----+-----+
            |
            v
     Deterministic Rules
            |
      +-----+-----+
      |           |
   Automatic    Approval
      |           |
      +-----+-----+
            |
            v
          Odoo
        /       \
       v         v
      DMS      Audit
```

## Responsibility boundaries

### n8n

- Channel integration
- Workflow orchestration
- Retry/timeout handling
- Routing
- Calling OCR/AI services
- Calling Odoo
- Approval notifications

### Odoo

- Master/business data
- Partners, products, orders and accounting records
- Authoritative validation data
- Final business records
- Approval state where appropriate

### AI

- Classification
- OCR/vision interpretation
- Structured extraction
- Entity matching suggestions
- Never the final authority for accounting/business facts

### Rules

Rules must be deterministic and independently testable.

Examples:

- supplier exists
- invoice number is present
- invoice is not a duplicate
- PO exists
- PO and invoice totals are within configured tolerance
- company/currency/tax conditions are valid
- supplier bank-account changes always require human verification

### DMS

The original document and relevant metadata are archived. OCA DMS is the primary candidate for reuse.

### Audit

Record:

- source channel
- source message/document ID
- workflow ID
- n8n execution ID
- timestamps
- extracted payload
- validation results
- approval decisions
- Odoo record IDs
- errors and retries

## Idempotency

Vendor invoices should use a deterministic duplicate key such as:

`company + document_type + supplier_tax_id + invoice_number + invoice_date`

Document hashes and source-message IDs should also be retained where available.

## Security

- Least-privilege credentials
- Secure webhooks
- Upload validation
- Malicious-file controls
- Prompt-injection defense
- PII/KVKK controls
- Approval controls for high-risk actions
- No automatic supplier bank-account changes
- Full audit trail

## Proposed Odoo module

`intelligent_document`

Main model: `intelligent.document`

Suggested fields:

- name
- source_channel
- source_message_id
- document_type
- document_subtype
- partner_id
- company_id
- attachment_id
- ai_confidence
- validation_status
- approval_status
- odoo_model
- odoo_record_id
- workflow_execution_id
- risk_level
- error_message
- created_at
- processed_at

State machine:

`received → processing → classified → extracted → validated → approval_required → approved → completed`

Alternative terminal/error states:

`rejected`, `failed`, `manual_review`, `cancelled`

## External project reuse

The first candidates for inspection are:

- OCA DMS — likely reuse
- Apexive odoo-llm — evaluate selective reuse/reference
- invoice AI import modules — evaluate
- current n8n/Odoo integration options — evaluate before writing a custom node
- Odoo↔n8n bridge projects — evaluate
- WhatsApp↔n8n↔Odoo examples — evaluate

No external dependency should be adopted before checking Odoo 18 compatibility, license, security, maintenance status and architectural fit.
