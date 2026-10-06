# Mimari

## Sistem akışı

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

(E-posta / WhatsApp / Web / API → n8n → belge alımı → OCR / görüntü → AI sınıflandırma → yapılandırılmış JSON → Odoo araması (iş ortağı + satın alma siparişi) → deterministik kurallar → otomatik veya onaylı → Odoo → DMS ve denetim kaydı)

## Sorumluluk sınırları

### n8n

- Kanal entegrasyonu
- İş akışı orkestrasyonu
- Yeniden deneme/zaman aşımı yönetimi
- Yönlendirme
- OCR/AI servislerini çağırma
- Odoo'yu çağırma
- Onay bildirimleri

### Odoo

- Ana/iş verileri
- İş ortakları, ürünler, siparişler ve muhasebe kayıtları
- Yetkili doğrulama verisi
- Nihai iş kayıtları
- Uygun olduğunda onay durumu

### AI

- Sınıflandırma
- OCR/görüntü yorumlama
- Yapılandırılmış veri çıkarma
- Varlık eşleştirme önerileri
- Muhasebe/iş gerçekleri için asla nihai otorite değildir

### Kurallar

Kurallar deterministik ve bağımsız olarak test edilebilir olmalıdır.

Örnekler:

- tedarikçi mevcut
- fatura numarası var
- fatura mükerrer değil
- satın alma siparişi mevcut
- sipariş ve fatura toplamları yapılandırılan tolerans içinde
- şirket/para birimi/vergi koşulları geçerli
- tedarikçi banka hesabı değişiklikleri her zaman insan doğrulaması gerektirir

### DMS

Orijinal belge ve ilgili meta veriler arşivlenir. Yeniden kullanım için birincil aday OCA DMS'tir.

### Denetim

Kaydedilecekler:

- kaynak kanal
- kaynak mesaj/belge kimliği
- iş akışı kimliği
- n8n çalıştırma kimliği
- zaman damgaları
- çıkarılan veri
- doğrulama sonuçları
- onay kararları
- Odoo kayıt kimlikleri
- hatalar ve yeniden denemeler

## Idempotency

Tedarikçi faturaları aşağıdaki gibi deterministik bir mükerrer anahtarı kullanmalı:

`company + document_type + supplier_tax_id + invoice_number + invoice_date`

Mümkün olduğunda belge hash'leri ve kaynak mesaj kimlikleri de saklanmalı.

## Güvenlik

- En az yetkili kimlik bilgileri
- Güvenli webhook'lar
- Yükleme doğrulaması
- Zararlı dosya kontrolleri
- Prompt-injection savunması
- Kişisel veri/KVKK kontrolleri
- Yüksek riskli işlemler için onay kontrolleri
- Tedarikçi banka hesabında otomatik değişiklik yok
- Tam denetim kaydı

## Önerilen Odoo modülü

`intelligent_document`

Ana model: `intelligent.document`

Önerilen alanlar:

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

Durum makinesi:

`received → processing → classified → extracted → validated → approval_required → approved → completed`

Alternatif son/hata durumları:

`rejected`, `failed`, `manual_review`, `cancelled`

## Harici proje yeniden kullanımı

İncelenecek ilk adaylar:

- OCA DMS — büyük olasılıkla yeniden kullanılacak
- Apexive odoo-llm — seçici yeniden kullanım/referans olarak değerlendir
- fatura AI içe aktarma modülleri — değerlendir
- güncel n8n/Odoo entegrasyon seçenekleri — özel node yazmadan önce değerlendir
- Odoo↔n8n köprü projeleri — değerlendir
- WhatsApp↔n8n↔Odoo örnekleri — değerlendir

Odoo 18 uyumluluğu, lisans, güvenlik, bakım durumu ve mimari uyum kontrol edilmeden hiçbir harici bağımlılık benimsenmemeli.
