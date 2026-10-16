---
name: task-implementation
description: Analiz edilmiş bir task'ı veya roadmap'i koda döker; kodlama sırasında çıkan sorunları, eksikleri ve belirsizlikleri raporlar; kodlama bitince roadmap'e göre eksiksizlik kontrolü yapar. Kullanıcı "roadmap'i uygula", "bu task'ı kodla", "implement et" dediğinde kullan.
---

# Task Kodlama + Sorun Bildirimi + Eksiksizlik Kontrolü

Girdi: `task-analysis` skill'inin ürettiği roadmap (veya eşdeğer bir analiz dokümanı).
Roadmap yoksa önce `task-analysis` skill'ini çalıştırıp roadmap üret, sonra bu skill'e devam et.

Bu skill üç fazdan oluşur. Fazları sırayla uygula.

---

## Faz 1 — Hazırlık

1. Roadmap'i oku; uygulama adımlarını, kabul kriterlerini (AC) ve test planını çıkar.
2. Kod tabanını incele: mimari, isimlendirme, hata yönetimi, loglama, DI, test ve migration alışkanlıkları.
   **Yeni kod mevcut kodun stilini taklit etmeli.**
3. Build ve test komutlarını tespit et (README, Makefile, package.json scripts, CI dosyaları).
4. Mevcut test durumunu kaydet (kodlamadan önce testler geçiyor mu?). Önceden kırık testleri raporda ayrı belirt.
5. Bir **Takip Listesi** oluştur (TodoWrite/Task aracı varsa onu kullan): her roadmap adımı = bir madde.

## Faz 2 — Kodlama

Her roadmap adımı için:

1. Adımın yapılacaklarını uygula. Kodlama işini `backend-coder` agent'ına devredebilirsin
   (büyük/bağımsız adımlarda paralel çalıştırmak için uygundur).
2. Adıma ait testleri yaz (roadmap'teki T-x ID'leriyle eşleştir).
3. Build + ilgili testleri çalıştır. Kırmızıysa düzelt; düzeltilemiyorsa Sorun Kaydı'na yaz.
4. Takip Listesi'nde adımı tamamlandı olarak işaretle.

### Sorun Kaydı — kodlarken ne zaman bildirilir

Aşağıdakilerden biri olduğunda **hemen** kullanıcıya kısa bir not geç ve Sorun Kaydı'na ekle.
İşi bloklamıyorsa makul bir çözümle devam et; blokluyorsa o adımı atla, sonraki bağımsız adımla devam et.

| Tür | Örnek |
|---|---|
| `EKSİK-GEREKSİNİM` | Roadmap'te hata durumunda ne döneceği yazmıyor |
| `ÇELİŞKİ` | Roadmap X diyor, mevcut kod/şema Y'ye izin vermiyor |
| `TEKNİK-ENGEL` | Gerekli kütüphane/servis/erişim yok, migration çakışıyor |
| `RİSK` | Değişiklik geriye dönük uyumluluğu bozuyor, performans sorunu riski |
| `KAPSAM-DIŞI` | Düzgün çalışması için roadmap dışı bir değişiklik gerekiyor |
| `TEKNİK-BORÇ` | Mevcut kodda fark edilen ama bu task'ta düzeltilmeyen sorun |

Kayıt formatı:
```
[TÜR] <kısa başlık>
- Nerede: <dosya:satır veya roadmap adımı>
- Sorun: ...
- Etki: ...
- Yapılan / Öneri: ...
- Bloklayıcı mı: Evet/Hayır
```

### Kodlama kuralları

- Sadece roadmap kapsamındaki değişiklikleri yap; ilgisiz refactor yapma (gerekiyorsa `KAPSAM-DIŞI` olarak bildir).
- Gizli bilgi (şifre, token, anahtar) koda gömme; mevcut konfigürasyon mekanizmasını kullan.
- Girdi doğrulama, yetkilendirme, hata yönetimi ve loglamayı mevcut desenlere uygun ekle.
- Veritabanı değişikliklerini migration ile yap; geri alınabilir (down/rollback) olsun.
- Testi atlama, devre dışı bırakma veya "geçsin diye" assertion zayıflatma yapma.

## Faz 3 — Eksiksizlik Kontrolü

Kodlama bitince roadmap'e karşı sistematik kontrol yap:

1. **Adım kontrolü:** Her uygulama adımı ve alt maddesi yapıldı mı?
2. **Gereksinim izlenebilirliği:** Her FR/NFR → hangi dosyada karşılandı → hangi testle doğrulandı?
3. **Kabul kriterleri:** Her AC için kanıt (test adı, komut çıktısı veya kod referansı).
4. **Test planı:** Roadmap'teki her T-x yazıldı mı ve geçiyor mu?
5. **Uç durumlar ve güvenlik listesi:** Roadmap'in 8. ve 9. bölümlerindeki her madde ele alındı mı?
6. **Temizlik:** TODO/FIXME, debug log, yorum satırına alınmış kod, kullanılmayan import kalmadı mı?
7. **Tam build + tüm test paketi** son kez çalıştırılır; sonuç rapora yazılır.

Eksik bulunursa: tamamla ve kontrolü tekrarla. Tamamlanamıyorsa rapora `EKSİK` olarak yaz.

## Son rapor (şablon)

```markdown
# Kodlama Raporu: <Task adı>

## Durum: ✅ Tamamlandı | ⚠️ Kısmen tamamlandı | ❌ Bloklandı

## Yapılan değişiklikler
| Dosya | Değişiklik | Roadmap adımı |
|---|---|---|

## Eksiksizlik matrisi
| Gereksinim / AC | Karşılandığı yer | Test | Durum (✅/⚠️/❌) |
|---|---|---|---|

## Test sonuçları
- Komut: `...`
- Sonuç: X geçti / Y kaldı / Z atlandı
- Önceden kırık olan testler: ...

## Sorun Kaydı
<Faz 2'de kaydedilen tüm sorunlar, bloklayıcılar en üstte>

## Eksik kalanlar
- ... (yoksa "Yok")

## Sonraki adım
Kod review ve test için `task-review` skill'ini çalıştır.
```
