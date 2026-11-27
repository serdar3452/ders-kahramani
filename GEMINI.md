# DERS KAHRAMANI - GELİŞTİRME & MİMARİ KURALLARI (GEMINI.md)

Bu dosya, projenin mimari bütünlüğünü korumak, performansı optimize etmek ve daha önce yaşanan hataların (tema şişmesi, karakter bozulması, eksik senkronizasyon vb.) kesinlikle tekrarlanmaması için uyulması zorunlu kuralları içerir. Antigravity asistanı her oturumda bu kurallara uymakla yükümlüdür.

---

## 1. MİMARİ KURAL: AYRIK BULUT YAPISI (DECOUPLED CLOUD)
- **Tema Dosyası Sadece Motor/Kabuktur:** `ders-kahramani-sinirsiz-tema.xml` asla ve hiçbir koşulda soru veya müfredat verisi İÇERMEZ. Boyutu ~318 KB civarında sabit kalmalıdır.
- **Kesinlikle Soru Verisi Bundle Edilmez:** `build-blogger-theme-cloud.ps1` derleme betiğine `higher-ed.js`, `grade-*.js` gibi müfredat veri dosyaları EKLENMEZ. Tema 100.000 soru olsa da tek bir soru dahi barındırmaz.
- **İçerik CDN'den Dinamik Yüklenir:** Tüm ders verileri GitHub CDN üzerinden (`https://raw.githubusercontent.com/serdar3452/ders-kahramani/main/${courseId}.json`) istemcinin tarayıcısına ihtiyaç anında dinamik olarak çekilir.

---

## 2. İÇERİK EKLEME VE DAĞITIM PROTOKOLÜ (DEPLOYMENT)
Yeni bir ders veya ünite eklendiğinde **Tek Kaynak JSON + Tek Komut** akışı işletilir (Headless Chrome gerekmez):
1. **Tek Gerçek Kaynak:** Ders verisi doğrudan `data/cdn-courses/<courseId>.json` dosyasına standart UTF-8 (BOM'suz) JSON olarak yazılır/güncellenir.
2. **Tek Komutlu Doğrulama + Manifest + Atomik Yayın:**
   ```powershell
   .\publish.ps1 -Course <courseId>
   ```
   Bu komut sırasıyla `tools/validate-courses.ps1` (mojibake + şema + cevap bütünlüğü), `manifest.json` SHA-256 sürüm güncellemesi ve GitHub'a **atomik tek commit** gönderimini yapar. Token Windows ortam değişkeninden (`DK_GITHUB_TOKEN`) okunur; betiklere asla token yazılmaz.
3. *(Eski JS müfredat dosyaları düzenlenirse önce `.\export-cdn-courses.ps1`, ardından `.\publish.ps1 -Course <courseId>` çalıştırılır.)*

---

## 3. KARAKTER KODLAMASI (UTF-8 / MOJIBAKE ÖNLEME)
- **Windows PowerShell Tuzağı:** Windows PowerShell 5.1 varsayılan olarak ANSI (Windows-1254) okuma yapar. Betiklerde asla yalın `Get-Content` veya `Set-Content` kullanılmaz.
- **Standart UTF-8 Kuralı:** Dosya okuma ve yazma işlemlerinde daima UTF-8 (BOM'suz) kullanılır:
  ```powershell
  [System.IO.File]::ReadAllText($filePath, [System.Text.Encoding]::UTF8)
  [System.IO.File]::WriteAllText($filePath, $text, [System.Text.UTF8Encoding]::new($false))
  ```
- **Karakter Bütünlüğü Kontrolü:** Dosya kaydedildikten sonra Türkçe karakterlerin (ğ, Ğ, ı, İ, ş, Ş, ç, Ç, ö, Ö, ü, Ü) ve emojilerin bozulmadığı (`iÅŸletme` gibi mojibake olmadığı) mutlaka doğrulanmalıdır.

---

## 4. ÖNBELLEK VE SÜRÜM YÖNETİMİ (CACHE BUSTING)
- İçerik şeması veya CDN verilerinde köklü bir düzeltme/güncelleme yapıldığında, kullanıcıların tarayıcılarındaki eski veya bozuk önbelleğin temizlenmesi için `js/content-loader.js` dosyasındaki `CACHE_PREFIX` (örneğin `dk_cache_v6_course_` -> `dk_cache_v7_course_`) bir üst sürüme artırılır ve tema yeniden derlenir.

---

## 5. DERS VE SORU VERİ STANDARDI
Her ünitede aşağıdaki 5 pedagojik bileşen eksiksiz yer almalıdır:
1. `quizQuestions`: 5 seçenekli (A-E), ayrıntılı `solution` (çözüm açıklaması) içeren Boss sınavı soruları.
2. `openEndedQuestions`: Puanlama rubriği (`rubric`) ve model cevabı (`idealAnswer`) olan klasik açık uçlu sorular.
3. `trueFalseQuestions`: Açıklamalı (`explanation`) Doğru/Yanlış soruları.
4. `flashcards`: Ön yüz (`term`/`front`) ve arka yüz (`def`/`back`) bilgi kartları.
5. `matchingPairs`: Kavram-açıklama eşleştirme çiftleri.
