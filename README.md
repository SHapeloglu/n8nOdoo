# n8nOdoo

Odoo 18 için n8n ile orkestre edilen, açık kaynaklı akıllı belge ve iş süreci otomasyon platformu.

> **AI okur. Odoo bilir. Kurallar doğrular. Gerektiğinde insan onaylar. n8n orkestre eder. DMS saklar. Denetim kaydı kanıtlar.**

## Vizyon

E-posta, WhatsApp, Web ve API girdilerini güvenli, denetlenebilir bir iş akışı motoruyla Odoo'ya bağlamak. AI sınıflandırma, OCR ve yapılandırılmış veri çıkarma için kullanılır; belirleyici (deterministik) kurallar ve Odoo ana verileri yetkili kaynak olarak kalır.

## İlk üretim dilimi

**E-posta → Tedarikçi Faturası → OCR/AI → Odoo İş Ortağı/Satın Alma Siparişi doğrulaması → Onay → Taslak Tedarikçi Faturası → DMS → Denetim**

## İlkeler

- n8n orkestratördür; muhasebe/iş kuralı otoritesi değildir.
- Odoo; iş ortakları, ürünler, siparişler, faturalar ve şirket verileri için tek doğruluk kaynağıdır.
- AI veri çıkarır ve yorumlar; iş gerçeği uyduramaz.
- Doğrulama kuralları deterministiktir ve bağımsız olarak test edilebilir.
- Yüksek riskli işlemler insan onayı gerektirir.
- Orijinal belgeler DMS'te saklanır.
- Her işleme adımı denetlenebilir ve idempotenttir.
- Belge içeriği asla sistem talimatı olarak ele alınmaz.

## Planlanan bileşenler

- Odoo 18 özel modülü: `intelligent_document`
- n8n iş akışları ve yeniden kullanılabilir alt iş akışları
- Belge saklama/sınıflandırma için OCA DMS
- Takılıp çıkarılabilir OCR/LLM sağlayıcıları
- Normalleştirilmiş belge verisi için JSON şemaları
- Kural ve onay motoru
- Denetim ve idempotency katmanı

## Durum

Depo mimari ve araştırmayla başlıyor. Uygulama, M0'daki yeniden kullan/fork et/yeniden yaz değerlendirmesinden sonra başlar.

## Lisans

Lisans, bağımlılık/lisans uyumluluk incelemesinden sonra ilk uygulama sürümünden önce kesinleştirilecek.
