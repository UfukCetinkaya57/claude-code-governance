# PROJE KESFI & STACK TESPITI

Bu dosya yeni projeye girildiginde veya proje dosyasi yoksa kullanilir.
Ana CLAUDE.md'den referans verilir, her konusmada otomatik yuklenmez.

---

## Stack Tespiti (Otomatik)

Projeye girildiginde aktif stack tespit edilir:

| Sinyal | Stack | Dosya |
|--------|-------|-------|
| `*.csproj` / `*.sln` | .NET | `backend-governance/stack/dotnet.md` |
| `package.json` + Express/Fastify | Node.js | `backend-governance/stack/nodejs.md` |
| `composer.json` + Laravel | Laravel | `backend-governance/stack/laravel.md` |

Stack dosyasi tespit sonrasi okunur (otomatik yuklenmez).

---

## Otomatik Proje Kesfi

Ilk Adim'da aktif proje dosyasi yoksa, Team Lead projeyi tarar ve `proje/` klasorune dosya olusturur.

### Otomatik Tespit Edilenler (tarama ile)

| Alan | Nereden |
|------|---------|
| Proje adi | Kok klasor adi, package.json `name`, *.csproj AssemblyName |
| Stack | Ilk Adim'da zaten tespit ediliyor |
| DB | Config dosyalari: appsettings.json, .env, prisma/schema.prisma, config/database.php |
| Durum | git log var → devam / git yok → yeni |
| Cache | Dependency'lerde Redis paketi var mi (StackExchange.Redis, ioredis, predis) |
| CI/CD | `.github/workflows/` → GitHub Actions, `azure-pipelines.yml` → Azure DevOps |
| Auth yaklasimi | JWT paketi, Sanctum, Passport vb. dependency'lerden |
| Mevcut yapilar | Klasor yapisi (Controllers/, Services/, migrations/ vb.) |

### Kullaniciya Sorulanlar (otomatik tespit edilemez)

- **Domain:** Projenin alani (e-ticaret, fintech, SaaS, vb.)
- **Kritik akislar:** Hangi is akislari harden mode tetiklemeli?
- **Bilinen kisitlamalar:** Legacy entegrasyon, hosting limitleri, vb.
- **Domain-spesifik kurallar:** Sektore ozel kurallar

### Akis

```
1. Projeyi tara (dosyalar, config, dependency, git)
2. Tespit edilenleri doldur (Kimlik + Ortam tablolari)
3. Kullaniciya sor: domain, kritik akislar, kisitlamalar
4. proje/{proje-adi}.md olustur
5. proje/CLAUDE.md'de aktif proje olarak isaretle
```

Not: Kullanici "bilmiyorum" veya "sonra" derse, bos birak — ilerledikce tamamlanir.
