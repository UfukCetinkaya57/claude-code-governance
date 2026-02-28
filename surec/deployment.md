# DEPLOYMENT REHBERI

Governance framework'unun yeni projeye kurulumu.
Ana CLAUDE.md'den referans verilir, her konusmada otomatik yuklenmez.

---

## Yontem 1: Subagent'larla (Onerilen)

```
my-project/
├── CLAUDE.md                         ← ana governance dosyasinin icerigi
├── .claude/
│   └── agents/                       ← backend-governance/.claude/agents/ kopyala
│       ├── backend-developer.md
│       ├── security-reviewer.md
│       ├── quality-gate.md
│       ├── qa-engineer.md            ← uygulama ayaga kalktiktan sonra aktif
│       ├── architect.md
│       └── devops.md
├── backend-governance/               ← governance dosyalari
│   ├── api/CLAUDE.md
│   ├── guvenlik/CLAUDE.md
│   ├── stack/laravel.md
│   └── ...
└── app/
```

Farkli klasor adi kullanilirsa: bu dosya + 5 subagent dosyasindaki yollar guncellenir.

---

## Yontem 2: Subagent'siz (Basit)

Subagent kullanmak istemiyorsan, tum governance dosyalarini `@` ile import et:
```
@backend-governance/backend/CLAUDE.md
@backend-governance/api/CLAUDE.md
@backend-governance/guvenlik/CLAUDE.md
... (tum dosyalar)
```
Not: Bu ~1700 satir baslangicta yukler. Basit ama context agir olur.
