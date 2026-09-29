# M0 — Partner & Document Routing Design

## Purpose

Define how n8nOdoo decides what an incoming document means and which Odoo partner/process it belongs to without allowing AI to make unsupported accounting decisions.

## Core principle

```text
AI interpretation
      +
Odoo master data
      +
transaction history
      +
 deterministic rules
      ↓
validated routing decision
```

AI produces candidates and extracted facts. Odoo data and deterministic rules validate them.

## Step 1 — Identify receiving company

The company must be determined from trusted context before creating an accounting record.

Possible sources, in priority order:

1. Dedicated mailbox/channel configuration
2. n8n workflow configuration
3. Explicit document/company identifier
4. Manual review if ambiguous

Do not infer company solely from an LLM guess when multiple companies are possible.

## Step 2 — Extract partner identifiers

Preferred identifiers:

1. Turkish VKN/TCKN or applicable tax identifier
2. E-invoice/e-document identifiers where available
3. IBAN when relevant
4. Email/domain
5. Phone
6. Normalized legal name
7. Address

The system should preserve the original extracted value and the normalized value.

## Step 3 — Search Odoo partners

Candidate matching should query Odoo using progressively weaker signals.

```text
Exact tax ID
    ↓ no match
Exact external/e-invoice identifier
    ↓ no match
Known bank account / IBAN
    ↓ no match
Normalized legal name + company context
    ↓ no match
Email/domain/phone
    ↓
Manual review
```

A weak name match must never silently override an exact conflicting tax identifier.

## Customer vs supplier

A partner can have both roles. Do not use a simple customer/supplier binary classification.

Evaluate:

- supplier rank / vendor status
- customer rank / customer status
- existing vendor bills
- existing customer invoices
- purchase orders
- sales orders
- recent transaction history
- document direction and channel

Example:

```text
Partner exists as customer + supplier
            ↓
Incoming invoice
            ↓
Look for purchase-side evidence
            ↓
PO / vendor history / supplier configuration
            ↓
Route to vendor invoice flow
```

## Document classification

Initial document classes:

- vendor_invoice
- expense_invoice
- sales_order
- quote_request
- payment_receipt
- bank_statement
- delivery_note
- return_document
- contract
- legal_document
- official_letter
- customer_request
- support_request
- unknown

Classification should return both a type and confidence, but confidence alone does not authorize an accounting action.

## Purchase vs expense invoice

A vendor invoice is not automatically a purchase-order invoice.

### Purchase-side evidence

Look for:

- matching open/confirmed PO
- matching supplier
- matching product/service lines
- matching quantities
- matching prices within tolerance
- expected purchase taxes
- warehouse/purchase context

### Expense-side evidence

Possible indicators:

- no relevant PO
- recurring utility/telecom/rent/service supplier
- supplier configured for expense categories
- expense account/category history
- document explicitly describing a general operating expense

If both routes remain plausible, create a manual-review task rather than guessing.

## Purchase Order matching

Candidate PO scoring can use:

- partner exact match
- company exact match
- currency match
- product/service overlap
- quantity compatibility
- price compatibility
- date proximity
- PO state

Example conceptual score:

```text
partner exact       +40
company exact       +20
currency match      +10
product overlap     +15
quantity compatible +5
price compatible    +10
-------------------------
maximum             100
```

These are design weights, not production thresholds. Thresholds must be configurable and tested against real documents.

## Invoice validation

Before creating a vendor bill:

- supplier match is sufficiently strong
- company is known
- invoice number exists where required
- duplicate search completed
- currency is valid
- tax configuration is valid
- line totals reconcile
- subtotal + tax = total within configured tolerance
- PO match is either valid or explicitly not required
- high-risk conditions are clear

## Duplicate detection

Primary candidates:

```text
company + supplier + supplier_invoice_number
```

Secondary signals:

- invoice date
- total
- currency
- document hash
- source message ID
- attachment hash

Duplicates should be idempotently ignored or routed to review, never silently posted twice.

## Risk rules

Always require human verification for at least:

- supplier bank-account change
- conflicting tax identifiers
- ambiguous company
- ambiguous partner
- duplicate suspicion
- unusually high-value invoice according to company policy
- PO/Invoice mismatch above configured tolerance
- unsupported or unreadable document

## Decision object

The routing layer should produce a normalized decision such as:

```json
{
  "document_type": "vendor_invoice",
  "company_id": 1,
  "partner_candidate_id": 42,
  "partner_match": {
    "method": "tax_id",
    "confidence": 1.0,
    "validated": true
  },
  "purchase_context": {
    "po_candidate_id": 381,
    "match_status": "matched"
  },
  "route": "vendor_bill",
  "approval_required": false,
  "risk_level": "low",
  "reasons": [
    "Exact supplier tax ID match",
    "Matching purchase order",
    "Invoice totals within tolerance"
  ]
}
```

This object is a proposal for workflow routing, not an accounting record.

## Manual review

Manual review must show:

- original document
- extracted fields
- candidate Odoo partner(s)
- relevant PO/SO candidates
- validation failures
- risk reasons
- proposed action

The reviewer can approve, reject or correct the proposed routing.

## Implementation boundary

### n8n

- receive/extract
- call AI/OCR
- call Odoo lookup APIs
- combine candidate data
- execute workflow routing
- create approval request
- retry/error handling

### Odoo module

- expose safe lookup methods where standard API calls are insufficient
- hold normalized document, decision, approval and audit records
- create/update final business records
- enforce server-side validation for critical operations

### AI

- classify
- extract
- suggest candidate matches
- explain extraction uncertainty

AI must not directly create/post accounting entries.

## Next implementation tasks

- [ ] Define partner candidate API contract
- [ ] Define normalized company context
- [ ] Define partner matching normalization functions
- [ ] Define PO candidate API
- [ ] Define duplicate-check API
- [ ] Define purchase-vs-expense rule configuration
- [ ] Define risk policy configuration
- [ ] Create routing decision JSON schema
- [ ] Build test cases for customer+supplier dual-role partners
- [ ] Build test cases for no-PO expense invoices
- [ ] Build test cases for ambiguous partner matches
