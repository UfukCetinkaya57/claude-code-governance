[English](README.en.md)

# Backend Governance Framework

Claude Code CLI'yı yapılandırılmış bir mühendislik takımına dönüştüren, çok katmanlı kalite ve güvenlik pipeline'ı ile backend geliştirmeyi yöneten governance framework'ü.

---

## Proje Açıklaması

Bu framework, Claude Code'u bir **Team Lead / Koordinatör** olarak çalıştırır. Claude kodu kendisi yazmak yerine, uzman sub-agent'lara delege eder ve her görev için uygun pipeline'ı seçer.

Backend geliştirme için tasarlanmıştır. .NET, Node.js ve Laravel için hazır stack profilleri içerir.

---

## Temel Özellikler

### 5 Uzman Agent

| Agent | Rol |
|-------|-----|
| `backend-developer` | Kod yazar: endpoint, service, migration, validation |
| `security-reviewer` | Güvenlik review: auth, injection, CORS, brute force |
| `quality-gate` | 11 nokta kalite kontrol ve test değerlendirmesi |
| `architect` | Mimari karar, YAGNI kontrolü, ADR |
| `devops` | Deployment, Docker, monitoring, logging, rollback |

### 3 Kademeli Pipeline (Maliyet / Kalite Dengesi)

| Kademe | Kapsam | Agent Sayısı |
|--------|--------|--------------|
| **Hafif** | Tek dosya değişikliği, auth/DB yok | 1 agent |
| **Normal** | Standart feature'lar (varsayılan) | 2 agent |
| **Tam** | Auth, ödeme, migration, public API | 3-4 agent |

### 4 Mühendislik Modu

| Mod | Kullanım | Sertlik |
|-----|----------|---------|
| `explore` | Deneysel, PoC, spike | Düşük |
| `build` | Standart feature (varsayılan) | Normal |
| `harden` | Auth, ödeme, migration, güvenlik | Yüksek |
| `incident` | Canlı hata, veri kaybı, acil rollback | Kritik |

### Diğer Özellikler

- **Feedback Döngüsü** — Sub-agent'lar birbirleriyle direkt konuşamaz. Tüm iletişim Team Lead üzerinden geçer. Maksimum 3 döngü, sonra kullanıcıya eskalasyon.
- **Çatışma Önceliği** — Güvenlik > Doğruluk > Basitlik > Performans
- **Proje Keşfi** — Stack, DB ve mimari yapıyı otomatik tespit eder.
- **Çoklu Stack Desteği** — .NET, Node.js, Laravel için hazır profiller.

---

## Klasör Yapısı

```
backend-governance/
├── CLAUDE.md                  # Ana governance anlaşması (Team Lead talimatları)
├── .claude/agents/            # Agent tanımları
│   ├── backend-developer.md
│   ├── security-reviewer.md
│   ├── quality-gate.md
│   ├── architect.md
│   ├── devops.md
│   └── qa-engineer.md
├── api/CLAUDE.md              # API kuralları
├── backend/CLAUDE.md          # Backend pattern'leri
├── guvenlik/CLAUDE.md         # Güvenlik kuralları
├── kalite/CLAUDE.md           # Kalite standartları
├── karar/CLAUDE.md            # Karar kayıtları (ADR)
├── mimari/CLAUDE.md           # Mimari kurallar
├── operasyon/CLAUDE.md        # Operasyon ve izleme
├── proje/                     # Proje profilleri
│   ├── CLAUDE.md              # Proje profilleri nasıl çalışır
│   └── SABLON.md              # Yeni projeler için şablon
├── qa/CLAUDE.md               # QA süreci
├── stack/                     # Stack'e özel kurallar
│   ├── CLAUDE.md
│   ├── dotnet.md
│   ├── nodejs.md
│   └── laravel.md
├── surec/                     # Süreçler ve iş akışları
│   ├── CLAUDE.md              # Pipeline detayları, feedback döngüleri
│   ├── proje-kesfi.md         # Proje keşif süreci
│   └── deployment.md          # Deployment süreci
├── test/CLAUDE.md             # Test kuralları
└── veri/CLAUDE.md             # Veri ve veritabanı kuralları
```

---

## Hızlı Başlangıç

**1. Repo'yu klonlayın**

```bash
git clone https://github.com/UfukCetinkaya57/claude-code-governance.git
```

**2. Projenizden backend-governance'a symlink oluşturun**

```bash
# Linux / Mac
ln -s /path/to/backend-governance /path/to/your-project/backend-governance

# Windows
mklink /D "C:\path\to\your-project\backend-governance" "C:\path\to\backend-governance"
```

> Governance dizinini ASLA kopyalamayın. Her zaman symlink kullanın. Böylece bu repo'daki güncellemeler tüm projelerinize anında yansır.

**3. CLAUDE.md otomatik algılanır**

Claude Code, proje kökünde bulunan `CLAUDE.md` dosyasını otomatik olarak algılar. Ek yapılandırma gerekmez.

**4. Proje profili oluşturun**

```bash
cp backend-governance/proje/SABLON.md backend-governance/proje/my-project.md
```

Şablon dosyasını projenize göre doldurun: stack, DB, mimari, aktif servisler.

**5. Çalışmaya başlayın**

Claude Code'a bir görev verin. Claude Team Lead olarak hareket edecektir.

---

## Nasıl Çalışır

1. Claude Code'a bir görev verirsiniz.
2. Claude (Team Lead olarak) görevi analiz eder, mühendislik modunu ve pipeline kademesini seçer.
3. Uygulamayı `backend-developer`'a delege eder.
4. Görev auth veya güvenlik içeriyorsa `security-reviewer`'dan geçirir.
5. `quality-gate` kontrollerini çalıştırır.
6. Tüm adımlar tamamlandığında Proje Genel Durum Raporu ile sonuçları bildirir.

Sub-agent'lar birbirleriyle direkt iletişim kuramaz. Her handoff Team Lead üzerinden gerçekleşir. Bir agent sorun bulduğunda döngü durmaz — sorun çözülene kadar tekrar eder, maksimum 3 döngüde çözülmezse kullanıcıya eskalasyon yapılır.

---

## Özelleştirme

- `stack/` klasörüne kendi stack profillerinizi ekleyin.
- `.claude/agents/` içindeki dosyaları düzenleyerek agent davranışlarını değiştirin.
- `proje/` içinde proje profilleri oluşturarak domain-spesifik kurallar ekleyin.
- `surec/CLAUDE.md` içinde pipeline kademelerini ve tetikleme kriterlerini ayarlayın.

---

## Gereksinimler

- [Claude Code CLI](https://docs.anthropic.com/en/docs/claude-code)
- Yönetilecek bir backend projesi (.NET, Node.js veya Laravel)

---

## Lisans

MIT
