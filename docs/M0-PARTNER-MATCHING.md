# M0 — İş Ortağı ve Belge Yönlendirme Tasarımı

## Amaç

n8nOdoo'nun gelen bir belgenin ne anlama geldiğine ve hangi Odoo iş ortağına/sürecine ait olduğuna, AI'ın dayanaksız muhasebe kararları vermesine izin vermeden nasıl karar verdiğini tanımlamak.

## Temel ilke

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

(AI yorumu + Odoo ana verisi + işlem geçmişi + deterministik kurallar → doğrulanmış yönlendirme kararı)

AI adaylar ve çıkarılmış gerçekler üretir. Odoo verisi ve deterministik kurallar bunları doğrular.

## Adım 1 — Alıcı şirketi belirle

Muhasebe kaydı oluşturmadan önce şirket güvenilir bağlamdan belirlenmeli.

Olası kaynaklar, öncelik sırasıyla:

1. Özel posta kutusu/kanal yapılandırması
2. n8n iş akışı yapılandırması
3. Belgede açık şirket tanımlayıcısı
4. Belirsizse elle inceleme

Birden fazla şirket mümkünken şirketi yalnızca LLM tahminine dayanarak çıkarma.

## Adım 2 — İş ortağı tanımlayıcılarını çıkar

Tercih edilen tanımlayıcılar:

1. Türk VKN/TCKN veya geçerli vergi tanımlayıcısı
2. Varsa e-fatura/e-belge tanımlayıcıları
3. İlgiliyse IBAN
4. E-posta/alan adı
5. Telefon
6. Normalleştirilmiş ticari unvan
7. Adres

Sistem, çıkarılan orijinal değeri ve normalleştirilmiş değeri birlikte saklamalı.

## Adım 3 — Odoo iş ortaklarında ara

Aday eşleştirme, Odoo'yu giderek zayıflayan sinyallerle sorgulamalı.

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

(Birebir VKN → yoksa birebir harici/e-fatura tanımlayıcısı → yoksa bilinen banka hesabı / IBAN → yoksa normalleştirilmiş unvan + şirket bağlamı → yoksa e-posta/alan adı/telefon → elle inceleme)

Zayıf bir unvan eşleşmesi, çelişen birebir bir vergi tanımlayıcısını asla sessizce geçersiz kılmamalı.

## Müşteri ve tedarikçi

Bir iş ortağı her iki role de sahip olabilir. Basit bir müşteri/tedarikçi ikili sınıflandırması kullanma.

Değerlendir:

- tedarikçi sırası / tedarikçi durumu
- müşteri sırası / müşteri durumu
- mevcut tedarikçi faturaları
- mevcut müşteri faturaları
- satın alma siparişleri
- satış siparişleri
- yakın tarihli işlem geçmişi
- belge yönü ve kanal

Örnek:

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

(İş ortağı hem müşteri hem tedarikçi → gelen fatura → satın alma tarafı kanıtı ara → sipariş / tedarikçi geçmişi / tedarikçi yapılandırması → tedarikçi faturası akışına yönlendir)

## Belge sınıflandırma

İlk belge sınıfları:

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

Sınıflandırma hem tür hem güven skoru döndürmeli; ancak güven skoru tek başına bir muhasebe işlemine yetki vermez.

## Satın alma faturası ve gider faturası

Bir tedarikçi faturası otomatik olarak satın alma siparişi faturası değildir.

### Satın alma tarafı kanıtları

Ara:

- eşleşen açık/onaylı satın alma siparişi
- eşleşen tedarikçi
- eşleşen ürün/hizmet satırları
- eşleşen miktarlar
- tolerans içinde eşleşen fiyatlar
- beklenen alış vergileri
- depo/satın alma bağlamı

### Gider tarafı kanıtları

Olası göstergeler:

- ilgili satın alma siparişi yok
- tekrarlayan elektrik-su/telekom/kira/hizmet tedarikçisi
- gider kategorileri için yapılandırılmış tedarikçi
- gider hesabı/kategori geçmişi
- açıkça genel bir işletme giderini tarif eden belge

Her iki yol da makul kalıyorsa tahmin etmek yerine elle inceleme görevi oluştur.

## Satın alma siparişi eşleştirme

Aday sipariş puanlamasında kullanılabilecekler:

