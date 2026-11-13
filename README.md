# Öğrenci Başarı Tahmini Projesi (Karar Ağacı Odaklı Analiz)

Bu projede `student-mat.csv` (Matematik) ve `student-por.csv` (Portekizce) veri setlerini kullanarak öğrencilerin dönem sonu final notunu (`G3`, 0-20 arası) tahmin eden **Karar Ağacı (`DecisionTreeRegressor` ve `DecisionTreeClassifier`)** merkezli makine öğrenmesi modelleri kurdum. Tüm veri keşfi, görselleştirmeler, karar ağacı şemaları (`plot_tree`) ve karşılaştırma kodları `ogrenci_basari_tahmini.ipynb` dosyasında yer almaktadır.

## Klasördeki Dosyalar

- `student-mat.csv`: Matematik dersine ait 395 öğrenci kaydı (33 sütun)
- `student-por.csv`: Portekizce dersine ait 649 öğrenci kaydı (33 sütun)
- `student.txt`: Değişken açıklamaları ve veri seti notları
- `student-merge.R`: İki derste ortak olan 382 öğrenciyi eşleştiren referans R kodu
- `karar_agaci_modelleme.ipynb` / `ogrenci_basari_tahmini.ipynb`: Veri analizi, karar ağacı modellemesi (`DecisionTreeRegressor`, `DecisionTreeClassifier`) ve görselleştirme notebook'u
- `requirements.txt`: Kullanılan Python kütüphaneleri

## Veri Keşfi ve Grup Sızıntısı Önlemi

İki CSV dosyasında eksik (`NaN`) veya birebir tekrar eden satır bulunmamaktadır. Sütunların 17'si kategorik, 16'sı sayısal tiptedir. `G3` ortalaması Matematikte `10.42` (std `4.58`), Portekizcede `11.91` (std `3.23`) düzeyindedir. Matematikte `38` (`%9.6`), Portekizcede `15` (`%2.3`) öğrencinin final notu `0`'dır; çoğunda `G1 > 0` ve `absences = 0` olduğu için final sınavına girmeyen veya dersi bırakan öğrenciler olduğu görülmektedir.

`student-merge.R` içindeki 13 demografik ve ailevi sütunla yapılan eşleştirmede `382` ortak satır doğrulanmış, iki dersteki `G3` notları arasında `0.48` korelasyon bulunmuştur. İki veri setini `course` (`mat` / `por`) sütunuyla birleştirirken (`1044` satır), aynı öğrencinin hem eğitim hem test setine düşmesini engellemek için 13 ortak sütundan `student_group` kimliği üretilmiş ve hem eğitim/test ayrımında (`GroupShuffleSplit`, %20 test) hem de 5 katlı çapraz doğrulamada (`GroupKFold`) bu grup yapısı kullanılmıştır:

- Tekil öğrenci grubu sayısı: `662`
- Eğitim seti: `831` satır (`521` Portekizce, `310` Matematik, ortalama `G3 = 11.38`)
- Test seti: `213` satır (`128` Portekizce, `85` Matematik, ortalama `G3 = 11.21`)
- Eğitim ve test setleri arasındaki grup kesişimi: `0`

## Karar Ağacı Modellemesi ve Sonuçlar

Karar Ağaçları doğrusal olmayan eşikleri (`G2 == 0` veya `failures > 0` gibi) doğal biçimde yakaladığı ve öğretmen/rehberlik servisi tarafından doğrudan okunabilen kurallar ürettiği için ana model olarak tercih edilmiştir. Sınırsız derinlikli karar ağacının aşırı öğrenme (overfitting) davranışı budanmış karar ağaçları (`max_depth`, `min_samples_leaf`, `min_samples_split`, `ccp_alpha`) ve diğer referans modellerle karşılaştırılmıştır.

### Senaryo 1: Dönem İçi Tahmin (`G1` ve `G2` Dahil)

| Model | CV RMSE (5-Fold) | CV MAE | CV R2 | Train RMSE | Test RMSE | Test MAE | Test R2 |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Dummy (Ortalama) | 3.845 | 2.824 | -0.005 | 3.846 | 3.928 | 2.865 | -0.002 |
| Karar Ağacı (Sınırsız Derinlik) | 2.110 | 1.177 | 0.696 | 0.000 | 1.630 | 0.948 | 0.827 |
| Linear Regression | 1.662 | 1.024 | 0.813 | 1.543 | 1.444 | 1.011 | 0.865 |
| Ridge (alpha=10) | 1.651 | 1.009 | 0.815 | 1.545 | 1.444 | 1.009 | 0.865 |
| **Karar Ağacı (max_depth=4)** | **1.567** | **0.935** | **0.831** | **1.475** | **1.239** | **0.831** | **0.900** |
| Random Forest | 1.567 | 0.950 | 0.833 | 0.561 | 1.272 | 0.853 | 0.895 |
| Gradient Boosting | 1.512 | 0.918 | 0.845 | 1.015 | 1.334 | 0.906 | 0.884 |

Sınırsız derinlikli Karar Ağacı eğitim setini ezberlerken (`Train_RMSE = 0.000`), derinliği `max_depth=4` ve `min_samples_leaf=10` ile sınırlandırılan Karar Ağacı test setinde `RMSE = 1.239`, `MAE = 0.831` ve `R2 = 0.900` ile en düşük test hatasını vermiştir.

