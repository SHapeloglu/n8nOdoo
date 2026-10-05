# SESSION.md — n8nOdoo Oturum Günlüğü

Her çalışma oturumunda buraya kısa bir kayıt düşülür: ne yapıldı, hangi kararlar alındı, sıradaki adım ne. Amaç, bir sonraki oturuma (veya başka bir geliştiriciye/Claude örneğine) hızlıca bağlam aktarmak.

---

## Şablon

```markdown
## YYYY-AA-GG

**Yapılanlar:**
- ...

**Alınan kararlar / neden:**
- ...

**Açık sorunlar / bilinen eksikler:**
- ...

**Sıradaki adım:**
- ...
```

---

## 2026-10-05

**Yapılanlar:**
- Eksik proje çalışma dosyaları oluşturuldu: `BACKLOG.md`, `CLAUDE.md`, `SESSION.md`.
- İçerik; README, dosya yapısı, bağımlılık dosyaları ve git geçmişinden çıkarıldı.

**Açık sorunlar / bilinen eksikler:**
- Repo kökünde `.gitignore` yok — `venv/`, `__pycache__/`, `.env`, build çıktıları için eklenmeli.
- Otomatik test bulunamadı — kritik akışlar için en azından duman (smoke) testleri eklenmeli.

**Sıradaki adım:**
- `CLAUDE.md` ve `ARCHITECTURE.md` içeriğini gözden geçirip proje sahibinin bilgisiyle tamamla.

### Bu tarihten önceki son commit'ler (referans)

- 2026-09-29 — feat: add read-only Odoo partner lookup workflow
- 2026-09-29 — docs: define Odoo partner matching API design
- 2026-09-29 — docs: define partner matching and document routing
- 2026-09-28 — feat: add normalized invoice schema
- 2026-09-28 — docs: analyze invoice LLM reference implementation
- 2026-09-28 — docs: add M0 reuse and dependency assessment
- 2026-09-28 — docs: initialize n8nOdoo project documentation
- 2026-09-28 — docs: initialize n8nOdoo project documentation
- 2026-09-28 — docs: initialize n8nOdoo project documentation
- 2026-09-28 — docs: initialize n8nOdoo project documentation
- 2026-09-28 — docs: initialize n8nOdoo project documentation
