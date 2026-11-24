# GÖREV: "Sürgü" — 4×4 taş kaydırma bulmacası
# Öğretici + 80 seviye (31–80 arası oyunun kendi ürettiği tahtalar) + altın + kozmetik market
# Platform: mobil tarayıcı (herkese açık link olarak yayınlanacak web sitesi)

Bu işi baştan sona sen yapacaksın. Plan yazıp durma. Kodu yaz, çalıştır, test et, bulduğun hataları düzelt ve sonunda açıp oynayabileceğim çıktıyı teslim et.
Bu metin tek başına yeterli. Referans oyunu veya videoları açman gerekmiyor. Aşağıdaki kurallar bağlayıcıdır.
"§" bu belgenin bölümlerini gösterir. Seviyeler S01…S80, öğretici tahtaları Ö1…Ö3 diye anılır.

§0 ÇALIŞMA ALANI VE ÖNCELİK

Önce çalışma alanını incele:
- Önceki bir prototip varsa önce onu oku ve çalıştır. Kural motorunu, durum geçişlerini ve seviye geçişini bu belgeyle karşılaştır. Yanlışları düzelt, işe yarayan görsel kodu koru.
- Bilinen hata: Oyuncu bilerek yanlış oynadığında önceki prototip bazen seviyeyi kendiliğinden geçiyordu. Kök nedenini bul ve raporda yaz. Muhtemel nedenler: kazanmanın animasyon bitişine, Levha sayısının sıfırlanmasına veya eski bir oturumdan kalan zamanlayıcıya bağlanması.
- Proje yoksa depo kökünde sıfırdan kur. Bu PROMPT.md dosyasını silme.

Öncelik sırası. Hepsini yap. Süre yetmezse en alttan kes ve raporda belirt.
- P0: Motor (§3–§5), öğretici, S01–S30, gecikmeli çıkmaz uyarısı, kayıt, motor ve arayüz testleri, site/ klasörü, tek dosyalık yedek ve GitHub Pages yayın iş akışı (§15).
- P1: Seviye üretici (S31–S80), altın ekonomisi, market ve kozmetikler, service worker, internetsiz oynama, kurulum ve paylaş düğmeleri.
- P2: Sonsuz Atölye modu, ilerleme taşıma, ek görsel cilalar.

§1 REFERANS VE KANIT DÜZEYİ
Esin kaynağımız, GTA VI içindeki kurgusal bir video oyununun kulübelerinde, masalarda duran, fiziksel görünümlü kaydırma tahtaları. O oyundan hiçbir isim, model, doku, logo, ekran görüntüsü veya ses kullanma. Yalnızca mekaniğin özünü kendi özgün tasarımımızla yeniden kuruyoruz.

[K] = Birden çok yazılı rehberde geçen, referansa dayanan kural.
[T] = Referansta belirsiz veya doğrulanmamış; bu prototip için verilmiş tasarım kararı.

Referans hakkında bildiklerimiz:
- [K] Tahta 4×4, satranç tahtası gibi iki renkli ve büyük karelidir. Mum ışığındaki bir çalışma masasında durur. Oyunun geneli siyah-beyazdır.
- [K] Üç taş türü vardır: hareket etmeyen alçak düz parçalar, sivri tepeli piramit parçalar ve üç kat kareden oluşan daha uzun parçalar.
- [K] Hareket: Hareketli bir taş seçilir ve dört yönden birine itilir. Taş düz bir çizgide, tahta kenarına veya başka bir taşa çarpana kadar kayar. Yarı yolda durdurulamaz.
- [K] Yok etme, taşın DURDUĞU karenin ortogonal komşularında olur:
  * Sivri, komşu düz parçaları ve komşu uzun parçaları yok eder.
  * Uzun parça komşu sivriyi yok eder. Yani sivri ile uzun parça yan yana gelince ikisi birden gider.
  * Uzun parça düz parçaya bir şey yapamaz. Düz parçayı yalnızca sivri yok eder.
- [K] Kazanma: Tahtada hiç taş kalmaması.
- [K] Hamle sınırı, süre, hedef kare, eşleştirme veya kenardan düşme yoktur. Yanlış bir hamle tahtayı çözülemez hâle getirebilir; o zaman oyuncu tahtayı baştan kurar.
- [K] Bazı tahtalarda hareketli taş sayısı artar (rehberlerde 6. taşa kadar numaralanmış parçalar var). Dizilimler oyundan oyuna değişir.
- Doğrulama: Bu kural modeli, referanstaki ilk tahtanın yayımlanmış 3 hamlelik çözümünü ortogonal komşulukla yeniden üretiyor. Çapraz (8 komşulu) model bu çözümü üretemiyor.
- Belirsiz kalan noktalar ve bu prototipteki kararlar:
  * İki sivri yan yana gelince: Etkisiz [T].
  * Kayarken geçilen karelerin yanındaki taşlar: Etkilenmez, yalnızca durulan yer sayılır [T]. Kaynakların çoğu da böyle diyor.
  * Hamle kalmayınca tahtanın kendiliğinden sıfırlanması: Bunu tek ve doğrulanmamış bir kaynak söylüyor. Bizde otomatik sıfırlama yok; bunun yerine gecikmeli uyarı var (§5).

§2 KOORDİNATLAR VE YAZIM
- Oyuncu tahtanın bir kenarında oturur. Sütunlar soldan sağa A, B, C, D. Sıralar oyuncuya en yakın olan 1'den en uzaktaki 4'e kadar gider. Hücre adı örneği: "B3".
- Yönler:
  * ↑ YUKARI = oyuncudan uzaklaşır (sıra artar).
  * ↓ AŞAĞI = oyuncuya yaklaşır.
  * ← SOL = A'ya doğru.
  * → SAĞ = D'ye doğru.
- Izgaralar oyuncunun gördüğü gibi yazıldı: En üst satır 4. sıra (arka), en alt satır 1. sıra (ön).
- Harfler: S = Sivri, K = Kule, L = Levha, . = boş.
- Dizilim dizesi: 16 karakter. Sırasıyla 4. sıra A→D, 3. sıra A→D, 2. sıra A→D, 1. sıra A→D.
  Örnek "....L......SLK.S" şu tahtadır:
    4 . . . .
    3 L . . .
    2 . . . S
    1 L K . S
      A B C D
- Hamle yazımı: "B1↑" = B1'deki taşı YUKARI it.
- Kodda tek bir iç sözleşme seç ve her yerde kullan. Üretici için önerilen: hücre başına 2 bit, toplam 32 bitlik tek bir uint32.

§3 TAŞ TÜRLERİ
- Sivri (S):
  * Hareket: 4 yöne kayar.
  * Durduğunda komşu Levha'ları yok eder.
  * Komşu Kule'yi yok eder ve kendisi de yok olur.
- Kule (K):
  * Hareket: 4 yöne kayar.
  * Durduğunda komşu Sivri'leri yok eder ve kendisi de yok olur.
  * Levha'ya etkisi yoktur.
- Levha (L):
  * Hareket etmez.
  * Yalnızca komşusunda duran bir Sivri onu yok eder.
- Etkisiz çiftler: Sivri–Sivri [T], Kule–Kule, Kule–Levha, Levha–Levha.

