# Project Plan

## Phase 0 — M0: Architecture & Research

Goal: decide what to reuse, fork, rewrite or avoid before coding.

### Deliverables

1. Dependency/reuse matrix
2. License compatibility matrix
3. Odoo 18 compatibility matrix
4. Security review notes
5. Final architecture
6. MVP technical specification

### Research targets

- OCA DMS
- OCA DMS auto-classification
- Apexive odoo-llm
- account invoice AI/import modules
- n8n Odoo nodes
- Odoo↔n8n bridges
- WhatsApp↔n8n↔Odoo examples

## Phase 1 — M1: Repository Foundation

- Finalize repository license
- Add contribution rules
- Add documentation structure
- Add CI skeleton
- Define coding/testing conventions
- Define versioning strategy

## Phase 2 — M2: Odoo Integration

Build the smallest possible integration:

```
n8n → Odoo 18 → search partner → JSON response
```

Requirements:

- authentication
- least privilege
- error normalization
- timeout handling
- connection tests

## Phase 3 — M3: Document Intake

```
Email → n8n → PDF/image → intelligent.document
```

Requirements:

- file validation
- source metadata
- idempotency
- safe attachment handling
- processing state

## Phase 4 — M4: AI Processing

```
Document → OCR/Vision → Classification → Structured JSON
```

Requirements:

- versioned schemas
- provider abstraction
- confidence score
- malformed-output handling
- prompt-injection defenses
- no direct AI→Odoo write path

## Phase 5 — M5: Validation

Validate against Odoo:

- partner
- company
- currency
- PO
- invoice number
- totals
- lines
- tax
- duplicate status

Separate AI confidence from business validation.

## Phase 6 — M6: Approval

Create approval records with:

- document
- risk
- reason
- proposed action
- requester
- approver
- timestamps
- decision

High-risk actions must not bypass approval.

## Phase 7 — M7: Odoo Vendor Bill

After successful validation/approval:

- create draft vendor bill
- attach original document
- link intelligent.document
- write audit event
- guarantee idempotency

## Phase 8 — M8: DMS + Audit

- archive original file
- retain metadata
- record every workflow transition
- support manual retry
- support failed/dead-letter cases

## Phase 9 — M9: Test & Hardening

- unit tests
- integration tests
- workflow tests
- duplicate tests
- malformed document tests
- security tests
- approval bypass tests
- retry/idempotency tests
- performance baseline

## Phase 10 — M10: Additional Scenarios

Expand only after the invoice flow is stable:

1. expense invoice
2. payment receipt
3. sales order
4. quote request
5. delivery note
6. WhatsApp
7. voice
8. legal/contract documents

## Definition of Done for v0.1

A supplier invoice arriving by email can be:

1. received safely
2. stored
3. classified
4. OCR'd/extracted
5. matched to an Odoo partner
6. matched to a PO when available
7. validated deterministically
8. routed to approval when required
9. converted to a draft vendor bill
10. archived in DMS
11. fully audited
12. safely retried without creating duplicates
