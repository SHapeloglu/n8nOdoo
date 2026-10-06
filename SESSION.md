# SESSION.md — n8nOdoo

En yeni kayıt en üstte: yapılanlar, kararlar, açık konular, sonraki adım.

---

## 2026-10-06

- Tüm .md belgeleri Türkçeye çevrildi; "belgeler İngilizce" kuralı "belgeler Türkçe" olarak değiştirildi.

---

## 2026-10-05

- Şablondan üretilmiş `CLAUDE.md`, `BACKLOG.md` ve `SESSION.md`, depodan (README, ARCHITECTURE, PROJECT_PLAN, ROADMAP, docs/, şema, iş akışı) türetilen içerikle değiştirildi.
- Açık: `.gitignore` yok (kod/venv/.env oluşmadan önce ekle); henüz test yok; M1 iş ortağı arama iş akışı gerçek bir Odoo 18 örneğine karşı çalıştırılmadı.
- Sonraki: `001_odoo_partner_lookup.json`'u salt okunur API kullanıcısıyla Odoo test DB'sine karşı çalıştır ve `docs/M1-ODOO-PARTNER-MATCHING-API.md` içindeki kabul senaryolarının sonuçlarını kaydet (bilinen VKN, yalnızca müşteri, yalnızca tedarikçi, her ikisi, belirsiz, VKN eksik, pasif, çoklu şirket, kimlik doğrulama hatası, zaman aşımı).

---

## 2026-09-29

- İş ortağı eşleştirme ve belge yönlendirme kuralları (`docs/M0-PARTNER-MATCHING.md`).
- Odoo iş ortağı eşleştirme API tasarımı (`docs/M1-ODOO-PARTNER-MATCHING-API.md`).
- İlk n8n iş akışı: VKN ile salt okunur Odoo iş ortağı araması.

## 2026-09-28

- Proje belgeleri oluşturuldu (README, ARCHITECTURE, PROJECT_PLAN, ROADMAP, TASKS).
- M0 yeniden kullanım/bağımlılık değerlendirmesi ve fatura-LLM referans analizi.
- Normalleştirilmiş tedarikçi faturası JSON şeması.