§4 BİR HAMLENİN ÇÖZÜLME SIRASI (UYULMASI ZORUNLU)
tryMove(durum, hücre, yön):
  1. Durum OYNANIYOR değilse reddet. Hiçbir şey değişmez.
  2. Hücrede Sivri veya Kule yoksa reddet. Hiçbir şey değişmez.
  3. Yön boyunca komşu hücre tahtanın içinde ve boş olduğu sürece ilerle. Mesafe 0 çıkarsa reddet. Bu "geçersiz girdi"dir: taşlar, hamle sayısı ve durum değişmez.
  4. Taşı durduğu hücreye koy. Bu taşa "itilen taş" diyoruz.
  5. Yalnızca durduğu hücrenin 4 ortogonal komşusuna bak. Çapraz komşular ve yol üzerindeki kareler sayılmaz.
     - İtilen taş Sivri ise: Komşu Levha listeye eklenir. Komşu Kule varsa o Kule de itilen taş da listeye eklenir.
     - İtilen taş Kule ise: Komşu Sivri varsa o Sivri de itilen taş da listeye eklenir.
  6. Listedeki her şeyi AYNI ANDA kaldır. Zincirleme yok. İtilen taş yok olsa bile 5. adımda bulduğu bütün etkiler geçerlidir.
  7. Hamle sayısını bir artır ve durumu yeniden hesapla (§5).
  8. Şunu döndür: { nereden, nereye, mesafe, yok olan taşların id'leri, yeni durum }.
Değişmezler (assert ile test et):
  - Başlangıç dizilimlerinde Sivri–Levha veya Sivri–Kule olarak yan yana duran çift yoktur.
  - Her hamleden sonra tahtada böyle bir çift kalmaz.
Kurallar 8 simetriye (döndürme ve yansıma) göre değişmez. Üretici, aynı tahtanın kopyalarını ayıklamak için bunu kullanır.

§5 OYUN DURUMU VE GECİKMELİ ÇIKMAZ UYARISI

Görünür durumlar yalnızca iki tanedir:
- KAZANILDI: Tahtada hiç taş yok. Tek kazanma koşulu budur.
- OYNANIYOR: Geri kalan her durum.
Kayıp durumu ve kayıp ekranı yoktur.

İç bayraklar (yalnızca uyarı politikası kullanır, arayüze doğrudan yansımaz):
- hamleVar: En az bir geçerli hamle var mı?
- cozulebilir: EVET / HAYIR / BILINMIYOR. Her hamleden sonra bir Web Worker'da hesaplanır:
  * Önce ucuz zorunlu koşullara bak. Şunlardan biri doğruysa sonuç HAYIR:
    - Levha var ama Sivri yok.
    - Sivri var ama Kule yok.
    - Kule var ama Sivri yok.
  * Bunlar sonuç vermezse BFS çalıştır (durum sınırı 150 bin). Sınır aşılırsa sonuç BILINMIYOR.

Oyun, tahtanın çözülemez hâle geldiğini HEMEN söylemez. Ne yazı, ne ses, ne renk değişimi olur. Oyuncu uğraşırken kendisi fark etsin. Fark etmezse belli bir süre veya hamle sonra yumuşak bir uyarı çıkar:
- Saymaya başlama anı: cozulebilir'in HAYIR olduğu hamle.
- Uyarı, aşağıdaki eşiklerden hangisi önce dolarsa o zaman çıkar:
  * S01–S10: 4 hamle veya 30 sn aktif süre.
  * S11–S30: 6 hamle veya 45 sn.
  * S31 ve sonrası ile Sonsuz Atölye: 8 hamle veya 60 sn.
- Aktif süre yalnızca sayfa görünürken ve oyun ekranı açıkken sayılır.
- Hiç geçerli hamle kalmadıysa (ör. yalnızca Levha kaldı) eşik şudur: 3 başarısız deneme veya 10 sn.
- Uyarı metni: "Bu dizilimden tahta temizlenemiyor gibi görünüyor." Altında iki düğme: [Baştan kur] [Biraz daha dene].
  * "Biraz daha dene" seçilirse uyarı ancak eşik yeniden dolunca tekrar çıkar.
  * Bir denemede en fazla 2 uyarı gösterilir.
  * Baştan kurma bütün sayaçları sıfırlar.
- Çözülebilir bir tahtada uyarı ASLA çıkmaz. Sonuç BILINMIYOR ise de uyarı çıkmaz.
- Ayarlar: "Çıkmaz uyarısı: Geç (varsayılan) / Kapalı".
- Uyarı ceza değildir. Otomatik sıfırlama, otomatik geçiş veya altın kaybı yoktur.

Durum ve seviye geçişi:
- Durum yalnızca motordaki tahtadan hesaplanır. Animasyonun bitmesi, Levha sayısının sıfırlanması, zamanlayıcılar veya hamle sayısı durumu tek başına değiştiremez.
- KAZANILDI'ya geçildiği an:
  * Seviye tamamlandı olarak işaretlenir.
  * Ödül bir kez verilir (§8).
  * Sonraki seviyenin kilidi açılır.
  * "Sonraki tahta" düğmesi görünür.
  Sonraki seviye yalnızca bu düğmeyle yüklenir. Otomatik geçiş zamanlayıcısı yoktur.
- Yeniden Başlat animasyon sırasında da her zaman kullanılabilir. Tahtayı başlangıç dizilimine, hamle sayısını 0'a döndürür.
- Her seviye yüklemesi ve her yeniden başlatma yeni bir sessionId üretir. Animasyon, zamanlayıcı, ses, Worker yanıtı ve uyarı geri çağrıları, iş yapmadan önce kendi sessionId'lerinin hâlâ geçerli olduğunu kontrol eder. Eski bir oturumdan gelen geri çağrı yeni tahtaya dokunamaz, seviye açamaz, ödül veremez.
- Hamle animasyonu sürerken gelen yeni hamle girdileri yok sayılır, kuyruğa alınmaz [T]. Yeniden Başlat ve menü düğmeleri bu kilitten muaftır.

§6 ZORLUK ÖLÇÜTLERİ (TANIMLAR; TESTLER BUNLARI KULLANIR)
Her ölçüt, başlangıçtan erişilebilen bütün durumların grafiği üzerinde hesaplanır. Bu grafiğe başlangıç durumu ve boş tahta da dahildir.
- E: En kısa çözümün hamle sayısı.
- Ç: Farklı en kısa hamle dizilerinin sayısı. 10^6'da kesilir.
- T: İlk hamle tuzak oranı = (sonucu çözülemeyen ilk hamle sayısı) / (geçerli ilk hamle sayısı).
- Ö: Ölü durum oranı = (erişilebilen durumlar içinde çözülemeyenlerin sayısı) / (erişilebilen bütün durumlar).
- G: Güvenli hamle oranı, kanonik en kısa çözüm boyunca ortalama alınır. Her durumda (çözülebilir sonuç veren hamle sayısı) / (geçerli hamle sayısı) hesaplanır.
  * Kanonik çözüm: Her adımda, en kısa çözüme götüren ilk hamle seçilir.
  * "İlk hamle" sıralaması: Hücreler 4. sıradan 1. sıraya, her sırada A'dan D'ye; aynı hücrede yönler ↑ ↓ ← →.
- F (feda zorunlu): Feda hamlesi hiç yapmadan çözüm yoksa EVET.
  * Feda hamlesi: Sonunda Sivri sayısı azalan ve tahtada hâlâ en az bir Levha kalan hamle.
  * Yani oyuncu, bütün Levha'lar temizlenmeden bir Sivri'den vazgeçmek zorundadır.
- P: Toplam taş sayısı.
- H: Hareketli taş sayısı (S + K).
- sessiz: Kanonik çözümde hiçbir taşı yok etmeyen hamlelerin sayısı.
- Zorluk puanı: D = E + 4T + 4Ö + 3(1−G) + 2F + 0,3H. Sıralama ve rapor için kullanılır.
Not: Taş sayısını artırmak tek başına zorluk getirmez; rastgele dizilimlerde çok taş genelde işi kolaylaştırır. Zorluk T, Ö, G ve F ile ölçülür.

§7 İÇERİK

7.1 Akış:
- İlk açılışta öğretici otomatik başlar ("Geç" düğmesi var). Sonra ana menü açılır: Devam, Seviyeler, Market, Ayarlar, Öğretici.
- Seviyeler sırayla açılır. Bir seviyeyi bitirmek bir sonrakinin kilidini açar.
- Seviye seçim ekranı:
  * 80 kart.
  * Her kartta kilit, tamamlandı işareti, en kısa hamle mührü ve altın ödülü.
  * S31–S80 kartlarında "Atölye: bu tahtayı oyun kendisi yonttu" etiketi.

7.2 ÖĞRETİCİ (3 tahta, rehberli)
Rehberli oynanış:
- Her tahtada yalnızca gösterilen hamle kabul edilir.
- Başka bir girdi küçük bir sallantı yapar ve el/ok ipucunu yeniden gösterir.
- Kısa, tek satırlık metinler kullan.
- Öğretici bitince bir kereye mahsus +5 altın hoş geldin hediyesi verilir.

Ö1 "Kaydır" — dizilim "........S..K...."
    4 . . . .
    3 . . . .
    2 S . . K
    1 . . . .
  Hamle: A2→
  Metin önce: "Taşı parmağınla kaydır. Bir şeye çarpana kadar gider."
  Metin sonra: "Sivri ile Kule yan yana durunca ikisi de yok olur."

Ö2 "Levha" — dizilim ".....L.....K.S.."
    4 . . . .
    3 . L . .
    2 . . . K
    1 . S . .
  Hamleler: B1↑ (Sivri B2'de durur, B3'teki Levha kırılır), sonra D2← (Kule C2'de durur, Sivri ile birlikte gider).
  Metin: "Levha hareket etmez. Onu yalnızca yanında duran Sivri kırar."

Ö3 "Durdurucu" — dizilim ".S......LKL....."
    4 . S . .
    3 . . . .
    2 L K L .
    1 . . . .
  Hamleler: B2↓ (Kule B1'e iner; Levha'ların yanında durması onlara bir şey yapmaz), sonra B4↓ (Sivri Kule'ye çarpıp B2'de durur; iki Levha, Kule ve Sivri aynı anda gider).
  Metin: "Kule Levha'yı kırmaz ama durdurucu olur. Bir hamlede birçok taş gidebilir."

7.3 SEVİYE 1–30 (SABİT VERİ, ÇÖZÜCÜYLE DOĞRULANDI)
Biçim: Kod | dizilim | E | Ç | T (tuzak/geçerli ilk hamle) | Ö | G | F | P/H | kanonik çözüm
S08, S15 ve S28 "çapa" seviyelerdir; testlerde kullanılırlar. S30, feda gerektiren ilk tahtadır (ad: "Feda").
S01 .L...K..S..L.... E3 Ç1 T1/6 Ö.02 G.94 F- 4/2 | A2→ C2↑ B3↑
S02 K..L....S....... E3 Ç2 T2/5 Ö.01 G.87 F- 3/2 | A2→ D2↑ A4→
S03 LK.L..S......... E3 Ç3 T2/6 Ö.02 G.78 F- 4/2 | C3← A3→ B4→
S04 ..S....SK......L E3 Ç2 T2/9 Ö.53 G.42 F- 4/3 | C4↓ C1→ A2→
S05 LK......K.S..... E4 Ç1 T3/9 Ö.34 G.59 F- 4/3 | B4↓ C2↑ C4← B4↓
S06 ..K.LL....S....S E5 Ç1 T3/9 Ö.35 G.74 F- 5/3 | D1← A1↑ A2→ C2↓ C4↓
S07 ..L......S.SK... E4 Ç3 T4/9 Ö.39 G.54 F- 4/3 | B2→ D2↑ D4↓ A1→
S08 ....L......SLK.S E4 Ç1 T3/5 Ö.45 G.56 F- 5/3 | B1↑ D1← D2← B4↓   (çapa: "İlk Sürgü")
S09 ..K.L...K.....S. E5 Ç2 T4/8 Ö.41 G.53 F- 4/3 | A2→ C1← C4↓ A1↑ A2→
S10 L......SK.....S. E5 Ç1 T3/9 Ö.55 G.41 F- 4/3 | C1↑ C4← B4↓ B1→ A2→
S11 L.L..K..L..S..S. E5 Ç4 T4/10 Ö.26 G.68 F- 6/3 | B3↑ D2← C1↑ C3← B4↓
S12 ...S..K..L.L..S. E5 Ç1 T5/9 Ö.29 G.68 F- 5/3 | D4← A4↓ A1→ C1→ C3↓
S13 LL.K..S.......K. E6 Ç3 T5/9 Ö.27 G.56 F- 5/3 | C3← D4← A3↑ C1↑ C4→ A4→
S14 ..S.....S..K..LL E6 Ç1 T4/8 Ö.35 G.54 F- 5/3 | A2↓ A1→ B1→ C1↑ C4→ D2↑
S15 S.L..K..LL.....S E6 Ç2 T4/7 Ö.26 G.60 F- 6/3 | D1← A4↓ A1→ D1↑ D4← B3←   (çapa: "Köşe Hesabı")
S16 ...L..S...S..L.K E6 Ç5 T4/8 Ö.33 G.47 F- 5/3 | C3↑ C4← A4↓ D1↑ A1→ D4↓
S17 ..S......SS.K..L E5 Ç2 T5/11 Ö.78 G.50 F- 5/4 | C2↑ C4→ A1→ D4↓ C1↑
S18 .S.L....L...K..S E6 Ç3 T3/6 Ö.65 G.40 F- 5/3 | B4← A4↓ D1↑ D3← A3↑ A1↑
S19 L...KK...K..S... E6 Ç1 T4/8 Ö.77 G.59 F- 5/4 | B3→ A3→ D3↑ D4← A1↑ A3→
S20 .K..S.....L.K..L E7 Ç3 T5/8 Ö.68 G.58 F- 5/3 | A3→ B4← A4↓ A2→ D3↓ D2↓ D1←
S21 LL.S..L.S...S.K. E7 Ç4 T4/8 Ö.63 G.49 F- 7/4 | A2↑ C1↑ D4↓ D1← A1↑ A3→ C2←
S22 ..SSLK....L..S.L E7 Ç3 T5/9 Ö.66 G.54 F- 7/4 | B1→ C4↓ D4↓ C1← D1← A1↑ B3↓
S23 .L.SL.K......SS. E7 Ç4 T6/10 Ö.72 G.53 F- 6/4 | C3← D4← C4↓ B1← C1← A1↑ B3↓
S24 L.L.KL.S..K...K. E8 Ç3 T5/9 Ö.65 G.48 F- 7/4 | C1← D3↓ A1↑ A2→ C2↑ D1↑ D4← B4↓
S25 .K....S..K.K.L.L E8 Ç1 T7/12 Ö.51 G.31 F- 6/4 | B2← B4↓ C3↓ C1↑ B2→ A2→ C2↓ C4↓
S26 S..SS......KLL.. E9 Ç4 T4/8 Ö.71 G.44 F- 6/4 | A4→ A3↑ C4← B4↓ D4← A4↓ B4↓ B2↓ D2←
S27 .K.SS.L..L..S... E8 Ç1 T6/10 Ö.86 G.44 F- 6/4 | A3↓ A1→ D4↓ D2← B2↓ D1← C1↑ B4↓
S28 S.L..L.LK.....SS E8 Ç1 T6/8 Ö.81 G.41 F- 7/4 | D1↑ D2↑ A4→ C4↓ C2→ D4↓ D3← A2→   (çapa: "Üçlü Düğüm")
S29 .S..K.L..S.L.S.. E8 Ç3 T8/11 Ö.80 G.52 F- 6/4 | B1→ B2→ B4↓ A3→ D2← D1← C1↑ B3↓
S30 K..S.L.SS...S.K. E10 Ç2 T8/11 Ö.95 G.43 F+ 7/6 | A4→ C1↑ A1→ D3↓ A2→ D1← C2↓ A1→ B1↑ C4↓   ("Feda")
Bu verileri bir JSON dosyasına koy ve §2'deki ayrıştırıcıyla oku.
Senin motorun bu sayıları farklı hesaplarsa veriyi değiştirme; motoru düzelt. E, Ç, T ve F tam tutmalı. Ö ve G ±0,01 toleransla tutmalı.

7.4 SEVİYE 31–80: OYUNUN KENDİ ÜRETTİĞİ TAHTALAR ("ATÖLYE")
S31'den itibaren tahtaları oyun içindeki üretici yontar.

Determinizm:
- Aynı seviye her cihazda aynı olmalı.
- Tohum: FNV-1a hash("surgu-uretici-v1:" + n) → mulberry32 PRNG.
- Math.random kullanma. Duvar saatine göre durma kararı verme. Bütün sıralamalar deterministik olsun.

Yöntem:
1. Aralığa uyan, başlangıçta yan yana etkileşen çifti olmayan rastgele bir dizilim üret.
2. Ölçütleri hesapla (§6). Durum sınırı 150 bin; aşan dizilimi reddet.
3. Mutasyonla tepe tırmanışı yap. Mutasyonlar: taş taşı, tür değiştir, taş ekle, taş çıkar.
   - Hedef cezayı 0'a indirmek:
     ceza = 3·(E'nin aralık dışındaki mesafesi) + 10·max(0, Tmin−T) + 10·max(0, Ömin−Ö) + 10·max(0, G−Gmax) + 2·max(0, ln Ç − ln Çmax) + 8·[F isteniyor ama yok]
   - Eşit cezalı adımı kabul et. 40 iyileşmesiz adımda yeniden başlat.
   - Bütçe: Seviye başına 1200 değerlendirme.
4. Başka kısıtlar:
   - Öğretici, S01–S30 ve önceki bütün seviyelerle simetri-kanonik biçimde aynı olan tahta reddedilir.
   - E en fazla 14 olabilir. Sıkıcı uzunlukta tahta istemiyoruz.
5. Bütçe biter ve ceza > 0 kalırsa şu sırayla gevşet (her biri yeni bir 1200'lük bütçeyle):
   (a) Çmax × 2
   (b) Gmax + 0,05
   (c) Ömin − 0,05 ve Tmin − 0,05
   (d) F isteğini kaldır
   Hâlâ olmazsa en iyi adayı kabul et; şartı: çözülebilir olsun ve E ≥ Emin − 1.
   Her gevşetmeyi seviye meta verisine yaz. Çözülemeyen tahta asla verilmez.

Aralık hedefleri:
| Seviye   | P   | H   | E     | T ≥  | Ö ≥  | G ≤  | Ç ≤ | F (feda)                               |
| S31–S45  | 6–8 | 4–5 | 7–10  | 0,55 | 0,55 | 0,55 | 10  | istenmez                               |
| S46–S60  | 7–9 | 4–6 | 8–11  | 0,60 | 0,65 | 0,50 | 12  | n 3'e bölünüyorsa istenir              |
| S61–S72  | 7–9 | 5–7 | 9–12  | 0,65 | 0,75 | 0,45 | 8   | n 3'e bölünüyorsa istenir              |
| S73–S80  | 7–9 | 5–7 | 10–14 | 0,70 | 0,85 | 0,45 | 4   | hepsinde istenir                       |
- Nefes tahtaları: S35, S50 ve S65 bir önceki aralığın hedeflerini kullanır ve F istenmez. (S35'in "önceki aralığı" S21–S30'dur: P 6–8, H 4–5, E 7–9, T ≥ 0,5, Ö ≥ 0,5, G ≤ 0,55, Ç ≤ 6.)
- Feda tahtaları nadir bulunur. Bir ön denemede E10, P7, H6 olan bir F tahtası yaklaşık 300 değerlendirmede bulundu. F hedeflenen seviyelerde bütçeyi 2 katına çıkar. Rapor, S46–S80 arasında en az 8 F tahtası olup olmadığını göstermeli. Daha azsa bunu yaz, ama testi düşürme.

Çalışma zamanı:
- Üretim bir Web Worker'da yapılır. Oyuncu Sn'yi açtığında Sn+1 ve Sn+2 arka planda üretilir.
- Sonuç localStorage'da önbelleğe alınır: dizilim, kanonik çözüm, ölçütler, gevşetmeler. Anahtar URETICI_SURUM + n.
- Üretici sürümü değişirse önbellek yenilenir, ama tamamlanmış seviye kayıtları korunur.
- Hazır olmayan bir seviye açılırsa "Tahta yontuluyor…" göstergesi çıkar.
- Performans: uint32 durum kodlaması ve tipli dizilerle BFS kullan. Node'da seviye başına medyan süre ≤ 3 sn, en kötü ≤ 20 sn olmalı. Aşarsan optimize et.

7.5 SONSUZ ATÖLYE (P2)
- S80 bitince açılır.
- Rastgele tohum kullanır ve kaydedilir. Hedefler S73–S80 aralığınınkiyle aynıdır.
- Ödül: 3 altın, en kısa hamlede bitirilirse +1.

§8 ALTIN
Seviye ödülleri (yalnızca ilk temizlemede):
- S01–S20: 2 altın
- S21–S45: 3 altın
- S46–S70: 4 altın
- S71–S80: 5 altın
Diğer ödüller:
- En kısa hamle bonusu: Seviye tam E hamlede temizlenirse +1 altın. Her seviye için bir kez verilir ve sonraki bir tekrarda da kazanılabilir.
- Öğretici: +5 altın, bir kez.
Kurallar:
- Tekrar oynamak, kazanılmamış bonus dışında altın vermez.
- Ödül, KAZANILDI'ya geçiş anında motorun sonucundan verilir. Animasyon bitişine bağlı değildir. Aynı galibiyet için iki kez verilmez (idempotent).
- Kazanma ekranı şunları gösterir: "+2 altın", varsa "+1 en kısa hamle", "X hamlede temizledin (en kısa: E)".
- Altın sayacı üst çubukta her ekranda görünür. Sayının yükselerek dolması gibi arayüz animasyonları serbesttir.
- Toplam: Seviyelerden en fazla yaklaşık 350 altın kazanılır. Market toplamı 525 altındır. Bütün seviyeler bitince oyuncu marketin yaklaşık üçte ikisini almış olur; gerisi Sonsuz Atölye ile tamamlanır. İlk satın alma yaklaşık S04 civarında mümkün olur.

§9 MARKET VE KOZMETİKLER
Genel ilkeler:
- Kozmetikler oyunu yalnızca görsel olarak değiştirir; oynanışı etkilemez.
- Görsel olarak zengin ama hareketsiz olsunlar: oyunlardaki silah görünümleri (skinler) gibi duran, ayrıntılı malzeme ve biçimler. Hareketli efekt veya animasyonlu shader yok.
- Hepsi kodla üretilir: geometri ve prosedürel doku. Dışarıdan dosya yok.
- İsimler ve tasarımlar özgün olsun. Başka oyunların görünüm adlarını veya desenlerini kopyalama.

Nadirlik ve fiyatlar:
- Sıradan (gri çerçeve): 10 altın
- Nadir (mavi): 20 altın (sivri.lale 25)
- Destansı (mor): 35 altın
- Efsanevi (altın rengi): 60 altın

Katalog (id — ad — nadirlik — kısa tarif):
Tahta (kare renkleri, çerçeve ve masa tonu):
  tahta.ceviz     — Ceviz ve Kemik — varsayılan, ücretsiz
  tahta.zeytin    — Zeytin ve Keten — Sıradan
  tahta.gecemavi  — Gece Mavisi ve Gümüş — Nadir
  tahta.patina    — Bakır Patina — Destansı (pas yeşili ve krem kareler, bakır çerçeve)
  tahta.mermer    — Altın Damarlı Mermer — Efsanevi (siyah mermer altın damarlı, beyaz mermer gri damarlı)
Sivri (en çok çeşit burada):
  sivri.burgulu   — Burgulu Dikili Taş — varsayılan
  sivri.cakmak    — Yontma Çakmaktaşı — Sıradan (düzensiz yüzeyli mızrak ucu)
  sivri.kristal   — Kristal Obelisk — Nadir (yarı saydam, çok yüzlü)
  sivri.lale      — Lale Tepe — Nadir, 25 (oyma lale tabanlı, sivri uçlu)
  sivri.mizrak    — Dövme Demir Mızrak — Destansı
  sivri.obsidyen  — Obsidyen Taç — Efsanevi (siyah cam piramit, altın kakma çizgiler)
Kule:
  kule.fildisi    — Fildişi Diskler — varsayılan
  kule.bambu      — Bambu Boğumları — Sıradan
  kule.fener      — Fener Katları — Nadir (sabit, hafif iç ışıma)
  kule.sikke      — Sikke Yığını — Destansı (üç kalın sikke)
  kule.pagoda     — Mermer Pagoda — Efsanevi
Levha:
  levha.sekizgen  — Bakır Pası Sekizgen — varsayılan
  levha.kil       — Pişmiş Kil Karo — Sıradan
  levha.mozaik    — Mozaik Plaka — Nadir
  levha.muhur     — Oyma Mühür — Destansı
  levha.sedef     — Abanoz ve Sedef — Efsanevi

Okunabilirlik kuralları (her görünüm uymak zorunda; test edilir):
- Sivri:
  * Tek ve belirgin bir sivri tepesi olur.
  * Yüksekliği 0,45–0,65 hücre.
  * Kat kat görünmez; Kule ile karışmamalı.
- Kule:
  * Her zaman en uzun taştır: 0,70–0,85 hücre.
  * Gözle ayrılan tam 3 katı vardır.
- Levha:
  * Yüksekliği 0,06–0,14 hücre, düz.
- Taban izi hücrenin %60–75'i.
- Kontrast: Her taş görünümünün baskın rengi, her tahta temasının iki kare rengiyle karşılaştırılır. WCAG kontrast oranı en az 1,8 olmalı. Tutmuyorsa renderer o eşleşmede taşa ince bir kontur veya kenar ışığı ekler.

Market ekranı:
- Kategori sekmeleri ve kart ızgarası.
- Her kartta: önceden render edilmiş 3B küçük görsel, ad, nadirlik şeridi, fiyat; "Sahip" veya "Kuşanıldı" rozeti.
- Karta dokunulunca ayrıntı açılır:
  * Mevcut temada küçük bir tahta üstünde büyük önizleme. Sürükleyerek döndürülebilir.
  * "Tahtada gör" seçeneği.
  * "Satın al (35)" → onay → altın düşer, ürün sahip olunanlara eklenir → "Kuşan".
- Altın yetmiyorsa düğme pasif olur ve "X altın daha lazım" yazar.
- Satın alma idempotenttir: işlem sırasında düğme kilitlenir ve çift dokunuş tek satın alma sayılır.
- Kuşanılan görünüm oyuna hemen yansır.
- Varsayılanlar ücretsizdir ve baştan sahip olunur. Satış ve iade yok.

§10 KAYIT
- localStorage anahtarı "surgu.kayit". İçerik:
  { surum: 1, altin, ogreticiBitti, hosgeldinVerildi,
    seviyeler: { n: { tamam, enKisaBonus, enIyiHamle } },
    sahip: [id], kusanilan: { tahta, sivri, kule, levha },
    ayarlar: { ses, cikmazUyarisi }, sonsuz: { tohum, temizlenen } }
- Her değişiklikten sonra yaz. Okuma ve yazmayı try/catch içine al. localStorage kullanılamıyorsa bellekte çalış.
- Bozuk kayıtta çökme olmasın: Ham veriyi "surgu.kayit.bozuk" anahtarına yedekle, temiz kayıtla başla, konsola uyarı yaz.
- Ayarlarda onaylı bir "İlerlemeyi sıfırla" seçeneği olsun.
- Üretilmiş seviyelerin önbelleği ayrı bir anahtarda tutulur (§7.4).

§11 MİMARİ
- engine.js: DOM kullanmaz, saf ve deterministiktir.
  * Taşların kalıcı id'leri olur (S1, K1, L1...) ve animasyon bu id'lerle eşleşir.
  * Fonksiyonlar: parseLevel, tryMove, getStatus, legalMoves, toGrid.
- solver.js: BFS ve §6 ölçütleri. Fonksiyonlar: solve, analyze, canonical.
- generator.js: §7.4'teki üretici. Saftır; hem Node'da hem Worker'da çalışır.
- worker.js: Çözülebilirlik sorgusu ve seviye üretimi.
- game.js: sessionId, giriş kilidi, seviye akışı, ödül, uyarı politikası. Saat dışarıdan enjekte edilebilir olmalı (testler için sahte saat).
- economy.js ve store.js: Altın, katalog, satın alma, kuşanma.
- save.js: Kayıt.
- render.js ve skins/: Sahne, animasyon, hit-test, görünüm üreticileri. Renderer motor durumunu asla kendisi değiştirmez.
- audio.js: Prosedürel sesler.
- Test kancası:
  window.__surgu = { getGrid(), getStatus(), getLevel(), getMoveCount(), getGold(), isAnimating(), isHintVisible(), screenPointOf(hücre, 'top'|'body') }
  Arayüz testleri hamleyi bu kancayla değil, gerçek pointer/touch olaylarıyla yapar. Kanca yalnızca okumak ve ekran koordinatı bulmak içindir.
- Teknoloji:
  * Tercih three.js (WebGL).
  * Paket kuramıyorsan yerelde var mı bak. Yoksa Canvas 2D üzerinde kendi perspektif izdüşümünle aynı sahneyi kur ve raporda belirt.
  * Motor, çözücü ve üretici node --test ile tarayıcısız test edilebilmeli.

§12 SAHNE: TAHTANIN KARŞISINDA OTURMA HİSSİ
Kamera:
- Perspektif kamera, oyuncunun oturduğu 1. sıra kenarının ortasından bakar.
- Tahta düzlemine yaklaşık 50–60° yukarıdan bakar; dikey görüş açısı yaklaşık 35°. Hedef nokta tahta merkezinin biraz gerisidir.
- Her yeniden boyutlandırmada uzaklığı yeniden hesapla:
  * Dikey telefonda tahta ekran genişliğinin en az %90'ını doldursun.
  * Yatay telefonda ekran yüksekliğinin en az %85'ini doldursun.
- Arka sıradaki hücreler ekranda ön sıradakilerin en az %70'i genişliğinde olsun.
- Okunabilirlik ölçümü (§14-G) açıyı belirler.

Tahta:
- Kalın bir blok olsun. Ön kenar kalınlığı hücre genişliğinin yaklaşık %30'u kadar.
- Pahlı kenarı ve ince bir çerçevesi olsun. Kareler kakma gibi hafif gömük dursun.
- Altında ahşap masa yüzeyi görünsün.
- Renkler, malzeme ve çerçeve seçili temadan gelir.

Malzeme:
- Doku dosyası yok; damar ve benek dokularını kodla üret (canvas gürültüsü).
- Pürüzlülük farkları ve hafif bir bump/normal haritası fiziksel his versin.

Işık:
- Sıcak, mum ışığına benzeyen bir ana ışık (arka-sol üstten), soğuk ve zayıf bir dolgu ışığı, hafif bir ortam ışığı.
- Yumuşak gölge haritası olsun. Her taşın altında yumuşak, koyu bir temas gölgesi olsun.
- Ana ışığın şiddeti ±%4 kadar hafifçe titresin.

Varsayılan taşlar:
- Sivri: Hafif burgulu, dört yüzlü dar bir dikili taş. Koyu lacivert-siyah, tepesinde pirinç bir uç. Yükseklik ≈ 0,55 hücre.
- Kule: Yukarı doğru küçülen üç fildişi disk, katlar arasında koyu oluklar. Yükseklik ≈ 0,75 hücre.
- Levha: Alçak sekizgen plaka, bakır pası yeşili, üstünde oyma bir dalga simgesi. Yükseklik ≈ 0,1 hücre.

Hareket animasyonu:
- Kayma: Hücre başına yaklaşık 90 ms, ardından kısa bir yerleşme hareketi (ease-out). Hafif bir sürtünme izi bıraksın.
- Taşın çıktığı kare yaklaşık 400 ms boyunca soluk bir çerçeveyle işaretlensin. Durduğu karede küçük bir "tok" sarsıntısı olsun.
- Yok olma: Etkilenen taşlar yaklaşık 120 ms vurgulansın, sonra yaklaşık 300 ms içinde tahtaya gömülüp parçalanarak kaybolsun.
- Bir hamlenin toplam animasyonu en fazla 800 ms sürsün.

Seçim:
- Seçilen taş hafifçe yükselsin ve tabanında bir halka görünsün.
- Yalnızca en az 1 kare gidebileceği yönlerde küçük oklar görünsün.
- Varış yerinin veya yok olacak taşların önizlemesini gösterme.

Sesler:
- Ses dosyası yok; WebAudio ile özgün prosedürel sesler:
  * Kayma: Filtreli gürültü, mesafe uzadıkça uzar.
  * Durma: Tok bir ahşap vuruşu.
  * Yok olma: Kısa bir ufalanma.
  * Kazanma: Kısa ve yumuşak bir akor.
  * Satın alma: Sikke şıngırtısı.
- Ses kapatma düğmesi olsun. Ses bağlamını ilk dokunuşta aç.

İsimler:
- Oyunun adı "Sürgü".
- GTA, Rockstar, referans oyunun adı veya ona benzeyen Latince bir isim oyunun içinde hiçbir yerde geçmesin.

§13 KONTROLLER VE EKRANLAR
Giriş yolları:
- Dokunmatik: Bir taşın üzerinde başlayan kaydırma o taşı o yöne iter.
- Alternatif: Taşa dokunup seç, sonra ekranın herhangi bir yerinde kaydır.
- Masaüstü: Tıklayıp seç, sonra ok tuşları veya WASD.

Dokunma tespiti (hit-test):
- Dokunulan noktadan bir ışın gönder. Önce taşların çarpışma gövdelerine bak (görsel silueti kaplayan basit silindir ya da kutu), sonra tahta düzlemine.
- Parmağın ekranda gördüğü taş seçilir:
  * Öndeki uzun bir taşın arka kareye taşan tepesine dokunmak öndeki taşı seçer.
  * Arka taşın görünen kısmına dokunmak arka taşı seçer.
- Boş bir kareye dokunmak seçimi kaldırır.
- Levha'yı itmeye çalışmak küçük bir "hareket etmez" sallantısı gösterir ve başarısız deneme sayılır (§5).
- Çarpışma gövdeleri kuşanılan görünümün boyutuna göre güncellenir.

Yön tespiti:
- Kaydırmanın başlangıç ve bitiş noktalarını tahta düzlemine izdüşür. Tahta koordinatlarındaki farkın baskın eksenini kullan.
- Eşik: en az 0,3 hücre ve en az 24 px.
- İki eksendeki fark birbirine %15'ten yakınsa hareketi belirsiz say ve yok say.
- Bitiş noktası tahta düzleminin dışına düşerse ekrandaki vektörü kamera eksenlerine göre tahta yönüne çevir.

Sayfa:
- touch-action: none kullan; kaydırırken sayfa kaymasın veya yakınlaşmasın.
- Ekran yönü değişimine ve güvenli alanlara uyum sağla.
- Piksel oranını (DPR) en fazla 2 tut. Orta seviye telefonda akıcı çalışsın; gölge haritası yaklaşık 1024 olsun.

Ekranlar:
- Ana menü: Arka planda masa ve tahta görünür.
- Seviye seçimi.
- Oyun ekranı. Üst çubukta: seviye kodu, altın, hamle sayısı, Yeniden Başlat, menü.
- Market.
- Ayarlar: ses, çıkmaz uyarısı, ana ekrana ekle, ilerlemeyi dışa/içe aktar, ilerlemeyi sıfırla.
- Öğretici.
- Kazanma katmanı: ödüller, "Sonraki tahta", "Seviyeler".

§14 TESTLER
Kurallar:
- Yalnızca kodu okuyarak "geçer" deme.
- Çalıştırdığın her testin komutunu ve çıktı özetini raporla.
- Çalıştıramadığın bir kontrolü "ÇALIŞTIRILAMADI: <neden>" diye açıkça yaz ve yapabildiğin en yakın kontrolü yap. Örneğin sayfaya ?selftest=1 ile sentetik pointer olayları gönderen bir öz-test ekle.

A) MOTOR (node --test)
Her test şu yapıdadır: başlangıç dizilimi → girdi → beklenen son dizilim → beklenen durum.

A1 Kazanma
  Başlangıç: S08.
  Girdi ve her adımdan sonraki beklenen dizilim:
  - B1↑ (hiçbir şey yok olmaz):
      4 . K . .
      3 L . . .
      2 . . . S
      1 L . . S
  - D1← (Sivri B1'de durur, A1 yok olur):
      4 . K . .
      3 L . . .
      2 . . . S
      1 . S . .
  - D2← (Sivri A2'de durur, A3 yok olur):
      4 . K . .
      3 . . . .
      2 S . . .
      1 . S . .
  - B4↓ (Kule B2'de durur; A2, B1 ve Kule'nin kendisi yok olur): tahta boş.
  Beklenen durum: Ara adımlarda OYNANIYOR, son adımda KAZANILDI.

A2 Kurala uygun ama çözümü bozan hamleler (eski hatayı yakalar)
  Başlangıç: S08.
  - D2←: Sivri, B2'den geçerken B1'deki Kule'nin yanından geçer (yol etkisizdir). A2'de durur; A3 ve A1 yok olur. A2'ye çapraz duran B1 Kulesi kalır (çapraz etkisizdir).
      4 . . . .
      3 . . . .
      2 S . . .
      1 . K . S
      Durum: OYNANIYOR, cozulebilir = HAYIR.
  - D1←: Sivri C1'de durur. C1'deki Sivri ve B1'deki Kule birlikte yok olur.
      4 . . . .
      3 . . . .
      2 S . . .
      1 . . . .
      Durum: OYNANIYOR.
  Bütün Levha'lar gitti ve animasyon bitti, ama seviye GEÇİLMEMELİ.

A3 Geçersiz girdiler
  Başlangıç: S08. Şu girdilerin her biri reddedilir:
  - A3→ (Levha hareket etmez)
  - B1← (A1 dolu, mesafe 0)
  - D1→ (tahta kenarı)
  - C2↑ (boş hücre)
  Beklenen: Dizilim, hamle sayısı (0) ve durum değişmez.

A4 Hamle kalmayan tahta
  Başlangıç: "S.K............L"
      4 S . K .
      3 . . . .
      2 . . . .
      1 . . . L
  Girdi: A4→. Sivri B4'te durur; Sivri ve C4'teki Kule birlikte yok olur.
  Beklenen son dizilim: Yalnızca D1'de L kalır.
  Beklenen: Durum OYNANIYOR, hamleVar = false, cozulebilir = HAYIR. Kazanılmış sayılmaz.

A5 Kule Levha'yı yok etmez
  Başlangıç: S28.
  Girdi: A2→. Kule C1'in yanından geçer (etkisiz) ve D2'de durur. Kule ve D1'deki Sivri yok olur; D3'teki Levha kalır.
      4 S . L .
      3 . L . L
      2 . . . .
      1 . . S .
  Beklenen durum: OYNANIYOR.

A6 Sivri–Sivri etkisiz
  Başlangıç: S28.
  Girdi: D1↑ → D2↑ → A4→. Son hamlede Sivri C4'te, D4'teki Sivri'nin yanında durur; hiçbir şey yok olmaz.
      4 . . S S
      3 . L . .
      2 K . . .
      1 . . S .

A7 Aynı anda yok olma (tuzak)
  Başlangıç: S15.
  Girdi: B3↑. Kule B4'te durur; A4'teki Sivri ve Kule yok olur, C4'teki Levha kalır.
      4 . . L .
      3 . . . .
      2 L L . .
      1 . . . S

A8 Dörtlü temizlik
  Başlangıç: Ö3.
  Girdi: B2↓ → B4↓.
  Beklenen: Son hamlede 4 taş birden gider; tahta boş, KAZANILDI.

A9 Değişmez kontrolü
  Bütün testlerde her hamleden sonra Sivri–Levha veya Sivri–Kule olarak yan yana duran çift kalmadığını doğrula.

B) ÇÖZÜCÜ VE ÖLÇÜTLER
- Ö1–Ö3 ve S01–S30 için motorun hesapladığı E, Ç, T ve F, §7'deki değerlerle birebir aynı olmalı. Ö ve G ±0,01 toleransla aynı olmalı.
  Ö1–Ö3'ün değerleri:
  * Ö1: E1, Ç2, T 0/6
  * Ö2: E2, Ç3, T 1/6
  * Ö3: E2, Ç1, T 2/5
- Verilen her çözüm oynanınca KAZANILDI olmalı. Ara adımlarda hiçbir zaman KAZANILDI olmamalı.
- 33 tahtanın hepsi simetri-kanonik biçimde birbirinden farklı olmalı.

C) ÜRETİCİ
- C1 Determinizm: S31–S80'i Node'da iki kez üret; sonuçlar birebir aynı olmalı. Tarayıcıdaki Worker'ın S31, S55 ve S80 için ürettiği dizilim Node'unkiyle aynı olmalı.
- C2 Her üretilmiş seviye:
  * Çözülebilir olmalı ve başlangıçta yan yana etkileşen çift olmamalı.
  * P ve H aralıkta olmalı.
  * Ölçütler hedefleri tutmalı ya da gevşetme meta verisi kayıtlı olmalı.
  * Önceki bütün seviyelerden farklı olmalı.
  * Ölçütler ayrı bir analyze() çağrısıyla yeniden hesaplanıp aynı çıkmalı.
- C3 Ortalama D değeri aralıklar boyunca kesin olarak artmalı: (S21–S30) ≤ (S31–S45) < (S46–S60) < (S61–S72) < (S73–S80).
- C4 Süre: Seviye başına medyan ≤ 3 sn, en kötü ≤ 20 sn (Node).
- C5 rapor/seviye-tablosu.md: 80 seviyenin hepsi için dizilim, ölçütler, gevşetmeler, F sayısı ve üretim süresi.
- C6 Önbellek: Yeniden yüklemede aynı seviye önbellekten gelmeli. URETICI_SURUM artırılınca seviye yeniden üretilmeli, ama tamamlanma kaydı korunmalı.

D) GECİKMELİ ÇIKMAZ UYARISI (sahte saatle)
- D1 Hamle eşiği:
  S08 → D2← (tuzak; uyarı yok).
  Sonra D1←, A2↑, A4→ (tuzaktan sonra 3 hamle) → hâlâ uyarı yok.
  Sonra D4↓ (4. hamle) → uyarı görünür.
- D2 Süre eşiği:
  S08 → D2←. 29 sn aktif süre sonra uyarı yok; 31 sn sonra uyarı var.
  Sayfa arka plandayken geçen süre sayılmaz.
- D3 Hamle kalmayan tahta (A4'teki tahta):
  A4→ sonrası hemen uyarı yok.
  2 başarısız denemeden sonra uyarı yok; 3. denemede uyarı var.
  Ayrı bir koşuda: 10 sn sonra uyarı var.
- D4 Çözülebilir tahtada asla uyarı yok:
  S08 başlangıcında 5 dk bekle → uyarı yok.
  B1↑ yap, 5 dk bekle → uyarı yok.
- D5 "Biraz daha dene" seçilirse ikinci uyarı yeni bir eşikten sonra çıkar. Bir denemede en fazla 2 uyarı. Baştan kur sayaçları sıfırlar.
- D6 Ayar "Kapalı" iken hiç uyarı çıkmaz.
- D7 Sızıntı yok: Tuzak hamlesiyle güvenli hamleden hemen sonra arayüz (DOM, ses tetikleri, HUD) birebir aynı olmalı.

E) EKONOMİ VE MARKET
- E1 Yeni kayıt: 0 altın. Öğretici bitince 5 altın. Öğreticiyi tekrar oynamak altını değiştirmez.
- E2 S01 3 hamlede (E) temizlenir: +2 +1 → 8 altın. Aynı şekilde tekrar oynanır: +0.
- E3 S02, en kısa olmayan bir çözümle temizlenir (çözücüden bul): +2, bonus yok.
  Tekrar oynanıp 3 hamlede temizlenir: +1. Bir kez daha: +0.
- E4 Geliştirici kaydıyla: S21 → +3, S46 → +4, S71 → +5.
- E5 Kazanma animasyonu sırasında Yeniden Başlat: Ödül tam bir kez verilir, seviye tamamlanmış sayılır, tahta başlangıç dizilimine döner.
- E6 Satın alma:
  * 9 altınla 10'luk ürün alınamaz: düğme pasif, altın değişmez.
  * 10 altınla alınır: altın 0'a iner, ürün sahip olunanlara eklenir.
  * Kuşanılınca tahtaya yansır. Sayfa yeniden yüklenince kalıcıdır.
  * Çift dokunuş tek düşüş yapar.
- E7 Yeni kayıtta varsayılanlar sahip olunan ve kuşanılmış durumdadır.
- E8 Bozuk kayıt dizesiyle oyun çökmeden açılır, yedek alınır, konsola uyarı yazılır.

F) ARAYÜZ (Playwright + Chromium, mobil emülasyon 390×844, dokunmatik açık; kurulu değilse kurmayı dene)
- F1 İlk açılışta öğretici başlar. Rehberli olmayan bir hamle reddedilir. Üç tahta gerçek kaydırmalarla bitirilir. +5 altın verilir. Ana menüye dönülür.
- F2 S08 gerçek kaydırmalarla çözülür. "Sonraki tahta" görünür. 5 sn sonra hâlâ S08'dedir. Düğmeyle S09 yüklenir; hamle sayısı 0.
- F3 A2 senaryosu arayüzden oynanır. Animasyonlar bitip 2 sn daha geçer. Beklenen: OYNANIYOR, düğme yok.
- F4 Hızlı çift kaydırma: B1↑, 40 ms sonra D1←.
  Beklenen: Yalnızca ilk hamle uygulanır, hamle sayısı 1. Animasyon bitince D1← tekrarlanır ve uygulanır.
- F5 Animasyon sırasında yeniden başlatma: A1'in ilk üç hamlesi yapılır. B4↓'dan 50 ms sonra Yeniden Başlat. 1,5 sn beklenir.
  Beklenen: S08 başlangıç dizilimi, OYNANIYOR. Eski animasyon hiçbir taşı silmemiş. Ödül E5'e uygun.
- F6 Doğru taşın seçilmesi: S08, S28 ve S30 başlangıcında, her taş için 'top' ve 'body' noktalarına dokunulur.
  Beklenen: Seçilen taş o hücredeki taş. Arka (4.) ve ön (1.) sıra ayrıca kontrol edilir. Kule'nin arka kareye taşan tepesi Kule'yi seçer. Boş kare seçimi kaldırır.
  Bu test en uzun ve en kısa görünüm kuşanılıyken de tekrarlanır.
- F7 Ön ve arka sıradaki bir taşla dört yön gerçek kaydırmayla denenir. Beklenen: Motora giden yön her seferinde beklenen yön.
- F8 Seviye seçimi:
  * Kilitli seviye açılamaz.
  * S31 açılınca "Tahta yontuluyor…" görünür ya da seviye önbellekten gelir. Tahta oynanabilir.
  * Sayfa yenilenince aynı dizilim gelir.
- F9 Market akışı dokunmatik olarak: kategori seç, ürün aç, döndür, satın al, kuşan, oyuna dön. Beklenen: Tahta yeni görünümle görünür.

G) GÖRSEL KONTROL
- Ekran görüntüsü al ve kendin incele: her tahta teması, her taş görünümü (S28 üzerinde), market ekranı, kazanma katmanı ve uyarı. Hem dikey (390×844) hem yatay (844×390).
- Ölçülecekler:
  1. Her görünüm için, kameradan her taşın tepe merkezine gönderilen ışının ilk çarptığı nesne o taş olmalı (arka sıra okunabilirliği).
  2. Bütün görünüm × tema eşleşmelerinde kontrast kuralı (§9).
  3. Tahtanın ekranı doldurma oranları ve arka/ön sıra hücre oranı ≥ 0,7.
  4. Kayma animasyonundan en az 3 ara kare; taşın çıktığı ve vardığı kare bu karelerde seçilebilmeli.

