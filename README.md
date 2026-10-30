# EEM4011 — EKG filtreleri ve R tepesi gürültü toleransı

Bu proje, gürültülü EKG'de R tepesi tespitine üç filtrenin ayrı ayrı katkısını ölçer: 0,5 Hz yüksek geçiren (Y), 40 Hz alçak geçiren (A), 60 Hz çentik (Ç). Makine öğrenmesi kullanılmaz.

## Veri

- [MIT-BIH Arrhythmia Database](https://physionet.org/content/mitdb/1.0.0/): 100, 101, 103, 115, 122, 123, 234 kayıtları; kanal 0 ve `atr` atım anotasyonları.
- [MIT-BIH Noise Stress Test Database](https://physionet.org/content/nstdb/1.0.0/): yalnız `bw` ve `ma`, kanal 0.

Ham veri depoya eklenmez. Kod yalnız belirtilen kayıtları WFDB ile okur.

## Çalıştırma

1. Projenin `.ipynb` defterini Google Colab'da açın.
2. Hücreleri yukarıdan aşağıya çalıştırın. İlk hücre `wfdb` paketini kurar, `params.json` oluşturur ve verileri okur; internet gerekir.
3. Sonraki hücreler bellekteki verilerle tespit, gürültü taraması, filtre karşılaştırması ve PSD hesabını yapar.
4. Sonuç hücresi grafiği ve tabloları `sonuclar/` klasörüne kaydeder. İndirme hücresi bunları `params.json` ile paketler.

Colab oturumu sıfırlanırsa hücreleri baştan çalıştırın. Kod defterini ayrıca kaydedin; sonuç ZIP'i kodu içermez.

## Yöntem

Dedektör, işlenen sinyalin ortalaması + 3 standart sapma üzerindeki pozitif tepeleri en az 90 örnek aralıkla bulur. Referanslarla ±54 örnek içinde bire bir eşleştirme yapılır. SNR, +30'dan -30 dB'ye 2 dB adımla azaltılır; F1'in ilk kez 0.95'in altına düştüğü düzey kırılma SNR'sidir.

Y: 2. derece Butterworth; A: 4. derece Butterworth; Ç: Q=30. Üç filtre ileri–geri uygulanır ve ayrı ayrı değerlendirilir. Birleşik zincir denenmemiştir.

## Sonuç

Yedi kaydın tamamında ek gürültüsüz, filtresiz F1 > 0.95 bulunmuştur. Kayıt bazında `ham kırılma - filtreli kırılma` farkının medyanları:

| Gürültü | Y | A | Ç |
|---|---:|---:|---:|
| bw | 20 dB | 0 dB | 0 dB |
| ma | 6 dB | 0 dB | 0 dB |

Y için PSD'den beklenen SNR kazancı medyanı bw'de 17.564 dB, ma'da 6.529 dB'dir. PSD kazancı ile kırılma farkı farklı ölçütlerdir.

İlk deneyde 100 + bw için ham kırılma -4 dB, Y sonrası -22 dB bulunmuştur. Sonuçlar seçili kayıtlar ve sabit gürültü kesitleriyle sınırlıdır. Ayrı 60 Hz girişimi eklenmediğinden çentiğin böyle bir girişimdeki başarısı hakkında sonuç çıkarılmaz. Ayrıntılar `proje.md` dosyasındadır.

## Çıktılar

- `params.json`: deney ayarları
- `sonuclar/kirilma_sonuclari.csv`: kayıt bazında kırılmalar
- `sonuclar/psd_karsilastirma.csv`: ayrıntılı ölçüm ve PSD karşılaştırması
- `sonuclar/medyan_ozet.csv`: medyanlar
- `sonuclar/filtre_katkisi.png`: karşılaştırma grafiği

Bu tablolar hesaplanmış deney sonuçlarıdır; ham EKG dosyaları değildir.
