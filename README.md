# Akıllı Şehir Aydınlatmasında Derin Pekiştirmeli Öğrenme: Berlin Örneği
Bu çalışma, Berlin kentsel hareketlilik verileri kullanılarak, kentsel aydınlatma sistemlerinin enerji verimliliğini artırmayı ve trafik güvenliğini optimize etmeyi amaçlayan Derin Pekiştirmeli Öğrenme (DQN) tabanlı akıllı bir kontrol mekanizması geliştirmeyi hedeflemektedir. Araştırma kapsamında, TomTom servisinden elde edilen saat verisi, araç hızı ve trafik yoğunluğu verileri ile Open-Meteo servisinden sağlanan yağış ve hava durumu verileriyle birleştirilerek hibrit bir veri seti oluşturulmuştur. Geliştirilen Derin Pekiştirmeli Öğrenme modeli, Berlin'in altı aylık verileri üzerinde eğitilerek, ortamdaki değişkenlere göre aydınlatma seviyesini otonom bir şekilde optimize etmeyi öğrenmiştir. Elde edilen bulgular, modelin özellikle düşük trafik hacmi ve açık hava koşullarında ışık şiddetini kısarak önemli ölçüde enerji tasarrufu sağladığını; olumsuz hava koşulları ve yoğun trafik durumlarında ise güvenlik öncelikli davranarak aydınlatma seviyesini artırdığını kanıtlamaktadır. Çalışma, yapay zeka destekli otonom sistemlerin, kentsel enerji yönetimi ile güvenlik arasındaki dengeyi başarıyla kurabildiğini ve akıllı şehir uygulamaları için sürdürülebilir bir altyapı sunduğunu ortaya koymaktadır.
# Veri Seti
## Trafik Verileri

•	Speed [kmh] (Hız): Araçların o anki seyir hızıdır. Standartlaştırılmış (Z-skor) bir değer olduğu için ortalaması 0'dır. Pozitif değerler ortalamanın üstünde yüksek hızı, negatif değerler ise yavaşlamış trafiği temsil eder. Yapay zeka, düşük hızlarda (yüksek trafik veya yaya riski) aydınlatmayı daha dikkatli yönetmeyi öğrenir.

•	Congestion level [%] (Trafik Yoğunluğu): Trafiğin sıkışıklık düzeyidir. Yine standartlaştırılmış bir değerdir. Yüksek trafik yoğunluğu, caddede daha fazla araç ve dolayısıyla daha fazla görünürlük ihtiyacı (güvenlik) demektir.

•	hour_sin (Sinüs) ve hour_cos (Kosinüs): Saatin döngüsel pozisyonunun dikey ve yatay bileşleridir. Bu iki değer birlikte, modelin günün hangi vaktinde olduğunu (sabah, öğle, gece) hatasız bir şekilde anlamasını sağlar.

<img width="790" height="354" alt="image" src="https://github.com/user-attachments/assets/982b5e4f-5308-44b2-be70-9b8f3f0eaf37" />

## Hava Durumu Verileri

•	precipitation (Yağış): Milimetrik bazda yağış miktarıdır. Standartlaştırılmıştır. Değer arttıkça yapay zeka kötü hava koşullarının yarattığı riskleri hesaba katarak "Güvenlik Cezası" tetiklenmesin diye ışığı daha dikkatli yönetir.

•	weather_code (Hava Durumu Kodu): Kategorik bir veridir. 0 (açık hava) ile 75 (yoğun kar) arasında değişir. Ajan, 51 ve üzerindeki kodları (yağışlı/karlı havalar) "kötü hava" olarak tanımlayıp ışık seviyesini daha yüksek tutmaya çalışır.

<img width="945" height="392" alt="image" src="https://github.com/user-attachments/assets/cecf3c43-ddd7-4441-bd51-a3222131b154" />

# Metodoloji

<img width="885" height="591" alt="image" src="https://github.com/user-attachments/assets/30814ba0-39f9-4c91-b4f2-debd0c37ca9e" />


