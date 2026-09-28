# Tasks

## M0 — Research

### Reuse / dependency analysis

- [ ] Inspect OCA DMS architecture
- [ ] Inspect `dms_auto_classification`
- [ ] Inspect Apexive `odoo-llm` architecture
- [ ] Inspect `account_invoice_import_llm`
- [ ] Inspect current n8n Odoo integration options
- [ ] Inspect Odoo↔n8n bridge projects
- [ ] Inspect WhatsApp↔n8n↔Odoo examples
- [ ] Record Odoo 18 compatibility
- [ ] Record license for every candidate
- [ ] Record maintenance/activity
- [ ] Record security considerations
- [ ] Decide: reuse / fork / rewrite / avoid

### Architecture

- [ ] Freeze component boundaries
- [ ] Define normalized document JSON
- [ ] Define invoice JSON schema
- [ ] Define validation contract
- [ ] Define approval contract
- [ ] Define audit event contract
- [ ] Define idempotency strategy
- [ ] Define error/dead-letter strategy

## M1 — Repository

- [ ] Finalize LICENSE
- [ ] Add CONTRIBUTING.md
- [ ] Add CI
- [ ] Add docs directories
- [ ] Add issue templates
- [ ] Add security policy

## M2 — Odoo

- [ ] Create `intelligent_document` module
- [ ] Create `intelligent.document`
- [ ] Create states
- [ ] Create security/access rules
- [ ] Create attachment relation
- [ ] Create workflow execution fields
- [ ] Create audit model
- [ ] Create approval model
- [ ] Add tests

## M3 — n8n

- [ ] Create email intake workflow
- [ ] Create file validation subworkflow
- [ ] Create Odoo lookup subworkflow
- [ ] Create error handler
- [ ] Create retry path
- [ ] Create idempotency check

## M4 — AI/OCR

- [ ] Select initial OCR provider
- [ ] Select initial LLM provider
- [ ] Define classification prompt
- [ ] Define invoice extraction schema
- [ ] Validate JSON schema
- [ ] Handle low confidence
- [ ] Add prompt-injection safeguards

## M5 — Invoice Rules

- [ ] Partner matching
- [ ] Duplicate detection
- [ ] PO matching
- [ ] Line matching
- [ ] Total tolerance
- [ ] Currency validation
- [ ] Tax validation
- [ ] Company validation
- [ ] Bank-account-change high-risk rule

## M6 — Approval

- [ ] Approval request creation
- [ ] Approver routing
- [ ] Approval notification
- [ ] Approval callback
- [ ] Rejection path
- [ ] Expiration/escalation
- [ ] Audit approval decision

## M7 — Vendor Bill

- [ ] Create draft bill
- [ ] Attach original document
- [ ] Link source record
- [ ] Prevent duplicate creation
- [ ] Record audit event

## M8 — DMS

- [ ] Install/integrate OCA DMS
- [ ] Define workspace/folder strategy
- [ ] Define metadata
- [ ] Archive original document
- [ ] Link DMS file to Odoo document

## M9 — Quality

- [ ] Unit tests
- [ ] Integration tests
- [ ] End-to-end invoice test
- [ ] Duplicate invoice test
- [ ] Unreadable document test
- [ ] Wrong supplier test
- [ ] PO mismatch test
- [ ] Approval bypass test
- [ ] Retry/idempotency test
- [ ] Security test
- [ ] Performance baseline

## Future

- [ ] Expense documents
- [ ] Payment receipts
- [ ] Sales orders
- [ ] Quote requests
- [ ] Delivery notes
- [ ] WhatsApp
- [ ] Voice
- [ ] Contracts
- [ ] Legal/official documents
