# M1 — Odoo Partner Matching API Design

## Goal

Define the smallest Odoo integration needed for n8n to search and identify a partner before any document is posted.

## Principle

The first integration is deliberately read-only.

```text
n8n
 ↓
Odoo 18
 ↓
partner lookup
 ↓
normalized candidates
 ↓
n8n rules
```

No invoice, payment, PO or partner mutation is performed in this milestone.

## Candidate search order

Use the strongest identifiers first:

1. Exact VAT/VKN/TCKN when present
2. Exact commercial/company registration identifier when configured
3. Exact normalized email/domain when appropriate
4. Exact normalized phone when appropriate
5. Exact partner reference (`ref`) when supplied
6. Name search as a weaker candidate search

Name-only matches must not be treated as authoritative identity.

## Odoo partner fields

Initial fields to retrieve:

- `id`
- `name`
- `vat`
- `company_type`
- `is_company`
- `customer_rank`
- `supplier_rank`
- `email`
- `phone`
- `mobile`
- `ref`
- `parent_id`
- `commercial_partner_id`
- `active`
- `company_id`

Additional fields should be requested only when a concrete rule requires them.

## Normalized response

n8n should not depend on the complete Odoo partner record. The integration should normalize it:

```json
{
  "query": {
    "vat": "...",
    "name": "...",
    "email": "...",
    "phone": "..."
  },
  "candidates": [
    {
      "partner_id": 123,
      "name": "Example Ltd.",
      "vat": "...",
      "is_customer": true,
      "is_supplier": true,
      "commercial_partner_id": 123,
      "company_id": 1,
      "match": {
        "method": "vat_exact",
        "confidence": 1.0
      }
    }
  ],
  "status": "matched"
}
```

Possible statuses:

- `matched`
- `multiple_candidates`
- `not_found`
- `invalid_query`
- `odoo_error`

## Match semantics

`confidence` is a matching score, not an AI confidence score.

Examples:

```text
VAT exact                 1.00
Partner reference exact  1.00
Email exact               0.90
Phone exact               0.85
Name exact                0.70
Name fuzzy                < 0.70
```

These values are initial design values only. Production thresholds must be tested against real company data.

## Customer vs supplier

Do not treat `customer_rank > 0` and `supplier_rank > 0` as mutually exclusive.

A partner can be both.

The route is determined later using:

- incoming document type
- partner role
- open purchase orders
- open sales orders
- historical transactions
- company context
- deterministic business rules.

## Company context

Every lookup must include the receiving Odoo company context whenever the deployment is multi-company.

Do not assume that a partner found in Odoo automatically belongs to the receiving company.

## Security

The M1 integration should use a least-privilege Odoo integration user.

Read access required initially:

- `res.partner`

No write/create/unlink access is needed for M1 partner lookup.

Credentials must stay in n8n's credential store and must never appear in workflow JSON, logs or prompts.

## n8n sub-workflow

Proposed reusable workflow:

```text
SUB — Odoo Partner Lookup
Input:
  vat?
  name?
  email?
  phone?
  ref?
  company_id?

        ↓
Validate query
        ↓
VAT lookup
        ↓ no result
Reference lookup
        ↓ no result
Email/phone lookup
        ↓ no result
Name candidate lookup
        ↓
Normalize candidates
        ↓
Return status + candidates
```

## Next implementation step

Build the read-only `n8n → Odoo → res.partner` proof of concept.

Acceptance test:

```text
Given a known supplier VAT number
When n8n calls the partner lookup
Then exactly the expected Odoo partner is returned
And no Odoo data is modified.
```

Then add tests for:

- customer-only partner
- supplier-only partner
- customer + supplier partner
- duplicate/ambiguous name
- missing VAT
- inactive partner
- multi-company partner
- Odoo authentication failure
- Odoo timeout
