# EKG’de Filtrelerin R Tepesi Tespitine Etkisi

EEM4011 projesinde, gürültülü EKG’de R tepesi tespitinin kırılma SNR’sine 0,5 Hz yüksek geçiren, 40 Hz alçak geçiren ve 60 Hz çentik filtrelerinin katkısı incelenmektedir. Makine öğrenmesi kullanılmamaktadır.

## Veri setleri

- [MIT-BIH Arrhythmia Database](https://physionet.org/content/mitdb/1.0.0/): 100, 101, 103, 115, 122, 123 ve 234 kayıtlarının 0. kanalı ve `atr` anotasyonları.
- [MIT-BIH Noise Stress Test Database](https://physionet.org/content/nstdb/1.0.0/): `bw` (taban çizgisi kayması) ve `ma` (kas gürültüsü) kayıtlarının 0. kanalı.

Veri dosyaları depoya eklenmemiştir. Kod, yalnız belirtilen kayıtları WFDB üzerinden PhysioNet’ten okur.

## Çalıştırma

1. Depodaki `.ipynb` dosyasını Google Colab’da açın.
2. Hücreleri yukarıdan aşağıya sırayla çalıştırın.
3. İlk hücre `wfdb` paketini kurar, sabitleri `params.json` dosyasına yazar ve verileri belleğe alır. Sonraki hücreler bu verileri kullanır.

İnternet bağlantısı gereklidir. Colab oturumu sıfırlanırsa hücreleri baştan çalıştırın.

## Yöntem ve ilk sonuç

Tespit eşiği, işlenen sinyalin ortalaması + 3 × standart sapmasıdır. Tepeler arasında en az 90 örnek bulunur; referansla ±54 örnek içinde bire bir eşleşen tespitler doğru kabul edilir.

SNR, +30’dan −30 dB’ye 2 dB adımlarla azaltılır. F1’in ilk kez 0.95’in altına düştüğü düzey kırılma SNR’sidir.

100 kaydı ve `bw` gürültüsü için:

| Ölçüm | Sonuç |
|---|---:|
| Ek gürültüsüz, filtresiz F1 | 0.999560 |
| Filtresiz kırılma SNR’si | −4 dB |
| 0,5 Hz yüksek geçiren filtre sonrası kırılma SNR’si | −22 dB |
| İyileşme | 18 dB |

Yüksek geçiren filtre, 2. derece Butterworth tasarımıyla ileri–geri uygulanmıştır. İlk sonuç yalnız 100 kaydı ve kullanılan `bw` kesiti için geçerlidir; diğer kayıtlar, `ma`, alçak geçiren ve çentik filtre sonuçları henüz değerlendirilmemiştir.# eem4011-proje
