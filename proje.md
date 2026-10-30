# EEM4011 — EKG filtreleri ve R tepesi gürültü toleransı

## Amaç ve kapsam

Bu projede 0,5 Hz yüksek geçiren (Y), 40 Hz alçak geçiren (A) ve 60 Hz çentik (Ç) filtrelerinin gürültülü EKG'de R tepesi tespitinin kırılma SNR'sine katkısı ölçülmektedir. Filtreler aynı gürültülü girdiye tek tek uygulanır; birleşik zincir bu deneyde değerlendirilmemiştir. Makine öğrenmesi veya sınıflandırıcı eğitimi kullanılmaz.

Çalışmanın sorusu: Aynı kayıt, gürültü kesiti, SNR taraması ve dedektör altında her filtre kırılma SNR'sini ne kadar değiştirir? Bu, ders ölçeğinde kontrollü bir karşılaştırmadır; literatürde ilk kez yapıldığı veya bir araştırma boşluğunun kesin olarak doğrulandığı iddia edilmez.

## Veri

- MIT-BIH Arrhythmia Database: https://physionet.org/content/mitdb/1.0.0/
- Kayıtlar: 100, 101, 103, 115, 122, 123, 234; yalnız kanal 0 (MLII).
- Referans: `atr` anotasyonlarındaki `N L R B A a J S V r F e j n E / f Q` sembolleri. Buradaki R, sağ dal bloğu atımı sembolüdür.
- MIT-BIH Noise Stress Test Database: https://physionet.org/content/nstdb/1.0.0/
- Yalnız `bw` (taban çizgisi kayması) ve `ma` (kas gürültüsü), kanal 0 kullanılır.
- Çalıştırma çıktısında tüm sinyallerin örnekleme hızı 360 Hz, uzunluğu 650000 örnektir.

Yalnız seçilen kayıtlar WFDB ile okunur. Ham veri dosyaları GitHub deposuna eklenmez. Yedi kayıt ayrı ayrı değerlendirilmiştir; bu deney bağımsız kişilerde genelleme testi olarak sunulmaz.

## Deney yöntemi

Gürültünün EKG ile aynı uzunluktaki başlangıç kesiti alınır ve ortalaması çıkarılır. Her kayıtta aynı gürültü kesiti kullanılır. Güç, varyans olarak tanımlanır:

`alpha = sqrt(var(x) / (var(n) * 10**(SNR/10)))`

`gürültülü EKG = x + alpha*n`

Giriş SNR'si +30'dan -30 dB'ye 2 dB adımlarla taranır. Aynı gürültülü EKG, ham ve üç ayrı filtre koşulunda değerlendirilir.

| Koşul | Tasarım |
|---|---|
| Ham | Filtre yok |
| Y | Butterworth, 0,5 Hz yüksek geçiren, 2. derece |
| A | Butterworth, 40 Hz alçak geçiren, 4. derece |
| Ç | 60 Hz çentik, Q=30 |

Filtreler `sosfiltfilt` ile ileri–geri uygulanır. Dereceler tek geçiş tasarımını belirtir; ileri–geri uygulama toplam cevabı değiştirir. İşlem çevrimdışıdır ve gerçek zamanlı uygulama olarak sunulmaz.

### Dedektör ve eşleştirme

Eşik, her koşulda işlenen sinyalin kendi `ortalaması + 3 × standart sapması` olarak yeniden hesaplanır. `find_peaks`, bu eşikten yüksek pozitif tepeleri en az 90 örnek aralıkla seçer. Dedektör kuralı sabittir; sayısal eşik filtrelenmiş sinyale göre değişebilir. Yumuşatma, türev veya Pan–Tompkins uygulanmaz.

Tespitler ve referanslar zaman sırasıyla, ±54 örnek toleransla bire bir eşleştirilir. Bir tespit veya referans birden fazla eşleşmede kullanılmaz.

`F1 = 2*TP / (2*TP + FP + FN)`

Ana ölçüt, yüksekten düşüğe SNR taramasında F1'in ilk kez 0.95'in altına düştüğü SNR'dir. İyileşme `kırılma(ham) - kırılma(filtre)` olarak hesaplanır; pozitif fark daha güçlü gürültüye dayanımı gösterir. Sonuçlar 2 dB çözünürlüklüdür. İlk düzeyde başarısızlık `>+30`, tüm düzeylerde başarı `<-30` etiketiyle gösterilir; bunlar ölçülmüş sayısal değerler değildir. Bu deneyin tablosunda tarama dışı sonuç yoktur.

## Ölçülen sonuçlar

### Ek gürültüsüz, filtresiz kontrol

| Kayıt | F1 |
|---|---:|
| 100 | 0.999560 |
| 101 | 0.995456 |
| 103 | 0.996883 |
| 115 | 0.997183 |
| 122 | 0.999394 |
| 123 | 0.999342 |
| 234 | 0.993252 |

Tüm kayıtlar 0.95 eşiğini sağlamıştır. Ek gürültüsüz ifadesi, kayıtlara yapay olarak gürültü eklenmediğini belirtir; kaynak EKG'lerin kusursuz temiz olduğu varsayılmaz.

### Kırılma SNR'si (dB)

