# M0 — account_invoice_import_llm Analizi

## Kaynak

Apexive `odoo-llm`, `account_invoice_import_llm` modülü, Odoo 18.

## Modülün zaten çözdükleri

Modül, ilk MVP için yararlı bir referans uygulama sunuyor:

- PDF/görüntü faturadan veri çıkarma
- Mistral OCR
- yapılandırılmış fatura verisi çıkarma
- tedarikçi adı/VKN çıkarma
- fatura numarası ve tarihleri
- para birimi
- ara toplam/vergi/toplam
- fatura satırları
- OCA `account_invoice_import` entegrasyonu
- taslak faturanın elle işlenmesi
- gömülü XML yoksa yedek ayrıştırma

Modül açık, yapılandırılmış bir şema kullanıyor ve çıkarılan sonucu OCA Invoice Pivot Format'a dönüştürüyor.

## Önemli mimari gözlem

Mevcut modül **Odoo merkezli**: belge önce bir Odoo fatura/içe aktarma sihirbazına ulaşıyor, ardından OCR ve LLM veri çıkarma Odoo içinde gerçekleşiyor.

Bizim projemiz bilinçli olarak **n8n merkezli**:

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

(E-posta / WhatsApp / Web → n8n → belge alımı → OCR / AI → normalleştirilmiş JSON → Odoo araması + deterministik doğrulama → gerektiğinde onay → Odoo kaydı)

Bu nedenle modülü projemize olduğu gibi kopyalamamalıyız.

## Yeniden kullanım kararı

### REFERANS OLARAK YENİDEN KULLAN

Şu kavramları yeniden kullan:

1. Yapılandırılmış fatura JSON şeması
2. OCR → yapılandırılmış veri çıkarma akışı
3. OCA Invoice Pivot Format eşlemesi
4. Sağlayıcı soyutlaması
5. Doğrudan belge işaretlemesinden OCR metni + LLM'e yedek geçiş
6. Açık doğrulama/hata yönetimi

### OLASI DOĞRUDAN BAĞIMLILIK

Normal OCA fatura içe aktarma iş akışının istendiği bir Odoo kurulumunda `account_invoice_import_llm`'i doğrudan kullanmayı değerlendir.

n8nOdoo platformu için n8n iş akışını bir Odoo arayüz sihirbazına bağımlı kılmak yerine bu bağımlılığı isteğe bağlı tut.

### KOPYALAMA

Odoo merkezli işleme akışının tamamını n8nOdoo'ya kopyalama. Özellikle platform, kullanıcıların önce taslak fatura oluşturup sonra "Process with AI"a tıklamasını gerektirmemeli.

## Önerilen n8nOdoo uyarlaması

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

1. E-postayla PDF gelir.
2. n8n `intelligent.document` oluşturur.
3. n8n kaynak meta verisini/orijinal dosyayı saklar.
4. OCR sağlayıcısı baytları işler.
5. LLM normalleştirilmiş fatura JSON'u döndürür.
6. n8n JSON şemasını doğrular.
7. Odoo araması aday tedarikçiyi bulur.
8. Deterministik kurallar tedarikçi/sipariş/tutar/vergi/para birimini doğrular.
9. Risk motoru otomatik mi onaylı mı olacağına karar verir.
10. Odoo taslak tedarikçi faturası yalnızca doğrulama/onaydan sonra oluşturulur.
11. Orijinal PDF DMS'te arşivlenir.
12. Denetim olayı kaydedilir.

## Projemiz için temel tasarım iyileştirmesi

Mevcut modül AI çıktısını OCA Invoice Pivot Format'a eşliyor. Bunun yerine ara, sağlayıcıdan bağımsız bir sözleşme getirmeliyiz:

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

Bu sözleşmeyi Odoo/OCA'ya özgü yapılara yalnızca Odoo adaptörü dönüştürmeli.

## Kritik doğrulama kuralları

AI ile veri çıkarma, iş doğrulaması değildir.

En azından:

- tedarikçi VKN'si/adı Odoo'ya karşı eşleştirilmeli
- fatura numarası mükerrerlik açısından kontrol edilmeli
- şirket, alım bağlamından belirlenmeli
- para birimi doğrulanmalı
- varsa satın alma siparişi eşleştirilmeli
- miktarlar ve fiyatlar siparişle karşılaştırılmalı
- vergi oranları Odoo'da yapılandırılmış vergilere karşı doğrulanmalı
- toplamlar yeniden hesaplanıp çıkarılan değerlerle karşılaştırılmalı
- şüpheli tedarikçi banka hesabı değişiklikleri insan doğrulaması gerektirir

## Yedek strateji

Apexive uygulamasında yararlı iki seviyeli bir strateji var:

```text
Direct document annotation
        ↓ failure/unavailable
OCR text
        ↓
LLM structured extraction
```

(Doğrudan belge işaretleme → hata/yoksa → OCR metni → LLM ile yapılandırılmış veri çıkarma)

Bu kavramı iş akışı seviyesinde korumalıyız.

## MVP sonucu

`account_invoice_import_llm` yapmamız gereken fatura-OCR araştırmasını önemli ölçüde azaltıyor. **Referans uygulama ve isteğe bağlı Odoo tarafı bileşen** olarak ele alınmalı; alım, yönlendirme, doğrulama, onay, idempotency ve denetim orkestrasyonu n8nOdoo'ya aittir.

Sonraki M0 görevi: normalleştirilmiş `invoice.schema.json`'u tanımlamadan önce altta yatan OCA `account_invoice_import` sözleşmesini ve Apexive adaptörünün kullandığı Odoo model/alanlarını incele.