- iş ortağı birebir eşleşmesi
- şirket birebir eşleşmesi
- para birimi eşleşmesi
- ürün/hizmet örtüşmesi
- miktar uyumu
- fiyat uyumu
- tarih yakınlığı
- sipariş durumu

Kavramsal puan örneği:

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

Bunlar tasarım ağırlıklarıdır, üretim eşikleri değil. Eşikler yapılandırılabilir olmalı ve gerçek belgelere karşı test edilmeli.

## Fatura doğrulaması

Tedarikçi faturası oluşturmadan önce:

- tedarikçi eşleşmesi yeterince güçlü
- şirket biliniyor
- gerektiği yerde fatura numarası var
- mükerrer araması tamamlandı
- para birimi geçerli
- vergi yapılandırması geçerli
- satır toplamları tutuyor
- ara toplam + vergi = toplam (yapılandırılan tolerans içinde)
- sipariş eşleşmesi ya geçerli ya da açıkça gerekli değil
- yüksek risk koşulları yok

## Mükerrer tespiti

Birincil adaylar:

```text
company + supplier + supplier_invoice_number
```

İkincil sinyaller:

- fatura tarihi
- toplam
- para birimi
- belge hash'i
- kaynak mesaj kimliği
- ek hash'i

Mükerrerler idempotent olarak yok sayılmalı veya incelemeye yönlendirilmeli; asla sessizce iki kez işlenmemeli.

## Risk kuralları

En azından şunlar için her zaman insan doğrulaması iste:

- tedarikçi banka hesabı değişikliği
- çelişen vergi tanımlayıcıları
- belirsiz şirket
- belirsiz iş ortağı
- mükerrer şüphesi
- şirket politikasına göre olağandışı yüksek tutarlı fatura
- yapılandırılan toleransın üstünde sipariş/fatura uyuşmazlığı
- desteklenmeyen veya okunamayan belge

## Karar nesnesi

Yönlendirme katmanı şuna benzer normalleştirilmiş bir karar üretmeli:

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

Bu nesne iş akışı yönlendirmesi için bir öneridir, muhasebe kaydı değildir.

## Elle inceleme

Elle inceleme şunları göstermeli:

- orijinal belge
- çıkarılan alanlar
- aday Odoo iş ortak(lar)ı
- ilgili satın alma/satış siparişi adayları
- doğrulama hataları
- risk gerekçeleri
- önerilen işlem

İnceleyen kişi önerilen yönlendirmeyi onaylayabilir, reddedebilir veya düzeltebilir.

## Uygulama sınırı

### n8n

- alma/veri çıkarma
- AI/OCR çağırma
- Odoo arama API'lerini çağırma
- aday verileri birleştirme
- iş akışı yönlendirmesini yürütme
- onay talebi oluşturma
- yeniden deneme/hata yönetimi

### Odoo modülü

- standart API çağrılarının yetmediği yerde güvenli arama metotları sunmak
- normalleştirilmiş belge, karar, onay ve denetim kayıtlarını tutmak
- nihai iş kayıtlarını oluşturmak/güncellemek
- kritik işlemler için sunucu tarafı doğrulamayı zorunlu kılmak

### AI

- sınıflandırma
- veri çıkarma
- aday eşleşme önerme
- veri çıkarma belirsizliğini açıklama

AI muhasebe kayıtlarını doğrudan oluşturmamalı/işlememeli.

## Sonraki uygulama görevleri

- [ ] İş ortağı aday API sözleşmesini tanımla
- [ ] Normalleştirilmiş şirket bağlamını tanımla
- [ ] İş ortağı eşleştirme normalleştirme fonksiyonlarını tanımla
- [ ] Satın alma siparişi aday API'sini tanımla
- [ ] Mükerrer kontrol API'sini tanımla
- [ ] Satın alma/gider kural yapılandırmasını tanımla
- [ ] Risk politikası yapılandırmasını tanımla
- [ ] Yönlendirme kararı JSON şemasını oluştur
- [ ] Müşteri+tedarikçi çift rollü iş ortakları için test senaryoları yaz
- [ ] Siparişsiz gider faturaları için test senaryoları yaz
- [ ] Belirsiz iş ortağı eşleşmeleri için test senaryoları yaz
