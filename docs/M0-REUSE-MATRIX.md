# M0 — Yeniden Kullan / Fork Et / Yeniden Yaz Değerlendirmesi

## Amaç

Bu belge, uygulamaya geçmeden önce mevcut açık kaynak bileşenlerin ilk değerlendirmesini kaydeder. Proje hedefi Odoo 18 + n8n'dir.

## İlk matris

| Bileşen | Odoo 18 | İlk karar | Neden |
|---|---|---|---|
| OCA DMS | Evet | **YENİDEN KULLAN** | Olgun belge yönetimi temeli; otomatik sınıflandırma modüllerini içerir. |
| OCA `dms_auto_classification` | Evet | **YENİDEN KULLAN / GENİŞLET** | Deterministik belge yönlendirme için yararlı; ancak AI sınıflandırma bizim orkestrasyon sorumluluğumuzda kalır. |
| OCA `dms_field_auto_classification` | Evet | **YENİDEN KULLAN / DEĞERLENDİR** | Belgelerin Odoo kayıtlarına gömülmesi gerektiğinde yararlı. |
| Apexive `odoo-llm` çekirdeği | Evet | **YENİDEN KULLAN / KONTROLLÜ BAĞIMLILIK** | Sağlayıcı soyutlaması, model yönetimi ve güvenlik/araç çerçevesi sağlar. |
| Apexive `llm_tool_ocr_mistral` | Evet | **YENİDEN KULLAN / İSTEĞE BAĞLI SAĞLAYICI** | Mistral vision ile mevcut PDF/görüntü OCR'ı. OCR sağlayıcısı değiştirilebilir kalmalı. |
| Apexive `account_invoice_import_llm` | Evet | **REFERANS + OLASI YENİDEN KULLANIM** | MVP fatura veri çıkarma/içe aktarmayla doğrudan örtüşüyor; benimsemeden önce uygulaması ve bağımlılık sınırları incelenmeli. |
| n8n | — | **ÇEKİRDEK** | Ana orkestrasyon, yönlendirme, kanal entegrasyonu, yeniden denemeler ve onaylar. |
| Mevcut n8n/Odoo node'ları | Değişken | **DEĞERLENDİR** | Bakımı süren, Odoo 18 uyumlu entegrasyonu tercih et; arşivlenmiş/eski node'lara test olmadan bağımlı olma. |
| Özel Odoo modülü | Odoo 18 | **KENDİMİZ YAZACAĞIZ** | Belge durumu, denetim, onay ve entegrasyon sözleşmeleri projeye özgü. |
| İş kuralları | Odoo 18 + n8n | **KENDİMİZ YAZACAĞIZ** | Finansal doğrulama deterministik ve bağımsız test edilebilir olmalı. |
| Denetim/idempotency | Odoo + n8n | **KENDİMİZ YAZACAĞIZ** | Çekirdek ürün gereksinimi; AI davranışına bağlı olmamalı. |

## İncelenen kanıtlar

### OCA DMS

OCA DMS deposu `dms`, `dms_auto_classification`, `dms_field`, `dms_field_auto_classification`, `dms_user_role`, `hr_dms_field` ve ilgili modülleri içeren bir Odoo 18 dalı sunuyor. Depo AGPL-3.0 olduğunu belirtiyor, ancak tek tek modül lisansları her `__manifest__.py` içinde kontrol edilmeli. Dağıtım için modül seçmeden önce upstream depoya bak.

Kaynak: https://github.com/OCA/dms

### Apexive Odoo LLM

Odoo 18 dalı çekirdek bir `llm` çerçevesi, asistanlar, araç çerçevesi, birden fazla sağlayıcı, OCR araçları, bilgi/RAG ve muhasebe araçları içeriyor. Depo, Odoo 18 modülleri arasında `account_invoice_import_llm` ve `llm_tool_ocr_mistral`'ı açıkça listeliyor.

Kaynak: https://github.com/apexive/odoo-llm

### Güncel bakım uyarısı

`odoo-llm` aktif geliştiriliyor ve açık issue'ları var. Bu reddetmek için bir neden değil, ancak üretimde benimsenirken test edilmiş bir commit/sürüme sabitlenmeli ve entegrasyon testleriyle kapsanmalı. Özellikle açık issue listesinde bir fatura içe aktarma vergi sorunu ve bir Odoo 18 Enterprise sorunu var.

Kaynak: https://github.com/apexive/odoo-llm/issues

## Mimari karar

AI/OCR yığınını ürünün merkezi **yapmayacağız**. Merkez, n8n iş akışı ile deterministik Odoo doğrulamasıdır.

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

(Kanal → n8n → belge alımı → OCR / LLM sağlayıcısı → normalleştirilmiş JSON → Odoo araması → deterministik doğrulama → gerektiğinde onay → Odoo işlemi → DMS arşivi → denetim)

## Harici bağımlılık kuralları

Herhangi bir bağımlılık eklemeden önce doğrula:

1. Odoo 18 uyumluluğu
2. Tam olarak o modülün lisansı
3. Çalışma zamanı bağımlılıkları
4. Güvenlik modeli
5. Bakım/aktivite
6. Test kapsamı
7. Yükseltme/taşıma yolu
8. Bağımlılığın bir adaptör arkasında yalıtılıp yalıtılamayacağı

## Önemli sınır

`account_invoice_import_llm` fatura sorununun bir kısmını çözebilir, ama tüm ürün mimarisini belirlemesine izin vermemeliyiz. Yeniden kullanılabilir soyutlama şudur: **belge alımı → veri çıkarma → doğrulama → onay → ERP işlemi**.
