# PROJE: {Proje Adi}

> Bu dosya yeni bir projede ilk calisma basladiginda otomatik olusturulur.
> Team Lead projeyi tarar, tespit edebildiklerini doldurur, geri kalanini kullaniciya sorar.
> Doldurulan dosya o projenin CLAUDE.md'si gibi davranir — tum governance
> kurallari + bu dosyadaki proje-spesifik kurallar birlikte gecerli olur.

---

## Kimlik

> Otomatik: Proje adi, Stack, DB, Durum — proje taranarak doldurulur.
> Kullaniciya sorulur: Domain.

| Alan | Deger | Kaynak |
|------|-------|--------|
| Proje adi | {ornek: TrendwayTravel Poland} | otomatik |
| Domain | {ornek: seyahat, e-ticaret, fintech, SaaS} | kullaniciya sor |
| Stack | {ornek: .NET, Node.js, Laravel} | otomatik |
| DB | {ornek: PostgreSQL, SQL Server, MySQL} | otomatik |
| Durum | {ornek: yeni / devam / bakim / migration} | otomatik |

---

## Domain-Spesifik Kurallar

> Kullaniciya sorulur. Domain belirlendikten sonra asagidaki orneklerden ilgili olanlar secilir,
> gerekirse yenileri eklenir. Kullanici "sonra" derse bos birakilir.

### E-Ticaret ise:
- Fiyat hesaplamalarinda DECIMAL kullan, FLOAT ASLA
- Stok azaltma idempotent olmali (ayni siparis 2 kez stok dusmemeli)
- Odeme islemi basarisiz olursa siparis durumu tutarli kalmali
- Kupon/indirim hesabi server-side, client'a guvenme

### Fintech / Odeme ise:
- Tum parasal islemler transaction icinde
- Audit log zorunlu (kim, ne zaman, ne yapti)
- PCI-DSS uyumlulugu kontrolu
- Idempotency key zorunlu (her odeme isteginde)

### SaaS / Multi-tenant ise:
- Tenant isolation: her sorguda tenant filtresi zorunlu
- Tenant ID middleware'de set edilir, manual gecilmez
- Cross-tenant veri sizintisi testi zorunlu
- Rate limiting tenant bazli

### API Gateway / Public API ise:
- Versiyonlama stratejisi kararli (URL vs header)
- Breaking change = major version artisi
- Deprecation sureci tanimli (min 3 ay uyari)
- API key yonetimi ve throttling

---

## Proje-Spesifik Kararlar (ADR)

> Projeye ozel alinan kararlar buraya yazilir.

| # | Karar | Tarih | Neden |
|---|-------|-------|-------|
| 1 | {ornek: Response formati: Envelope pattern} | {tarih} | {Frontend ekibi tutarli format istiyor} |
| 2 | {ornek: PK: UUID (auto-increment degil)} | {tarih} | {Distributed sistem plani var} |
| 3 | ... | ... | ... |

---

## Bilinen Kisitlamalar

> Kullaniciya sorulur. Projenin teknik veya is kisitlari.

- {ornek: Legacy SOAP servisi ile entegrasyon zorunlu}
- {ornek: Hosting: shared hosting, Docker yok}
- {ornek: Ucuncu parti API rate limiti: 100 istek/dk}
- {ornek: iOS App Store review sureci 2 hafta}

---

## Kritik Akislar

> Kullaniciya sorulur. Bu projedeki en riskli / en onemli is akislari.
> Bunlar icin harden mode otomatik aktif olur.

1. {ornek: Odeme akisi (siparis -> odeme -> onay -> stok dusme)}
2. {ornek: Kullanici kayit + email dogrulama}
3. {ornek: Veri migration'i (eski sistemden yeni sisteme)}

---

## Ortam Bilgisi

> Otomatik: Config dosyalari, dependency ve CI/CD dosyalarindan tespit edilir.

| Ortam | URL / Bilgi | Kaynak |
|-------|-------------|--------|
| Development | {localhost:5000 veya bos} | otomatik |
| Staging | {staging.example.com veya bos} | otomatik / kullaniciya sor |
| Production | {api.example.com veya bos} | otomatik / kullaniciya sor |
| DB | {connection bilgisi yerine sadece tur: PostgreSQL 15} | otomatik |
| Cache | {Redis / yok} | otomatik |
| CI/CD | {GitHub Actions / Azure DevOps / yok} | otomatik |
