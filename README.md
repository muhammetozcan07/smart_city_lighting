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

# Bulgular
Aşağıdaki resimde derin pekiştirmeli öğrenme (DQN) ajanının 200 eğitim bölümü boyunca elde ettiği toplam ödül değerleri gösterilmektedir. Açık mavi çizgi her eğitim turunda elde edilen toplam ödülü, koyu mavi çizgi ise 10 turluk hareketli ortalamayı temsil etmektedir. Eğitimin ilk bölümlerinde ajan çevreyi keşfetme aşamasında olduğu için ödül değerlerinde büyük dalgalanmalar görülmektedir. Yaklaşık 20. bölümden sonra hareketli ortalamanın hızlı bir şekilde yükselmesi, ajanın çevreyi öğrenmeye başladığını göstermektedir. İlerleyen bölümlerde toplam ödül 400–440 aralığında kararlı bir seyir izlemekte ve yalnızca keşif stratejisinden (epsilon-greedy) kaynaklanan küçük dalgalanmalar gözlenmektedir. Eğitim sonunda hareketli ortalamanın yaklaşık 440 seviyesine ulaşması, DQN ajanının uygun aydınlatma kararlarını büyük ölçüde öğrenerek istikrarlı bir performans sergilediğini göstermektedir.

<img width="945" height="523" alt="image" src="https://github.com/user-attachments/assets/4eee494c-120e-44fb-9f3f-29ccc557943d" />

Aşağıdaki resimde ise sunulan canlı test çıktısı incelendiğinde, eğitilmiş derin pekiştirmeli öğrenme (DQN) ajanının çevresel değişkenleri son derece isabetli bir şekilde analiz ettiği ve veri setiyle tam uyumlu kararlar aldığı açıkça görülmektedir. Sisteme yansıyan anlık sensör verilerine göre; saat 02:00 itibarıyla trafik yoğunluğunun %16.4 gibi oldukça düşük bir seviyede seyrettiği ve ortalama araç hızının 47.5 km/s ile trafik akışının akıcı (sıkışıksız) olduğu tespit edilmiştir. Aynı zamanda yağışın bulunmaması (0.00 mm) ve hava koşullarının olağan düzeyde (Hava Kodu: 3 - Bulutlu) olması, yoldaki sürüş güvenliğini tehdit edecek meteorolojik bir risk faktörü barındırmamaktadır. Gecenin ilerleyen saatleri olmasına rağmen, hem trafik hacminin düşüklüğü hem de olumlu hava şartları göz önüne alınarak veri setindeki ideal aydınlatma seviyesi %25 olarak belirlenmiştir. Yapay zeka modelinin de o anki durumu (state) başarıyla çözümleyerek birebir aynı oranda (%25) aydınlatma kararı aldığı ve bu doğru tahmini sayesinde maksimum başarı değeri olan 10.00 tam puanı kazandığı kaydedilmiştir. Bu sonuç, geliştirilen modelin sadece kuralları ezberlemediğini; aynı zamanda gecenin geç saatlerinde boş ve yağışsız bir yolda ışıkları tamamen açmanın (%100) bir enerji israfı olacağını kavrayarak, güvenlik ve enerji tasarrufu arasındaki optimum dengeyi başarıyla kurabildiğini kanıtlamaktadır.

<img width="478" height="359" alt="image" src="https://github.com/user-attachments/assets/50839437-127c-48f8-be1b-65d2c0a1a763" />




