# Proje Planı

## Aşama 0 — M0: Mimari ve Araştırma

Hedef: kodlamadan önce neyin yeniden kullanılacağına, fork edileceğine, yeniden yazılacağına veya kaçınılacağına karar vermek.

### Çıktılar

1. Bağımlılık/yeniden kullanım matrisi
2. Lisans uyumluluk matrisi
3. Odoo 18 uyumluluk matrisi
4. Güvenlik inceleme notları
5. Nihai mimari
6. MVP teknik şartnamesi

### Araştırma hedefleri

- OCA DMS
- OCA DMS otomatik sınıflandırma
- Apexive odoo-llm
- fatura AI/içe aktarma modülleri
- n8n Odoo node'ları
- Odoo↔n8n köprüleri
- WhatsApp↔n8n↔Odoo örnekleri

## Aşama 1 — M1: Depo Temeli

- Depo lisansını kesinleştir
- Katkı kurallarını ekle
- Belge yapısını ekle
- CI iskeletini ekle
- Kodlama/test kurallarını tanımla
- Sürümleme stratejisini tanımla

## Aşama 2 — M2: Odoo Entegrasyonu

Mümkün olan en küçük entegrasyonu kur:

```
n8n → Odoo 18 → search partner → JSON response
```

Gereksinimler:

- kimlik doğrulama
- en az yetki
- hata normalleştirme
- zaman aşımı yönetimi
- bağlantı testleri

## Aşama 3 — M3: Belge Alımı

```
Email → n8n → PDF/image → intelligent.document
```

Gereksinimler:

- dosya doğrulama
- kaynak meta verisi
- idempotency
- güvenli ek işleme
- işleme durumu

## Aşama 4 — M4: AI İşleme

```
Document → OCR/Vision → Classification → Structured JSON
```

Gereksinimler:

- sürümlü şemalar
- sağlayıcı soyutlaması
- güven skoru
- bozuk çıktı yönetimi
- prompt-injection savunmaları
- AI'dan Odoo'ya doğrudan yazma yolu yok

## Aşama 5 — M5: Doğrulama

Odoo'ya karşı doğrula:

- iş ortağı
- şirket
- para birimi
- satın alma siparişi
- fatura numarası
- toplamlar
- satırlar
- vergi
- mükerrerlik durumu

AI güven skorunu iş doğrulamasından ayır.

## Aşama 6 — M6: Onay

Şunları içeren onay kayıtları oluştur:

- belge
- risk
- gerekçe
- önerilen işlem
- talep eden
- onaylayan
- zaman damgaları
- karar

Yüksek riskli işlemler onayı atlamamalı.

## Aşama 7 — M7: Odoo Tedarikçi Faturası

Başarılı doğrulama/onaydan sonra:

- taslak tedarikçi faturası oluştur
- orijinal belgeyi ekle
- intelligent.document ile bağla
- denetim olayı yaz
- idempotency'yi garanti et

## Aşama 8 — M8: DMS + Denetim

- orijinal dosyayı arşivle
- meta veriyi sakla
- her iş akışı geçişini kaydet
- elle yeniden denemeyi destekle
- başarısız/dead-letter durumlarını destekle

## Aşama 9 — M9: Test ve Sıkılaştırma

- birim testleri
- entegrasyon testleri
- iş akışı testleri
- mükerrer kayıt testleri
- bozuk belge testleri
- güvenlik testleri
- onay atlatma testleri
- yeniden deneme/idempotency testleri
- performans referans ölçümü

## Aşama 10 — M10: Ek Senaryolar

Yalnızca fatura akışı kararlı hale geldikten sonra genişlet:

1. gider faturası
2. ödeme dekontu
3. satış siparişi
4. teklif talebi
5. irsaliye
6. WhatsApp
7. ses
8. hukuki/sözleşme belgeleri

## v0.1 için Tamamlanma Tanımı

E-postayla gelen bir tedarikçi faturası:

1. güvenli şekilde alınabilir
2. saklanabilir
3. sınıflandırılabilir
4. OCR'dan geçirilip verisi çıkarılabilir
5. bir Odoo iş ortağıyla eşleştirilebilir
6. varsa bir satın alma siparişiyle eşleştirilebilir
7. deterministik olarak doğrulanabilir
8. gerektiğinde onaya yönlendirilebilir
9. taslak tedarikçi faturasına dönüştürülebilir
10. DMS'te arşivlenebilir
11. tamamen denetlenebilir
12. mükerrer kayıt oluşturmadan güvenle yeniden denenebilir