| Kayıt | Gürültü | Ham | Y | A | Ç |
|---|---|---:|---:|---:|---:|
| 100 | bw | -4 | -22 | -2 | -4 |
| 100 | ma | -4 | -10 | -2 | -4 |
| 101 | bw | -2 | -22 | -2 | -2 |
| 101 | ma | -2 | -8 | -2 | -2 |
| 103 | bw | -2 | -22 | -2 | -2 |
| 103 | ma | -2 | -8 | -2 | -2 |
| 115 | bw | -2 | -22 | -2 | -2 |
| 115 | ma | -2 | -8 | -2 | -2 |
| 122 | bw | 2 | -18 | 2 | 2 |
| 122 | ma | 2 | -8 | 2 | 2 |
| 123 | bw | -4 | -22 | -4 | -4 |
| 123 | ma | -4 | -6 | -4 | -4 |
| 234 | bw | 2 | -20 | 2 | 2 |
| 234 | ma | 0 | -8 | 2 | 0 |

### PSD öngörüsü ve ölçülen fark

Welch yöntemi: örnekleme hızı 360 Hz, Hann penceresi, 16384 örnek segment, 8192 örnek örtüşme, `detrend=False`. Her sinyalin ortalaması Welch hesabından önce çıkarılır. İleri–geri filtreleme için güç çarpanı `abs(H)**4` olarak alınır. PSD integralleriyle kalan EKG ve gürültü güç oranları hesaplanır:

`G = 10*log10(EKG güç oranı / gürültü güç oranı)`

| Gürültü | Filtre | Kayıt bazında kırılma farkının medyanı (dB) | PSD SNR kazancının medyanı (dB) |
|---|---|---:|---:|
| bw | Y | 20 | 17.564 |
| bw | A | 0 | -0.054 |
| bw | Ç | 0 | -0.001 |
| ma | Y | 6 | 6.529 |
| ma | A | 0 | 0.060 |
| ma | Ç | 0 | 0.002 |

PSD öngörüsü güç değişimini, ölçülen fark ise dedektörün F1 eşiğine göre dayanımını ifade eder; eşit olmaları beklenmez. PSD hesabı kayıt uçlarındaki filtreleme etkilerini içermez.

## Yorum ve sınırlar

Y filtresi incelenen koşullarda en yüksek katkıyı vermiştir. A filtresinin medyan katkısı sıfırdır; bazı kayıtların kırılmasını 2 dB kötüleştirmiştir. Ç filtresi kırılma düzeyini değiştirmemiştir. Ayrı bir 60 Hz girişimi eklenmediğinden bu sonuç, çentiğin 60 Hz girişiminde etkisiz olduğunu göstermez.

Sonuçlar seçili yedi kayda, kullanılan sabit gürültü kesitlerine ve bu dedektöre aittir. Farklı gürültü başlangıçları, birleşik filtre zinciri, güven aralığı, kesim frekansı taraması ve bağımsız kişi genellemesi henüz değerlendirilmemiştir. Klinik başarı iddiası yoktur.

## Çalıştırma ve teslim

Colab defterindeki hücreler yukarıdan aşağıya çalıştırılır. İlk hücre `wfdb` kurar, `params.json` yazar ve yalnız seçilen verileri okur. Sonraki hücreler bellekteki verileri kullanır. Oturum sıfırlanırsa hücreler baştan çalıştırılmalıdır.

Teslim dosyaları: kod defteri (`.ipynb`), `params.json`, sonuç tabloları ve `filtre_katkisi.png`. Sonuç paketinde ham EKG verisi yoktur. Kod defteri ayrıca Colab'dan kaydedilmelidir; indirilen sonuç ZIP'i defteri içermez.

## Aşama 1 belgesine ilişkin düzeltme notu

Eski 50 Hz ve Pan–Tompkins planı yerine çalışan deney yukarıda açıklanmıştır. Önceki belgedeki 207 kaydına ait literatür sayıları, VMD/U-Net karşılaştırmaları ve Pro Report'a dayalı araştırma boşluğu iddiaları bu sürümde kullanılmamıştır; bu güncellemede birincil kaynak doğrulamaları yapılmamıştır. Eski “6/6” ifadesi mevcut projenin tüm literatür iddialarının doğrulandığına kanıt sayılmamaktadır; önceki kontrolün kapsamı ayrıca belgelenmelidir.

Apple Watch olayının tarih ve birincil kaynak doğrulaması ile öğrencinin duyuruyu kendisinin açması bu çalışma kapsamında tamamlanmış olarak beyan edilmez. Bu yüzden formdaki bütün düzeltme kutuları otomatik olarak tamamlanmış sayılmamalıdır.

## CV taslağı

Python ile seçili PhysioNet EKG kayıtlarında dijital filtrelerin R tepesi tespitinin kırılma SNR'sine etkisini karşılaştıran bir deney geliştirdim; kayıt bazında medyan katkıyı bw için ___ dB, ma için ___ dB olarak raporladım.

## Yapay zekâ kullanımı

ChatGPT; kod taslağı, çıktıların yorumlanması ve belge düzenlemesi için kullanılmıştır. Önceki proje belgesinde Science AI Journal / Research Gap Finder kullanımı da beyan edilmiştir. Bu sürümde deney sayıları öğrencinin paylaştığı Colab çıktılarından alınmıştır. Kaynakların ve nihai metnin kontrolü öğrenciye aittir. Formun öğrenciden istediği üç kişisel açıklama cümlesi bu belgeyle ikame edilmez.
