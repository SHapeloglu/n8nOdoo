# BACKLOG.md — n8nOdoo

Planlanmamış fikirler. Planlı kapsam `ROADMAP.md` (v0.1 tedarikçi faturaları → v0.2 finans belgeleri → v0.3 satış belgeleri → v0.4 WhatsApp) ve `TASKS.md` içinde.

- Girdi kanalı olarak Türk e-Fatura UBL-TR XML'i: tedarikçi e-Fatura gönderdiğinde OCR yerine yapılandırılmış XML'i doğrudan ayrıştır (`l10n_tr_sovos_efatura` bilgisini yeniden kullan).
- Yerel/kendi sunucusunda OCR + LLM sağlayıcı seçeneği (veri gizliliği), örn. `trocr-faz1`'deki PaddleOCR / TrOCR deneyimi, Ollama ile yerel LLM.
- Güven skoruna dayalı yönlendirme: yalnızca eşiğin üstündekileri otomatik işle, gerisini onay kuyruğuna gönder.
- v0.4 için Odoo sunucusunda zaten bulunan WhatsApp gateway modüllerini (`mail_gateway_whatsapp`, `wa_erp_bot`) yeniden kullan.
- Metrik paneli: kanal başına belge sayısı, otomatik / onaylı oranı, veri çıkarma hata oranı.