§15 TESLİM, YAYIN VE PAYLAŞIM

Amaç: Oyunu arkadaşlarıma bir linkle gönderebilmeliyim.
- Link telefonda ve bilgisayarda doğrudan tarayıcıda açılmalı.
- İsteyen, telefonunda "Ana ekrana ekle" ile oyunu uygulama gibi kurabilmeli ve ilk açılıştan sonra internetsiz oynayabilmeli.
- Sunucu, hesap veya veritabanı yok. Oyun tamamen statik bir site olacak.

ÇIKTILAR
1. site/ klasörü (yayınlanacak olan):
   - index.html: Oyunun tamamı. JS, CSS ve Worker kodu içine gömülü.
   - manifest.webmanifest:
     * name ve short_name: "Sürgü"
     * start_url ve scope: "./"
     * display: "standalone"
     * Renkler: tahta temasından.
     * İkonlar: 192 ve 512 piksel PNG, ayrıca bir "maskable" sürüm.
   - sw.js: Service worker.
   - İkonları build script'i kodla üretsin (ör. tek bir Sivri silueti). Dışarıdan dosya yok.
   - Bütün yollar göreli olsun ("./"). Site bir alt klasörde yayınlanacak (ör. kullanici.github.io/surgu/). Kök yola ("/") bağlı tek bir yol bile olmasın.
