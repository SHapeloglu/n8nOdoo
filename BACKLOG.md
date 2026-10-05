# BACKLOG.md — n8nOdoo

Unscheduled ideas. Planned scope lives in `ROADMAP.md` (v0.1 vendor invoices → v0.2 finance documents → v0.3 sales documents → v0.4 WhatsApp) and `TASKS.md`.

- Turkish e-Fatura UBL-TR XML as an input channel: parse structured XML directly instead of OCR when the supplier sends e-Fatura (reuse knowledge from `l10n_tr_sovos_efatura`).
- Local/self-hosted OCR + LLM provider option (data privacy), e.g. PaddleOCR / TrOCR experience from `trocr-faz1`, local LLM via Ollama.
- Confidence-based routing: auto-post only above a threshold, otherwise approval queue.
- Re-use the WhatsApp gateway modules already on the Odoo server (`mail_gateway_whatsapp`, `wa_erp_bot`) for v0.4.
- Metrics dashboard: documents per channel, auto vs approved ratio, extraction error rate.
