# PROJE-SPESIFIK KURALLAR

## Nasil Calisir

Bu klasor, her proje icin ozel kurallar icerir.
Governance kurallari genel ve stack-bagimsizdir — proje dosyalari ise o projeye ozel kararlari, kisitlamalari ve domain kurallarini tanimlar.

## Proje Dosyasi Olusturma

1. `SABLON.md` dosyasini kopyala: `cp SABLON.md my-project.md`
2. Tum alanlari doldur (Kimlik, Domain Kurallari, Kararlar, Kisitlamalar, Kritik Akislar, Ortam)
3. Asagidaki "Aktif Proje" bolumune dosya adini yaz

## Aktif Proje

Bu klasor sadece SABLON icerir.
Aktif projeler kendi dizinlerinde yasarlar (ornek: `../my-project/proje/`).
Her proje kendi `proje/CLAUDE.md` dosyasinda aktif projeyi isaret eder.

Yeni proje eklemek icin `SABLON.md` dosyasini kopyalayip doldurun.

## Proje Dosyasi Ne Icerir

| Bolum | Icerik |
|-------|--------|
| Kimlik | Proje adi, domain, stack, DB, durum |
| Domain Kurallari | Sektore ozel kurallar (e-ticaret, fintech, SaaS, vb.) |
| Proje-Spesifik ADR | Bu projeye ozel mimari kararlar |
| Bilinen Kisitlamalar | Teknik/is kisitlari (legacy entegrasyon, hosting, vb.) |
| Kritik Akislar | harden mode'u otomatik tetikleyen akislar |
| Ortam Bilgisi | Dev/staging/prod URL, DB turu, CI/CD |
