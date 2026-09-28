# M0 — Reuse / Fork / Rewrite Assessment

## Purpose

This document records the initial assessment of existing open-source components before implementation. The project target is Odoo 18 + n8n.

## Initial matrix

| Component | Odoo 18 | Initial decision | Why |
|---|---|---|---|
| OCA DMS | Yes | **REUSE** | Mature document-management foundation; includes auto-classification modules. |
| OCA `dms_auto_classification` | Yes | **REUSE / EXTEND** | Useful for deterministic/document routing, but AI classification remains our orchestration concern. |
| OCA `dms_field_auto_classification` | Yes | **REUSE / EVALUATE** | Useful when documents need to be embedded in Odoo records. |
| Apexive `odoo-llm` core | Yes | **REUSE / CONTROLLED DEPENDENCY** | Provides provider abstraction, model management and security/tool framework. |
| Apexive `llm_tool_ocr_mistral` | Yes | **REUSE / OPTIONAL PROVIDER** | Existing PDF/image OCR through Mistral vision. Keep OCR provider replaceable. |
| Apexive `account_invoice_import_llm` | Yes | **REFERENCE + POSSIBLE REUSE** | Directly overlaps with MVP invoice extraction/import; must inspect implementation and dependency boundaries before adoption. |
| n8n | N/A | **CORE** | Main orchestration, routing, channel integration, retries and approvals. |
| Existing n8n/Odoo nodes | Varies | **EVALUATE** | Prefer maintained Odoo 18-compatible integration; do not depend on archived/older nodes without tests. |
| Custom Odoo module | Odoo 18 | **WRITE OURSELVES** | Own document state, audit, approval and integration contracts are project-specific. |
| Business rules | Odoo 18 + n8n | **WRITE OURSELVES** | Financial validation must be deterministic and independently testable. |
| Audit/idempotency | Odoo + n8n | **WRITE OURSELVES** | Core product requirement; must not depend on AI behaviour. |

## Evidence reviewed

### OCA DMS

The OCA DMS repository exposes an Odoo 18 branch with `dms`, `dms_auto_classification`, `dms_field`, `dms_field_auto_classification`, `dms_user_role`, `hr_dms_field` and related modules. The repository states that the repository is AGPL-3.0 but individual module licenses must be checked in each `__manifest__.py`. See the upstream repository before selecting a module for distribution.

Source: https://github.com/OCA/dms

### Apexive Odoo LLM

The Odoo 18 branch contains a core `llm` framework, assistants, tool framework, multiple providers, OCR tooling, knowledge/RAG and accounting tools. The repository explicitly lists `account_invoice_import_llm` and `llm_tool_ocr_mistral` among its Odoo 18 modules.

Source: https://github.com/apexive/odoo-llm

### Current maintenance caveat

`odoo-llm` has active development and open issues. This is not a reason to reject it, but production adoption should be pinned to a tested commit/release and covered by integration tests. In particular, the open issue list contains an invoice-import tax issue and an Odoo 18 Enterprise issue.

Source: https://github.com/apexive/odoo-llm/issues

## Architecture decision

We will **not** make the AI/OCR stack the centre of the product. The centre is the n8n workflow plus deterministic Odoo validation.

```text
Channel
  -> n8n
  -> document intake
  -> OCR / LLM provider
  -> normalized JSON
  -> Odoo lookup
  -> deterministic validation
  -> approval when required
  -> Odoo action
  -> DMS archive
  -> audit
```

## Rules for external dependencies

Before adding any dependency, verify:

1. Odoo 18 compatibility
2. License of the exact module
3. Runtime dependencies
4. Security model
5. Maintenance/activity
6. Test coverage
7. Upgrade/migration path
8. Whether the dependency can be isolated behind an adapter

## Important boundary

`account_invoice_import_llm` may solve part of the invoice problem, but we should not allow it to dictate the entire product architecture. The reusable abstraction is **document intake → extraction → validation → approval → ERP action**.
