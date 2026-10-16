---
name: task-analysis
description: Ham bir task'ı (Jira/issue metni, kullanıcı isteği, bug raporu) kodlamaya hazır hale getirir ve doğrudan adım adım uygulanabilir bir geliştirme roadmap'i üretir. Kullanıcı bir task, issue, feature isteği veya "bunu analiz et / roadmap çıkar / nasıl yaparız" dediğinde kullan.
---

# Task Analizi → Geliştirme Roadmap'i

Bu skill'in tek bir çıktısı vardır: **aşağıdaki şablona birebir uyan bir Roadmap dokümanı.**
Kullanıcıdan ek bilgi, onay veya açıklama BEKLEME. Task ne kadar eksik olursa olsun roadmap'i
üret; bilinmeyenleri "Varsayımlar" ve "Açık Sorular" bölümlerine yaz ve işe devam et.

## Temel kurallar

1. **Soru sorup durma.** Belirsizlik varsa en makul varsayımı yap, `[VARSAYIM]` diye işaretle, roadmap'e devam et.
2. **Kod tabanı varsa önce oku.** Çalışma dizininde proje varsa şunları incele: dizin yapısı, kullanılan dil/framework,
   paket dosyaları (package.json, pom.xml, build.gradle, go.mod, requirements.txt, pyproject.toml, *.csproj vb.),
   mevcut katman yapısı (controller/service/repository), test klasörü, migration klasörü, konfigürasyon.
   Roadmap'teki dosya yollarını ve isimlendirmeyi mevcut koda göre yaz. Kod tabanı yoksa stack'ten bağımsız yaz
   ve varsayılan stack'i "Varsayımlar"a ekle.
3. **Stack bağımsızlığı.** Belirli bir dile bağlı kalma; proje neyi kullanıyorsa onun terimleriyle yaz.
4. **Her adım tek başına uygulanabilir olmalı.** Bir geliştirici (veya başka bir AI) sadece o adımı okuyarak işi yapabilmeli.
5. **Ölçülebilir kabul kriterleri.** "Çalışmalı" gibi muğlak ifade yok; "X isteği Y durumunda 409 döner" gibi test edilebilir yaz.
6. Araştırma gerekiyorsa (bilinmeyen kütüphane, harici API, standart) `researcher` agent'ını kullanabilirsin; yoksa bilgini kullan.

## Analiz süreci (içeride yap, çıktıya sadece sonucu yaz)

1. **Anla:** Task'ın iş amacı ne? Kim kullanacak? Hangi problem çözülüyor?
2. **Ayrıştır:** Fonksiyonel gereksinimler, fonksiyonel olmayan gereksinimler (performans, güvenlik, ölçeklenebilirlik,
   gözlemlenebilirlik), kısıtlar.
3. **Etki alanı:** Hangi modüller, endpoint'ler, tablolar, kuyruklar, servisler, konfigürasyonlar etkilenir?
4. **Veri modeli ve sözleşmeler:** Yeni/değişen entity, tablo, alan, index, DTO, API sözleşmesi, event şeması.
5. **Uç durumlar:** Boş/null girdi, yetkisiz erişim, eşzamanlılık, idempotency, büyük veri, zaman aşımı,
   harici servis hatası, geriye dönük uyumluluk.
6. **Riskler:** Teknik riskler, veri kaybı/migration riski, performans riski, güvenlik riski.
7. **Parçala:** İşi bağımlılık sırasına göre küçük, test edilebilir adımlara böl (her adım ideal olarak birkaç saatlik iş).

## Çıktı şablonu (birebir bu başlıklarla üret)

```markdown
# Roadmap: <Task'ın kısa adı>

## 1. Özet
<2-4 cümle: ne yapılacak, neden, sonuç ne olacak>

## 2. Kapsam
**Kapsam içi:**
- ...
**Kapsam dışı:**
- ...

## 3. Gereksinimler
### Fonksiyonel
- FR-1: ...
- FR-2: ...
### Fonksiyonel olmayan
- NFR-1: (performans / güvenlik / loglama / ölçeklenebilirlik) ...

## 4. Varsayımlar
- [VARSAYIM] ...

## 5. Teknik Tasarım
### Etkilenen bileşenler
| Bileşen / Dosya | Değişiklik türü (yeni/değişiklik/silme) | Açıklama |
|---|---|---|
### Veri modeli
<yeni/değişen tablo, alan, index, migration; yoksa "Değişiklik yok">
### API / Sözleşmeler
| Method | Path / Event | İstek | Yanıt | Hata kodları |
|---|---|---|---|---|
### Akış
<adım adım istek/iş akışı; gerekiyorsa kısa sıralı liste>

## 6. Uygulama Adımları
### Adım 1: <başlık>
- **Amaç:** ...
- **Yapılacaklar:**
  - [ ] ...
- **Dosyalar:** `path/to/file`
- **Bağımlılık:** Yok | Adım N
- **Tamamlanma kriteri:** ...
### Adım 2: ...
(… bağımlılık sırasına göre devam)

## 7. Test Planı
| ID | Tür (unit/integration/e2e) | Senaryo | Beklenen sonuç |
|---|---|---|---|
| T-1 | unit | ... | ... |
(Her FR için en az bir pozitif ve bir negatif test; her uç durum için bir test)

## 8. Uç Durumlar ve Hata Yönetimi
- ...

## 9. Güvenlik Kontrol Listesi
- [ ] Kimlik doğrulama / yetkilendirme
- [ ] Girdi doğrulama
- [ ] Hassas veri (loglama, maskeleme, şifreleme)
- [ ] (task'a özel maddeler)

## 10. Riskler
| Risk | Etki (Düşük/Orta/Yüksek) | Önlem |
|---|---|---|

## 11. Kabul Kriterleri
- [ ] AC-1: ...
- [ ] AC-2: ...

## 12. Açık Sorular
- ... (yanıt beklenmeden ilerlenebilir; seçilen varsayım 4. bölümde)

## 13. Tahmini Efor
| Adım | Tahmin |
|---|---|
| Toplam | ... |
```

## Kalite kontrol (çıktıyı vermeden önce)

- Her FR en az bir uygulama adımına ve bir teste bağlanıyor mu?
- Her adımın tamamlanma kriteri var mı?
- Adımlar bağımlılık sırasına göre mi?
- Kabul kriterleri test edilebilir mi?
- Hiçbir bölüm boş değil mi? (İlgisizse "Uygulanamaz — <neden>" yaz)

Roadmap tamamlandığında kodlamaya geçmek için `task-implementation` skill'i bu dokümanı girdi olarak kullanır.
Kullanıcı bir dosya istemişse roadmap'i `docs/roadmap/<task-adi>.md` olarak da kaydet.
