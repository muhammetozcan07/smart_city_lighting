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

Eğitim sürecinin merkezinde, derin pekiştirmeli öğrenme literatüründe oldukça kararlı sonuçlar veren bir DQN (Deep Q-Network) ajanı yer almaktadır. Bu ajan, "Experience Replay Buffer" olarak adlandırılan gelişmiş bir hafıza mekanizması kullanarak, her adımda karşılaşılan durumları, yapılan eylemleri, elde edilen ödülleri ve bir sonraki durum geçişlerini bir depo yapısında saklar; bu sayede geçmiş deneyimler rastgele gruplar halinde yeniden eğitilerek modelin verilerin zamansal korelasyonuna sıkışıp kalması (overfitting) engellenir. Epsilon-Greedy politikasıyla yönetilen ajan, eğitim boyunca keşif ve sömürü dengesini kurarak, Q-Network'ünü sürekli olarak backpropagation ve Adam optimizasyon algoritmalarıyla güncelleyerek en uygun politika matrisine ulaşmaya çalışır. 

Sistemin davranışını belirleyen aksiyon uzayı, 0, 25, 50, 100 değerlerinden oluşan dört farklı aydınlatma seviyesiyle tanımlanmıştır; bu ayrık yapı, aydınlatma armatürlerinin gerçek dünyadaki kademeli çalışma yeteneklerini temsil eder ve DQN ajanı, anlık trafik ve hava durumuna göre bu dört seviyeden en optimal olanını seçerek kararlarını verir. Bu kararları yönlendiren ödül sistemi ise sadece enerji verimliliği odaklı olmayıp, kentsel güvenliği gözeten üçlü bir yapıda kurulmuştur. Ödül puanlama mantığında, ajanın seçtiği aydınlatma seviyesi verideki ideal değerle tam olarak eşleştiğinde sistem ajana +10 puan kazandırmakta, tam eşleşme sağlanamadığında ise ideal değer ile seçilen değer arasındaki farkın mutlak değerinin 25'e bölümünün negatifini ceza puanı olarak yansıtmaktadır. İkinci olarak, ajanın tercih ettiği aydınlatma şiddeti ideal değerden daha yüksek olduğunda, gereksiz enerji tüketimini engellemek adına sisteme -1 puanlık bir enerji israf cezası uygulanmaktadır. Üçüncü ve en kritik olanı ise, yağışlı ve düşük görüş mesafesinin olduğu riskli hava koşullarında ideal aydınlatma ihtiyacı sıfırdan büyük olmasına rağmen ajan aydınlatmayı tamamen kapatmayı (seviye 0) tercih ederse, sisteme -5 puanlık ağır bir güvenlik ihlali cezası uygulanmaktadır. 

<img width="885" height="591" alt="image" src="https://github.com/user-attachments/assets/30814ba0-39f9-4c91-b4f2-debd0c37ca9e" />


