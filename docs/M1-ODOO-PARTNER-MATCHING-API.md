# M1 — Odoo İş Ortağı Eşleştirme API Tasarımı

## Hedef

Herhangi bir belge işlenmeden önce n8n'in bir iş ortağını araması ve tanımlaması için gereken en küçük Odoo entegrasyonunu tanımlamak.

## İlke

İlk entegrasyon bilinçli olarak salt okunurdur.

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

(n8n → Odoo 18 → iş ortağı araması → normalleştirilmiş adaylar → n8n kuralları)

Bu kilometre taşında fatura, ödeme, satın alma siparişi veya iş ortağı üzerinde hiçbir değişiklik yapılmaz.

## Aday arama sırası

Önce en güçlü tanımlayıcıları kullan:

1. Varsa birebir VKN/TCKN
2. Yapılandırılmışsa birebir ticaret sicili/şirket kayıt tanımlayıcısı
3. Uygunsa birebir normalleştirilmiş e-posta/alan adı
4. Uygunsa birebir normalleştirilmiş telefon
5. Verilmişse birebir iş ortağı referansı (`ref`)
6. Daha zayıf aday araması olarak unvan araması

Yalnızca unvana dayalı eşleşmeler yetkili kimlik olarak kabul edilmemeli.

## Odoo iş ortağı alanları

Başlangıçta çekilecek alanlar:

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

Ek alanlar yalnızca somut bir kural gerektirdiğinde istenmeli.

## Normalleştirilmiş yanıt

n8n, Odoo iş ortağı kaydının tamamına bağımlı olmamalı. Entegrasyon bunu normalleştirmeli:

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

Olası durumlar:

- `matched`
- `multiple_candidates`
- `not_found`
- `invalid_query`
- `odoo_error`

## Eşleşme anlamları

`confidence` bir eşleşme puanıdır, AI güven skoru değildir.

Örnekler:

```text
VAT exact                 1.00
Partner reference exact  1.00
Email exact               0.90
Phone exact               0.85
Name exact                0.70
Name fuzzy                < 0.70
```

(VKN birebir 1.00 · iş ortağı referansı birebir 1.00 · e-posta birebir 0.90 · telefon birebir 0.85 · unvan birebir 0.70 · unvan bulanık < 0.70)

Bunlar yalnızca ilk tasarım değerleridir. Üretim eşikleri gerçek şirket verisine karşı test edilmeli.

## Müşteri ve tedarikçi

`customer_rank > 0` ile `supplier_rank > 0`'ı birbirini dışlayan durumlar olarak ele alma.

Bir iş ortağı her ikisi de olabilir.

Yönlendirme daha sonra şunlarla belirlenir:

- gelen belge türü
- iş ortağı rolü
- açık satın alma siparişleri
- açık satış siparişleri
- geçmiş işlemler
- şirket bağlamı
- deterministik iş kuralları.

## Şirket bağlamı

Kurulum çoklu şirketliyse her arama alıcı Odoo şirket bağlamını içermeli.

Odoo'da bulunan bir iş ortağının otomatik olarak alıcı şirkete ait olduğunu varsayma.

## Güvenlik

M1 entegrasyonu en az yetkili bir Odoo entegrasyon kullanıcısı kullanmalı.

Başlangıçta gereken okuma erişimi:

- `res.partner`

M1 iş ortağı araması için write/create/unlink erişimi gerekmez.

Kimlik bilgileri n8n'in kimlik bilgisi deposunda kalmalı; iş akışı JSON'unda, loglarda veya prompt'larda asla görünmemeli.

## n8n alt iş akışı

Önerilen yeniden kullanılabilir iş akışı:

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

(Girdi: isteğe bağlı vat/name/email/phone/ref/company_id → sorguyu doğrula → VKN araması → sonuç yoksa referans araması → sonuç yoksa e-posta/telefon araması → sonuç yoksa unvanla aday araması → adayları normalleştir → durum + adayları döndür)

## Sonraki uygulama adımı

Salt okunur `n8n → Odoo → res.partner` kavram kanıtını kur.

Kabul testi:

```text
Given a known supplier VAT number
When n8n calls the partner lookup
Then exactly the expected Odoo partner is returned
And no Odoo data is modified.
```

(Bilinen bir tedarikçi VKN'si verildiğinde, n8n iş ortağı aramasını çağırınca tam olarak beklenen Odoo iş ortağı döner ve hiçbir Odoo verisi değişmez.)

Ardından şu testleri ekle:

- yalnızca müşteri olan iş ortağı
- yalnızca tedarikçi olan iş ortağı
- müşteri + tedarikçi iş ortağı
- mükerrer/belirsiz unvan
- VKN eksik
- pasif iş ortağı
- çoklu şirket iş ortağı
- Odoo kimlik doğrulama hatası
- Odoo zaman aşımı
