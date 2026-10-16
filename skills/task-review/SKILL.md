---
name: task-review
description: Kodlanmış bir task'ı kontrol eder, testleri çalıştırır ve eksik testleri yazar, güvenlik dahil detaylı kod review yapar ve önceliklendirilmiş feedback verir. Kullanıcı "review yap", "kodu kontrol et", "test et", "feedback ver", "PR'ı incele" dediğinde kullan.
---

# Kod Kontrolü + Test + Code Review + Feedback

Girdi: incelenecek değişiklik. Kaynağı şu sırayla belirle:
1. Kullanıcı bir PR / branch / commit / dosya verdiyse onu kullan.
2. Yoksa çalışma dizinindeki değişiklikler: `git diff` + `git diff --staged`; boşsa ana branch'e göre fark
   (`git diff $(git merge-base HEAD main)` — main yoksa master/develop).
3. Varsa roadmap (`task-analysis` çıktısı) ve kodlama raporunu (`task-implementation` çıktısı) da oku;
   review'ı bunlara karşı yap.

Kullanıcıdan ek bilgi bekleme; kapsamı yukarıdaki sırayla kendin belirle ve rapora yaz.

## Adım 1 — Bağlamı anla
- Değişikliğin amacı ne? (roadmap, commit mesajları, PR açıklaması)
- Değişen dosyaları ve bunların çağırdığı/çağrıldığı yerleri oku. Sadece diff'e değil, etkilediği koda da bak.

## Adım 2 — Doğrulama (paralel çalıştırılabilir)
Aşağıdaki agent'ları **aynı anda** başlat, sonuçlarını birleştir:
- `test-engineer`: mevcut testleri çalıştırır, değişikliğin test kapsamını ölçer, eksik testleri yazar ve çalıştırır.
- `security-auditor`: değişikliği güvenlik açısından inceler.

Agent kullanılamıyorsa bu işleri kendin yap: build, lint, tüm testler; eksik kritik testleri yaz.

## Adım 3 — Kod review kontrol listesi

**Doğruluk**
- Mantık hataları, off-by-one, null/boş değer, yanlış koşul, hatalı hata yakalama
- Eşzamanlılık: race condition, transaction sınırları, kilitler, idempotency
- Hata durumları: harici servis hatası, timeout, retry, kısmi başarısızlık
- Roadmap'teki gereksinimleri ve kabul kriterlerini gerçekten karşılıyor mu?

**Tasarım ve bakım**
- Mevcut mimari ve desenlere uyum; katman ihlali (ör. controller'da iş mantığı)
- Tekrarlanan kod, gereksiz karmaşıklık, sorumluluk dağılımı
- İsimlendirme, okunabilirlik, ölü kod, TODO/FIXME

**Veri ve API**
- Migration güvenli ve geri alınabilir mi? Büyük tablolarda kilit riski?
- Index ihtiyacı, N+1 sorgu, gereksiz veri çekme
- API geriye dönük uyumlu mu? Hata kodları ve yanıt formatı tutarlı mı?

**Performans ve operasyon**
- Döngü içinde I/O, büyük bellek kullanımı, sayfalama eksikliği
- Loglama seviyesi ve içeriği (hassas veri loglanmamalı), metrik/trace
- Konfigürasyon: sabit değerler, ortam bağımlılıkları

**Testler**
- Testler davranışı mı doğruluyor yoksa implementasyonu mu? Anlamsız assertion var mı?
- Negatif senaryolar ve uç durumlar test edilmiş mi?

## Adım 4 — Bulguları önceliklendir

| Seviye | Anlamı |
|---|---|
| 🔴 KRİTİK | Merge edilmemeli: bug, veri kaybı, güvenlik açığı, kırık test/build |
| 🟠 YÜKSEK | Merge öncesi düzeltilmeli: eksik gereksinim, önemli uç durum, ciddi performans sorunu |
| 🟡 ORTA | Düzeltilmesi önerilir: bakım, okunabilirlik, eksik test |
| 🔵 DÜŞÜK | Öneri / stil / küçük iyileştirme |

Her bulgu somut olmalı: dosya:satır, sorunun ne olduğu, neden önemli olduğu, nasıl düzeltileceği (mümkünse kod örneğiyle).
Emin olmadığın bulguları "doğrulanmadı" diye işaretle; tahmini kesin bilgi gibi yazma.

## Rapor şablonu

```markdown
# Code Review: <Task / PR adı>

## Karar: ✅ Onaylandı | 🔁 Değişiklik gerekli | ❌ Reddedildi
<1-2 cümle gerekçe>

## Özet
- İncelenen kapsam: <diff kaynağı, dosya sayısı>
- Bulgular: 🔴 x · 🟠 x · 🟡 x · 🔵 x

## Test sonuçları
- Build: ✅/❌
- Testler: X geçti / Y kaldı (komut: `...`)
- Yeni eklenen testler: ...
- Kapsam notu: ...

## Gereksinim kontrolü
| Gereksinim / AC | Durum | Not |
|---|---|---|

## Bulgular
### 🔴 KRİTİK
#### K-1: <başlık>
- **Konum:** `dosya:satır`
- **Sorun:** ...
- **Neden önemli:** ...
- **Öneri:** ...
### 🟠 YÜKSEK
...
### 🟡 ORTA
...
### 🔵 DÜŞÜK
...

## Güvenlik özeti
<security-auditor sonucundan özet>

## Olumlu noktalar
- ...

## Yapılacaklar (öncelik sırasıyla)
- [ ] ...
```

Bulgu yoksa uydurma; "Bulgu yok" yaz ve kararı buna göre ver.
Kullanıcı isterse düzeltmeleri `backend-coder` agent'ı ile uygulayıp review'ı tekrarla.