2. dist/surgu.html: Tek dosyalık yedek.
   - Çift tıklayınca (file://) çalışır.
   - Bu sürümde service worker ve kurulum özellikleri kapalıdır; oyun yine eksiksiz oynanır.
   - Worker, Blob URL ile açılamazsa üretici ana iş parçacığında küçük parçalara bölünerek çalışır.
3. Kaynak kod, testler, README, rapor/seviye-tablosu.md ve kısa rapor.
   Rapor şunları içerir:
   * Önceki prototipteki hata ve kök nedeni (varsa)
   * Yapılanlar
   * Test sonuçları
   * Ekran görüntüleri
   * Gevşetilen seviyeler ve F sayısı
   * Çalıştırılamayan kontroller
   * Öncelik nedeniyle yapılamayanlar
   * Yayın linki veya yayın için benim yapmam gereken adımlar

SERVICE WORKER VE GÜNCELLEME
- Yalnızca http/https üzerinde kaydedilsin. Sayfa file:// ile açıldıysa kayıt hiç denenmesin.
- index.html, manifest ve ikonlar önbelleğe alınsın. İlk ziyaretten sonra oyun internetsiz açılsın.
- Önbellek adı build sürümünü içersin (ör. "surgu-" + içerik hash'i).
- Yeni sürüm yayınlanınca:
  * Eski önbellek silinsin.
  * Oyunda "Yeni sürüm hazır, yenile" bildirimi çıksın.
  * Oyuncu eski sürümde sonsuza kadar takılı kalmasın.
- Güncellemede kayıt (altın, ilerleme, kozmetikler) korunur.

KURULUM VE PAYLAŞIM ARAYÜZÜ
- Ayarlar'da "Ana ekrana ekle" bölümü olsun:
  * Tarayıcı kurulum olayını (beforeinstallprompt) destekliyorsa "Yükle" düğmesi.
  * iPhone/Safari'de "Paylaş → Ana Ekrana Ekle" talimatı.
  * Oyun zaten kurulu olarak açıldıysa (display-mode: standalone) bu bölüm gizlensin.
- Ana menüde "Paylaş" düğmesi olsun:
  * navigator.share varsa sistemin paylaşım menüsünü açsın.
  * Yoksa site linkini panoya kopyalayıp "Link kopyalandı" desin.
- Sayfanın başlığı ve açıklaması olsun. Ayrıca og:title ve og:description etiketleri olsun, böylece link mesajlaşma uygulamalarında düzgün önizlensin.
- og:image mutlak bir adres ister:
  * Build sırasında SITE_URL değişkeni verilirse, kodla render edilmiş bir tahta görseli paylaşım görseli olarak eklensin.
  * Verilmezse og:image atlansın.
- Oyun dışarıya hiçbir istek yapmasın: harici yazı tipi, analitik, reklam veya CDN yok. Sistem yazı tiplerini kullan.

İLERLEME TAŞIMA (P2)
- İlerleme cihaza ve tarayıcıya bağlıdır (localStorage).
- Ayarlar'da iki seçenek olsun:
  * "İlerlemeyi dışa aktar": Kopyalanabilir bir kod üretir.
  * "İçe aktar": Bu kodu geri yükler.
- Kod sürüm numarası ve sağlama toplamı içersin. Bozuk ya da uyumsuz kod reddedilsin ve mevcut kayıt bozulmasın.

YAYIN (GITHUB PAGES)
- .github/workflows/pages.yml ekle. main dalına her push'ta şunu yapsın:
  1. Testleri çalıştır.
  2. Build al.
  3. site/ klasörünü actions/upload-pages-artifact ve actions/deploy-pages ile GitHub Pages'e yayınla.
  Testler düşerse yayınlama.
- Bu depoda GitHub Pages kaynağı zaten "GitHub Actions" olarak ayarlandı; değişikliklerin main'e alınınca iş akışı yayını kendisi yapacak.
- README'nin en başına şunları yaz:
  1. Oyunun linki: https://<kullanıcı-adı>.github.io/<depo-adı>/
  2. Yayın çalışmazsa kontrol edilecek adımlar: Settings → Pages → Source: "GitHub Actions" seçili mi; Actions sekmesinde son iş akışı yeşil mi.
  3. Alternatif: site/ klasörünü herhangi bir statik barındırma servisine yüklemek yeterlidir; özel ayar gerekmez.

YAYIN TESTLERİ (çalıştır ve sonuçlarını raporla)
- GitHub Pages'i taklit et: site/ klasörünü yerel bir statik sunucuyla ALT KLASÖR altında sun (ör. http://localhost:8080/surgu/).
  Playwright ile kontrol et:
  * Sayfa açılıyor.
  * Öğretici ve S01 oynanabiliyor.
  * Service worker kaydoluyor.
  Sonra ağı kes (setOffline) ve sayfayı yenile. Beklenen: Oyun yine açılır, kayıt yerindedir.
- Güncelleme simülasyonu: Küçük bir içerik değişikliğiyle yeniden build al.
  Beklenen: Açık sekmede "Yeni sürüm hazır" bildirimi çıkar; yenileyince yeni sürüm gelir ve kayıt korunur.
- Manifest kontrolü:
  * Zorunlu alanlar mevcut.
  * İkon dosyaları var ve boyutları doğru.
  * start_url ve scope göreli.
- Build çıktısını tara: "/" ile başlayan yerel yol olmamalı.
- dist/surgu.html'i file:// ile aç.
  Beklenen: Öğretici ve S01 oynanır, service worker kaydı denenmez, konsolda hata yoktur.
- Ağ isteği denetimi: Oyun boyunca site dışına hiç istek gitmemeli.
- "Paylaş" düğmesini navigator.share olmayan bir tarayıcıda dene. Beklenen: Link panoya kopyalanır.
- Ekran görüntüleri: Mobil (390×844) ve masaüstü (1280×800).

KAYNAKLAR (referans analizinde kullanıldı; açman gerekmiyor)
- GameRant, Sliding Tile Board Puzzle Solutions — taş türleri, hareket ve yok etme kuralları, 5 tahta, rastgele dizilimler, 6. taşa kadar numaralı çözümler: https://gamerant.com/gta-6-tenebrae-lux-sliding-tile-board-puzzles/
- GameRant, Tenebrae Lux Walkthrough — konsol, dört aşama: https://gamerant.com/gta-6-tenebrae-lux-walkthrough/
- ShowGamer, All Four Boards — "yarı yolda durmaz", hamle kalmayınca sıfırlanma (tek kaynak): https://showgamer.com/en/guides/8053-kak-reshit-vse-golovolomki-s-plitkami-tenebrae-lux-v-gta-6-polnyy-gayd
- ShowGamer, First Puzzle — 4×4 ızgara: https://showgamer.com/en/guides/7915-kak-reshit-pervuyu-golovolomku-tenebrae-lux-v-gta-6-reshenie-za-tri-hoda
- tenebraelux.site, topluluk çözücüsü — ortogonal komşuluk, durma anında tetiklenme (resmî değil): https://tenebraelux.site/puzzle
- Outputlag — siyah-beyaz tahta, mumlu tezgâh, taş piramitler ve düz ahşap bloklar: https://outputlag.com/guides/gta-6-tenebrae-lux/
- Atlas of Vice — düz parçalar hareket etmez, yok etme durma yerinde olur: https://atlasofvice.com/guides/tenebrae-lux-puzzles
- Pro-Gamer.gg — 4×4 ahşap ızgara, mum ışıklı masa. "Yol boyunca yok eder" iddiası diğer kaynaklarla çelişiyor; kullanılmadı: https://pro-gamer.gg/how-to-complete-tenebrae-lux-minigame-puzzle-gta-6-3-move-board-solution-guide/
- Insider Gaming, ilk tahtanın çözümü: https://insider-gaming.com/tenebrae-lux-gta-6/
- XboxPlay, aynı çözüm: https://xboxplay.games/gta-6/gta-6-tenebrae-lux-minigame-puzzle-solution-guide-77426
- Game Overdrive (PT), aynı çözüm ve sıfırlama: https://gameoverdrive.com.br/gta-6-como-resolver-tenebrae-lux/
- VGC, Feng Shui görevi — oyun içi konsol bağlamı: https://www.videogameschronicle.com/guide/gta-6-feng-shui-mission-walkthrough/
- GTA Wiki, Tenebrae Lux maddesi: https://gta.wiki/w/Tenebrae_Lux
- theclick.gg ve games.gg rehberleri: Sayfalar açılamadı, yalnızca arama özetleri okundu: https://www.theclick.gg/gta-6-how-to-complete-tenebrae-lux/ , https://games.gg/grand-theft-auto-vi/guides/gta-6-tenebrae-lux-minigame/
- Video zamanı yok: Hiçbir oynanış videosu kare kare izlenemedi. Kurallar yazılı kaynaklardan çıkarıldı ve ilk tahtanın yayımlanmış çözümünü yeniden üreterek doğrulandı.
- Öğretici, S01–S30 ve bütün ölçüt değerleri bu prompt hazırlanırken aynı kural modeliyle BFS'ten geçirildi.
