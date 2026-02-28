[English](README.md)

# Backend Governance Framework

Claude Code CLI'yi yapilandirilmis bir muhendislik takimina donusturen, cok katmanli kalite ve guvenlik pipeline'i ile backend gelistirmeyi yoneten governance framework'u.

---

## Proje Aciklamasi

Bu framework, Claude Code'u bir **Team Lead / Koordinator** olarak calistirir. Claude kodu kendisi yazmak yerine, uzman sub-agent'lara delege eder ve her gorev icin uygun pipeline'i secer.

Backend gelistirme icin tasarlanmistir. .NET, Node.js ve Laravel icin hazir stack profilleri icerir.

---

## Temel Ozellikler

### 5 Uzman Agent

| Agent | Rol |
|-------|-----|
| `backend-developer` | Kod yazar: endpoint, service, migration, validation |
| `security-reviewer` | Guvenlik review: auth, injection, CORS, brute force |
| `quality-gate` | 11 nokta kalite kontrol ve test degerlendirmesi |
| `architect` | Mimari karar, YAGNI kontrolu, ADR |
| `devops` | Deployment, Docker, monitoring, logging, rollback |

### 3 Kademeli Pipeline (Maliyet / Kalite Dengesi)

| Kademe | Kapsam | Agent Sayisi |
|--------|--------|--------------|
| **Hafif** | Tek dosya degisikligi, auth/DB yok | 1 agent |
| **Normal** | Standart feature'lar (varsayilan) | 2 agent |
| **Tam** | Auth, odeme, migration, public API | 3-4 agent |

### 4 Muhendislik Modu

| Mod | Kullanim | Sertlik |
|-----|----------|---------|
| `explore` | Deneysel, PoC, spike | Dusuk |
| `build` | Standart feature (varsayilan) | Normal |
| `harden` | Auth, odeme, migration, guvenlik | Yuksek |
| `incident` | Canli hata, veri kaybi, acil rollback | Kritik |

### Diger Ozellikler

- **Feedback Dongusu** — Sub-agent'lar birbirleriyle direkt konusamaz. Tum iletisim Team Lead uzerinden gecer. Maksimum 3 dongu, sonra kullaniciya eskalasyon.
- **Catisma Onceligi** — Guvenlik > Dogruluk > Basitlik > Performans
- **Proje Kesfi** — Stack, DB ve mimari yapiyi otomatik tespit eder.
- **Coklu Stack Destegi** — .NET, Node.js, Laravel icin hazir profiller.

---

## Klasor Yapisi

```
backend-governance/
├── CLAUDE.md                  # Ana governance anlasmasi (Team Lead talimatlari)
├── .claude/agents/            # Agent tanimlari
│   ├── backend-developer.md
│   ├── security-reviewer.md
│   ├── quality-gate.md
│   ├── architect.md
│   ├── devops.md
│   └── qa-engineer.md
├── api/CLAUDE.md              # API kurallari
├── backend/CLAUDE.md          # Backend pattern'leri
├── guvenlik/CLAUDE.md         # Guvenlik kurallari
├── kalite/CLAUDE.md           # Kalite standartlari
├── karar/CLAUDE.md            # Karar kayitlari (ADR)
├── mimari/CLAUDE.md           # Mimari kurallar
├── operasyon/CLAUDE.md        # Operasyon ve izleme
├── proje/                     # Proje profilleri
│   ├── CLAUDE.md              # Proje profilleri nasil calisir
│   └── SABLON.md              # Yeni projeler icin sablon
├── qa/CLAUDE.md               # QA sureci
├── stack/                     # Stack'e ozel kurallar
│   ├── CLAUDE.md
│   ├── dotnet.md
│   ├── nodejs.md
│   └── laravel.md
├── surec/                     # Surecler ve is akislari
│   ├── CLAUDE.md              # Pipeline detaylari, feedback donguleri
│   ├── proje-kesfi.md         # Proje kesif sureci
│   └── deployment.md          # Deployment sureci
├── test/CLAUDE.md             # Test kurallari
└── veri/CLAUDE.md             # Veri ve veritabani kurallari
```

---

## Hizli Baslangic

**1. Repo'yu klonlayin**

```bash
git clone https://github.com/UfukCetinkaya57/claude-code-governance.git
```

**2. Projenizden backend-governance'a symlink olusturun**

```bash
# Linux / Mac
ln -s /path/to/backend-governance /path/to/your-project/backend-governance

# Windows
mklink /D "C:\path\to\your-project\backend-governance" "C:\path\to\backend-governance"
```

> Governance dizinini ASLA kopyalamayin. Her zaman symlink kullanin. Boylece bu repo'daki guncellemeler tum projelerinize aninda yansir.

**3. CLAUDE.md otomatik algilanir**

Claude Code, proje kokunde bulunan `CLAUDE.md` dosyasini otomatik olarak algilar. Ek yapilandirma gerekmez.

**4. Proje profili olusturun**

```bash
cp backend-governance/proje/SABLON.md backend-governance/proje/my-project.md
```

Sablon dosyasini projenize gore doldurun: stack, DB, mimari, aktif servisler.

**5. Calismaya baslayin**

Claude Code'a bir gorev verin. Claude Team Lead olarak hareket edecektir.

---

## Nasil Calisir

1. Claude Code'a bir gorev verirsiniz.
2. Claude (Team Lead olarak) gorevi analiz eder, muhendislik modunu ve pipeline kademesini secer.
3. Uygulamayi `backend-developer`'a delege eder.
4. Gorev auth veya guvenlik iceriyorsa `security-reviewer`'dan gecirir.
5. `quality-gate` kontrollerini calistirir.
6. Tum adimlar tamamlandiginda Proje Genel Durum Raporu ile sonuclari bildirir.

Sub-agent'lar birbirleriyle direkt iletisim kuramaz. Her handoff Team Lead uzerinden gerceklesir. Bir agent sorun buldugunda dongu durmaz — sorun cozulene kadar tekrar eder, maksimum 3 dongude cozulmezse kullaniciya eskalasyon yapilir.

---

## Ozellestirme

- `stack/` klasorune kendi stack profillerinizi ekleyin.
- `.claude/agents/` icindeki dosyalari duzenleyerek agent davranislarini degistirin.
- `proje/` icinde proje profilleri olusturarak domain-spesifik kurallar ekleyin.
- `surec/CLAUDE.md` icinde pipeline kademelerini ve tetikleme kriterlerini ayarlayin.

---

## Gereksinimler

- [Claude Code CLI](https://docs.anthropic.com/en/docs/claude-code)
- Yonetilecek bir backend projesi (.NET, Node.js veya Laravel)

---

## Lisans

MIT
