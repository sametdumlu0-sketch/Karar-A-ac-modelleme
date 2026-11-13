# LinkedIn Paylaşım Metinleri ve Görsel Kullanım Rehberi

Projeyle birlikte paylaşabileceğiniz iki farklı LinkedIn metin taslağı (detaylı teknik öğrenci paylaşımı ve daha kısa özet paylaşım) aşağıda yer almaktadır. Paylaşıma eklemek üzere proje klasöründe iki görsel üretilmiştir:

- `linkedin_karar_agaci_grafik.png`: Model karşılaştırması, budama (pruning) etkisi, özellik önemliliği ve gerçek/tahmin dağılımını gösteren 4 panelli özet grafik.
- `linkedin_agac_semasi.png`: Dönem içi ve erken uyarı senaryolarında Karar Ağacının öğrendiği karar kurallarını gösteren şema.

---

## Seçenek 1: Doğal Öğrenci / Proje Hikayesi Odaklı Paylaşım (Önerilen)

Matematik ve Portekizce derslerine ait öğrenci performans veri setleri (1.044 kayıt, 33 değişken) üzerinde çalışarak yıl sonu başarı notunu (G3) tahmin eden Karar Ağacı (Decision Tree) odaklı bir makine öğrenmesi projesi hazırladım.

Veriyi incelerken ve modeli kurarken karşılaştığım birkaç önemli detay çalışmanın yönünü belirledi:

1. Veri sızıntısını (data leakage) önlemek:
İki ayrı CSV dosyasını birleştirirken 13 demografik ve ailevi özellik üzerinden 382 öğrencinin her iki derste de yer aldığını doğruladım. Rastgele train/test ayrımı yapsaydım aynı öğrencinin bir dersi eğitim, diğer dersi test setine düşecekti. Bunu engellemek için 662 tekil öğrenci grubu üzerinden GroupShuffleSplit ve 5 katlı GroupKFold kullandım.

2. Karar Ağacında budamanın (pruning) etkisi:
Sınırsız derinlikli DecisionTreeRegressor eğitim verisini tamamen ezberleyerek (Train RMSE: 0.000) test setinde 1.630 RMSE'de kaldı. Derinlik ve yaprak kısıtlaması (max_depth=4, min_samples_leaf=10) uyguladığımda ise aşırı öğrenme ortadan kalktı ve model test setinde 1.239 RMSE, 0.831 MAE ve 0.900 R² skoru ile Random Forest (1.272 RMSE) ve Gradient Boosting (1.334 RMSE) modellerini dahi geride bıraktı.

3. Dönem başında sınav notları olmadan erken uyarı (G1 ve G2 hariç):
İlk iki dönem sınav notlarını veri setinden çıkardığımızda modelin en çok geçmiş ders başarısızlıkları (failures), devamsızlık (absences), üniversite hedefi (higher) ve alınan ek destek sayısına dayandığını gördüm. Sadece bu özelliklerle kurulan budanmış sınıflandırma karar ağacı (DecisionTreeClassifier), öğrencinin dersten geçip kalacağını (G3 >= 10) %79.8 doğruluk ve 0.875 F1 skoru ile tahmin edebildi.

Veri keşfi, karar ağacı şemaları, hata analizi ve tüm Python kodlarının yer aldığı notebook'a GitHub üzerinden ulaşabilirsiniz:
[GitHub Repo Linki]

#VeriBilimi #MakineÖğrenmesi #Python #ScikitLearn #DecisionTree #KararAğacı #VeriAnalizi

---

## Seçenek 2: Kısa ve Öz Paylaşım

Öğrenci başarı verileri (Matematik ve Portekizce, 1.044 kayıt) üzerinde Karar Ağacı (DecisionTreeRegressor & DecisionTreeClassifier) kullanarak dönem sonu not tahmini yaptığım projemi tamamladım.

Çalışmada öne çıkan noktalar:
- İki veri setinde ortak olan 382 öğrencinin eğitim ve test setleri arasında sızıntı yaratmaması için GroupShuffleSplit ve GroupKFold ile grup bazlı doğrulama kurguladım.
- Budanmamış karar ağacındaki ezberleme sorununu (Train RMSE: 0.000 -> Test RMSE: 1.630) derinlik ve yaprak budamasıyla çözerek dönem içi senaryoda Test R² skorunu 0.900'a (RMSE: 1.239) taşıdım.
- Sınav notları (G1 ve G2) olmadan kurulan erken uyarı senaryosunda ise geçmiş başarısızlık sayısı, devamsızlık ve yükseköğrenim hedefi değişkenleriyle geçme/kalma durumunu %79.8 doğruluk (0.875 F1) ile sınıflandıran anlaşılır bir karar ağacı elde ettim.

Kodlar, karar ağacı kuralları ve detaylı analiz raporu için GitHub bağlantısı:
[GitHub Repo Linki]

#MachineLearning #Python #DataScience #DecisionTrees #ScikitLearn
