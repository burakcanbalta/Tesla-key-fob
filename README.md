# Tesla Key Fob Saldırıları

![aaa](https://github.com/user-attachments/assets/9bab4af5-e828-4546-91d6-084676cb1e55)


## Özet (Executive summary)

Kısa: Tesla araçlarının pasif anahtarsız giriş (Passive Keyless Entry & Start — PKES) ve telefon‑anahtarlık (phone-as-key) sistemlerinde 2016'dan itibaren bir dizi farklı güvenlik zafiyeti tespit edildi: zayıf/özel kriptografi (DST40), konfigürasyon/downgrade hataları, imzasız/yanlış doğrulanan firmware güncellemeleri, ve Bluetooth Low Energy (BLE) üzerinden gelişmiş röle (relay) yöntemleri. Ayrıca araç içi bileşenlere pivot yaparak CAN bus'a erişim sağlayan zincir zafiyetler de gösterildi. Bu raporda hem tarihçe hem de teknik neden‑sonuç ilişkileri, kullanılan saldırı türleri sınıflaması, araştırma‑düzeyinde kullanılan donanım kategorileri, tespit/adalet (forensics) işaretleri ve mühendislik / kullanıcı odaklı önlemler anlatılmaktadır.

---

## 1. Arka plan — Nedir, ne işe yarar?

**PKES**: Araçlar yakınlarındaki yetkili bir anahtarlık veya taşınabilir cihazı (fob / telefon) tespit edip kullanıcı etkileşimi olmadan kilidi açar, motoru çalıştırır. Amaç: Kullanıcı deneyimini (UX) kolaylaştırmak — cebinden anahtar çıkarmaya gerek kalmadan kapıyı aç, aracı çalıştır.

Bu kolaylık aynı zamanda **saldırganlara** yeni yüzeyler açar: radyo protokolleri, link katmanı, uygulama katmanı, ve araç içi ağlarda zincirsel (chained) zafiyetler.

---

## 2. Saldırı türleri (taxonomy) — yüksek seviyede

### 2.1 Relay (röle) saldırıları

* **Tanım (yüksek seviye):** Yetkili cihaz ile araç arasındaki radyo sinyalini iki nokta arasında iletme (bir nevi sinyal uzatma). Bu sayede araç, anahtarın menzilinde olduğunu düşünür.
* **Neden etkili:** Mesafe kontroline dayanılan sistemlerde (RSSI, gecikme toleransları) yazılım/işlemci gecikmesi normal varyasyon içine girerse saldırı görünmez olur.
* **Vaka örnekleri:** Model 3/Model Y BLE ile yeni nesil relay teknikleri (link layer relay) ile başarı gösterildi; NCC Group bu kategoride önemli ilerleme raporladı.

### 2.2 Replay / Jamming / Downgrade

* **Replay:** Önce kaydedilmiş bir mesajın yeniden gönderilmesi. Rolling‑code uygulamaları karşısında belirli tasarım hataları replay'e izin verir.
* **Jamming + Rollback:** Araç ve fob arasında ileri sürülen daha eski (zayıf) şifreleme protokoluna zorlayarak downgrade eden saldırılar KU Leuven tarafından gösterildi.

### 2.3 Kopyalama (cloning) / Zayıf kriptografi

* **Tanım:** Key‑fob içindeki kriptografik anahtarın kısmen veya tamamen açığa çıkarılması. Örnek: DST40 gibi zayıf/özel cipher'lar.
* **Konsept:** Zayıf anahtar alanı veya konfigürasyon hatası, brute‑force veya kriptoanalize indirgenebilir hâle getirir.

### 2.4 Firmware manipulation / provisioning flow hataları

* **Tanım:** Anahtarlık veya araç tarafındaki firmware güncelleme/eşleme akışındaki eksik doğrulama (imza vs. imzasız paket) nedeniyle kötü amaçlı yazılım yüklenmesi/cihaz davranışının değiştirilmesi.
* **Konsept:** Eğer cihaz imza doğrulaması yapmıyorsa, sahte paketlerle davranış değişikliği sağlanabilir.

### 2.5 Zincirsel saldırılar — pil göstergesi: wirelessly → gateway → canbus

* **Tanım:** Başarılı bir kablosuz erişim sonrası araçtaki başka bileşenlere (ör. Gateway ECU, telematik) pivot yapılarak CAN bus'a erişilip kritik sistemler etkilenebilir.
* **Vaka:** Keen Security Lab'in 2016 çalışması bu zincirsel saldırının gerçek dünyada mümkün olduğunu gösterdi.

---

## 3. Tarihçe — önemli olaylar

* **2016 — Keen Security Lab:** Tesla Model S üzerinde web‑browser + Wi‑Fi/zincirsel zafiyet zinciri ile uzaktan CAN erişimi göstermiştir. Bu çalışma otomotivde OTA code‑signing önemini vurguladı.
* **2018 — KU Leuven (Model S):** Mevcut key fob sisteminin DST40 gibi zayıf/proprietary cipher kullandığı, 40‑bit benzeri zayıflıkların araçların klonlanmasına yol açtığı duyuruldu.
* **2019‑2020 — KU Leuven follow‑ups (Model S & X):** Tesla'nın yeni fob'larında iyileştirmeler olsa da konfigürasyon/downgrade hataları sonucu yine kopyalama/analizin mümkün olduğu gösterildi.
* **2022 — NCC Group:** BLE link‑layer relay tekniği ile Model 3 phone‑as‑key sistemlerinde relay saldırıları gösterildi; ek gecikme marjı (ör. onlarca ms) sistemin kabul sınırında kaldığı için saldırı pratik oldu.

(Kaynaklar: KU Leuven/imec duyuruları, Wired makaleleri, NCC Group teknik yayınları, BlackHat/Keen whitepaper.)

---

## 4. Zafiyet kökenleri — neden bu kadar kırılgan?

1. **Miras / Proprietary kriptografi:** Eskiden kullanılan DST40 gibi özel, yeterince incelenmemiş ve kısa bit‑uzunluklu algoritmalar ciddi risk getirir. Açık, modern, incelenmiş algoritmalar (AES, EC‑based, standart protokoller) tercih edilmelidir.
2. **Yanlış konfigürasyon / downgrade izinleri:** Cihazların daha eski moda geri döndürülmesine izin vermek kritik bir hatadır.
3. **Eksik mutual authentication:** Araç ve fob arasındaki tek taraflı veya zayıf kimlik doğrulama, relay/replay'e zemin hazırlar.
4. **İmzalanmamış veya yanlış doğrulanmış firmware:** Firmware veya yapılandırma paketleri kriptografik olarak doğrulanmıyorsa cihaz kolaylıkla manipüle edilebilir.
5. **Uzaklık ölçümü eksikliği / güvenilir mesafe ölçümü yok:** Sadece RSSI veya gecikme toleransına dayanan karar mekanizmaları, röle saldırılarına açık olur.
6. **Bileşen etkileşimleri ve atacak zincirleri:** Birden fazla bileşende küçük zafiyetler zincirlenerek kritik etkiye ulaşabilir.

---

## 5. Araştırma / deneysel donanım kategorileri

* **SDR (Software Defined Radio) ve spektrum analizörleri:** PKES, rolling‑code protokollerinin radyo trafiğini incelemek ve protokol özelliklerini anlamak için kullanılır.
* **BLE sniffers / link layer debugging araçları:** Phone‑as‑key ve BLE tabanlı protokoller için zamanlama/cevap örüntülerini analiz etmek üzere.
* **CAN bus sniffers / logging araçları:** Araç içi ağ trafiğinin kayıt altına alınması, mesaj yapılarının analizi ve anomalilerin tespiti için.
* **Gömülü geliştirme kartları / SBC'ler (ör. araştırma‑class single board computers):** Ölçümler, prototiplemeler ve test otomasyonu için.
* **Donanım programlama / JTAG / SWD ekipmanı (sadece sahip olunan cihazlarda):** Firmware analizleri için kullanılır.

---

## 6. Case studies — ne oldu, neden kritikti? (yüksek seviyede)

### 6.1 KU Leuven — Model S (DST40 & downgrade)

* Problem: Eskiden kullanılan şifreleme algoritmasının zayıflığı ve yeni fob ve araç yazılımındaki konfigürasyon hatası sayesinde saldırı teorik olarak olasıydı.
* Sonuç: Tesla yazılım güncellemesi ve bazı fob değişiklikleriyle düzeltildi; kullanıcılar ek güvenlik (PIN) kullanmaya yönlendirildi.

### 6.2 KU Leuven / imec — Model X

* Problem: Keyless entry akışının bazı adımlarında yeterli doğrulama yapılmadığı ve provisioning/güncelleme mantığında açıklıklar olduğu görüldü. Bu da daha karmaşık senaryolarda fob davranışının manipüle edilebileceğini gösterdi.
* Etki: Hem yazılım yamaları hem de önerilerle kapatıldı.

### 6.3 NCC Group — Model 3 BLE relay

* Problem: Link‑layer relay tekniğiyle, BLE bağlantısının şifrelenmiş olması veya standart GATT zamanlaması içinde kalması gereken varsayımları atlatabiliyor.
* Önem: Telefon uzaktayken bile aracın açılabilmesi gösterildi — araçlar için yeni bir tehdit vektörü.

### 6.4 Keen Security Lab 2016 — Web/OTA → CAN pivot

* Problem: Aracın web tarayıcısının kötü amaçlı Wi‑Fi erişim noktasıyla etkileşimi aracılığıyla zincirsel bir saldırı gerçekleştirildi; sonuçta CAN verisi manipüle edilebildi.
* Etki: OTA code signing ve hızlı yamalarla kısmen önlendi; yine de gösterilen kavram çok etkileyiciydi.

---

## 7. Tespit, adli (forensics) ipuçları — neler bakılır?

Teknik izleme önerileri:

* **Anahtar eşleştirme/kaydı değişiklikleri:** Araç tarihçesinde beklenmeyen yeni fob eşleştirmeleri.
* **Gateway/telematik bağlantı kayıtları:** Olağandışı Wi‑Fi SSID, yetkisiz OTA denemeleri.
* **CAN bus anomalileri:** Normalden farklı mesaj türleri veya zamanlamalar.
* **Fob/telefon tarafı kayıtları:** Mobil uygulama kimlik doğrulama zamanları, şüpheli yeniden bağlanma olayları.
* **RSSI / distance metrics logları:** Ani ve fiziksel olarak tutarsız mesafe okumaları.

---

## 8. Mühendislik / ürün tasarımında alınabilecek önlemler

**Mimari ve protokol önerileri (yüksek seviye):**

1. **Açık, incelenmiş kriptografi kullanın:** Özel ve kısa anahtar uzunluklarından kaçının. Anahtar yönetimi ve anahtar uzunluğu standartlara uygun olsun.
2. **Mutual authentication & session binding:** Hem fob hem araç karşılıklı olarak kimlik doğrulaması yapmalı; oturumlar sağlam şekilde bağlanmalı.
3. **Signed firmware / secure boot:** Firmware paketleri ve boot zinciri imzalanmalı; tedarik zinciri doğrulanmalı.
4. **Distance bounding / time‑of‑flight (UWB gibi):** Röle saldırılarına direnci artırır.
5. **Provisioning flow hardening:** Fob eşleme süreçleri fiziksel/kriptografik kanıt gerektirmeli.
6. **Anomali tespiti ve telemetri:** Araç içi davranışlar, RSSI/timing anomalileriyle sürekli izlenmeli.
7. **Güncelleme süreçlerinin test ve verifikasyonu:** Backwards compatibility kararları risk‑maliyet analizine tabi tutulmalı.

---

## 9. Kullanıcı seviyesinde alınabilecek önlemler

* **PIN‑to‑Drive (varsa) aktif et:** Aracın yalnızca fiziksel/şifreli onay ile çalışmasını sağlar.
* **Passive entry kapatma seçeneği:** Eğer varsa, pasif giriş devre dışı bırakılsa bile anahtar‑button kullanımı gerekir.
* **Fob'ları ve telefonları güvenli sakla:** Fiziksel güvenlik.
* **Araç yazılımını güncel tut:** Üretici yamaları kritik.

---

## 10. Konseptual adım adım (Saldırı akışlarının **konseptual** gösterimi)

### 10.1 Relay (Röle) — Konseptual adımlar
1. **Hazırlık:** Saldırgan, hedef bölgede iki iletişim noktası belirler: araç yakınında bir cihaz (vehicle‑side) ve anahtar/telefon yakınında bir cihaz (key‑side).
2. **Senaryo oluşturma:** Araç ve anahtar arasındaki protokol akışlarına hâkim olmak için pasif gözlem yapar; zamanlama ve cevap örüntülerini inceler.
3. **Şeffaf taşıma:** Araç tarafı ve anahtar tarafı arasındaki radyo sinyallerini mümkün olduğunca şeffaf bir şekilde birbirine iletir (konsept olarak). Sistem, eklenen gecikmeyi normal varyasyon olarak algılayabilirse unlock/start tetiklenir.
4. **Sonuç değerlendirme:** İşlem başarılıysa araç yetkisiz olarak açılabilir; başarısızsa ölçülen gecikme/RTT verileri savunmacı için tespit eşiği belirlemede kullanılabilir.

### 10.2 Replay / Kopyalama — Konseptual adımlar
1. **Eavesdrop (dinleme):** Protokolden gelen mesaj akışları pasif olarak kaydedilir.
2. **Analiz:** Rolling‑code/nonce kullanımı analitik olarak değerlendirilir; zayıflık varsa tekrar kullanım senaryoları teorik olarak oluşturulur.
3. **Denetim:** Savunma amacıyla, kaydedilen örüntüler aracın loglarıyla karşılaştırılır; eşzamanlılık, sıra numarası atlamaları tespit edilir.

### 10.3 Firmware/Provisioning Zafiyeti — Konseptual adımlar
1. **Provisioning flow haritalama:** Fob ve araç arasındaki eşleştirme, üretim anahtarının yaşam döngüsü, OTA akışı çıkarılır.
2. **Doğrulama kontrolü:** İmza/sertifika mekanizmalarının varlığı ve geçerliliği denetlenir.
3. **Güvenlik açığı tespiti:** İmza doğrulaması yoksa veya yanlış yapılandırılmışsa, tasarım düzeyinde uyarılar üretilir.

---

## 11. Güvenli laboratuvar test planı

**Adım 1 — İzin & yönetim:** Test için açık yazılı izin/lojistik sağlanır; scope, zaman çizelgesi ve rollback planı belirlenir.

**Adım 2 — İzole test ortamı kurulumu:** Faraday çantası/oda, izole CAN bench, test fob’ları ve araç simülatörleri hazırlanır.

**Adım 3 — Baseline toplama:** Normal çalışma sırasında telemetri, RSSI dağılımları, protokol zamanlamaları toplanır.

**Adım 4 — Konseptual deneyler:** Relay etkinliğinin teorik sınırları laboratuvarda ölçülür — hedef amaç, savunma eşiğini (latency envelope) belirlemektir; **doğrudan aracı hedeflemeyecek şekilde** testler yürütülür.

**Adım 5 — Provisioning/firmware doğrulama:** Update paketlerinin imza doğrulama adımları, provisioning flow’larının mantıksal güvenliği değerlendirilir.

**Adım 6 — Anomali tespit kuralları geliştirme:** CAN/telemetri/RSSI anomali kuralları oluşturulur ve gerçek‑zamanlı uyarı sistemleri denetlenir.

**Adım 7 — Raporlama ve responsible disclosure:** Tespit edilen bulgular üreticiye/ilgili CERT’e koordineli biçimde bildirilir.

---

## 12. Uygulama ve ürün ek önerileri

1. **UWB prototipi entegrasyonu:** Distance bounding deneyleri başlatın; fiziksel layer testleriyle relay toleranslarını ölçün.
2. **Fob tasarımında secure element kullanımı:** Anahtarların donanım‑bazlı güvenli saklanması sağlanmalı.
3. **OTA code signing workflow:** Üretimden dağıtıma kadar imzalama anahtarlarının korunması ve revocation planları oluşturulmalı.
4. **Telemetri & ML tabanlı anomaly detection:** RSSI/timing/CAN örüntülerinden anormal senaryoları tespit eden modeller geliştirilmeli.

---

## 12. Sonuç

Tesla ve çağdaş otomotiv üreticilerinin PKES/phone‑as‑key sistemleri UX açısından mükemmel olsa da arkasında karmaşık güvenlik gereksinimleri vardır. Tarihçe gösteriyor ki hem tasarım/konfigürasyon hataları hem de protokol seçimleri pratik zafiyetlere yol açabiliyor. Güncel araştırma, daha güvenli mesafe ölçümü (UWB gibi), güçlü anahtar yönetimi, imzalı firmware ve kapsamlı telemetriyle bu risklerin büyük kısmını azaltabileceğimizi gösteriyor.