`G_ort`, `G_degisim` ve `G2_sifir_mi` özellikleri eklenip `GridSearchCV` uygulandığında elde edilen sonuçlar:

| Ayarlı Model (FE) | CV RMSE | Train RMSE | Test RMSE | Test MAE | Test R2 |
| --- | ---: | ---: | ---: | ---: | ---: |
| **Budanmış Karar Ağacı (FE)** (`max_depth=5, min_samples_leaf=15`) | **1.585** | **1.366** | **1.317** | **0.849** | **0.887** |
| Tuned Random Forest (FE) | 1.498 | 1.260 | 1.244 | 0.810 | 0.900 |
| Tuned Gradient Boosting (FE) | 1.467 | 1.325 | 1.264 | 0.865 | 0.896 |

Budanmış Karar Ağacı (`max_depth=5, min_samples_leaf=15, min_samples_split=10, ccp_alpha=0.0`) ilk 3 seviyede `G2`, `absences`, `G_ort` ve `romantic` değişkenleri üzerinden dallanmaktadır. Ayrıca iki dersi birleştirerek tek bir Karar Ağacı eğitmek, her iki dersi ayrı Karar Ağacı ile eğitmeye kıyasla test hatasını düşürmüştür:

- **Matematik Test:** Birleşik Karar Ağacı `RMSE = 1.349` (`R2 = 0.909`) vs Ayrı Karar Ağacı `RMSE = 1.536` (`R2 = 0.883`)
- **Portekizce Test:** Birleşik Karar Ağacı `RMSE = 1.296` (`R2 = 0.851`) vs Ayrı Karar Ağacı `RMSE = 1.464` (`R2 = 0.810`)

### Senaryo 2: Erken Uyarı Tahmini (`G1` ve `G2` Hariç)

Dönem başında hiç sınav notu yokken kurulan modellerin karşılaştırması:

| Model | CV RMSE (5-Fold) | CV MAE | CV R2 | Train RMSE | Test RMSE | Test MAE | Test R2 |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Karar Ağacı (Sınırsız Derinlik) | 4.639 | 3.308 | -0.482 | 0.000 | 4.443 | 3.141 | -0.282 |
| Dummy (Ortalama) | 3.845 | 2.824 | -0.005 | 3.846 | 3.928 | 2.865 | -0.002 |
| Linear Regression | 3.560 | 2.580 | 0.138 | 3.321 | 3.161 | 2.393 | 0.351 |
| Ridge (alpha=10) | 3.543 | 2.560 | 0.146 | 3.323 | 3.159 | 2.386 | 0.352 |
| **Karar Ağacı (max_depth=4)** | **3.464** | **2.541** | **0.184** | **3.208** | **3.147** | **2.394** | **0.357** |
| Gradient Boosting | 3.405 | 2.510 | 0.211 | 2.396 | 2.914 | 2.239 | 0.449 |
| Random Forest | 3.268 | 2.370 | 0.271 | 1.167 | 3.001 | 2.266 | 0.415 |

Yeni özellikler (`ebeveyn_egitim_ort`, `alkol_toplam`, `sosyal_yuk`, `devamsizlik_log`, `destek_sayisi`) ve `GridSearchCV` budaması sonrasında:

| Ayarlı Model (FE) | CV RMSE | Train RMSE | Test RMSE | Test MAE | Test R2 |
| --- | ---: | ---: | ---: | ---: | ---: |
| **Budanmış Karar Ağacı (FE)** (`max_depth=3, min_samples_leaf=15, ccp_alpha=0.1`) | **3.400** | **3.331** | **3.170** | **2.418** | **0.348** |
| Tuned Gradient Boosting (FE) | 3.330 | 2.869 | 2.901 | 2.212 | 0.454 |
| Tuned Random Forest (FE) | 3.295 | 2.396 | 2.982 | 2.287 | 0.423 |

Senaryo 2 Budanmış Karar Ağacı modelinde önem sırasına göre kullanılan 5 değişken `failures`, `absences`, `higher`, `destek_sayisi` ve `course` olmuştur. Ders bazında Portekizce test seti `RMSE = 2.640` (`MAE = 2.112`, `R2 = 0.384`), Matematik test seti `RMSE = 3.832` (`MAE = 2.879`, `R2 = 0.269`) sonuç vermiştir.

### Geçme / Kalma (`G3 >= 10`) Eşiği Üzerinden Karar Ağacı Başarısı

- **Senaryo 1 (Budanmış Regresyon Ağacı >= 10):** Doğruluk `0.878`, F1 skoru `0.914`
- **Senaryo 2 (Budanmış Regresyon Ağacı >= 10):** Doğruluk `0.775`, F1 skoru `0.850`
- **Senaryo 2 (`DecisionTreeClassifier`, `max_depth=4, min_samples_leaf=20`):** Doğruluk **`0.798`**, F1 skoru **`0.875`**

## Çalıştırma

```powershell
python -m venv .venv
.venv\Scripts\pip install -r requirements.txt
```

`ogrenci_basari_tahmini.ipynb` dosyasını açıp `.venv` çekirdeği ile tüm hücreleri çalıştırabilirsiniz.
