# Laboratuvar 7: AI Safety Challenge (Yapay Zeka ve Fonksiyonel Emniyet Entegrasyonu)
**Makine Adı:** UKM-OPT · Kameralı Optimizasyonlu Otomatik Boy Kesme Makinesi  
**Bootcamp Sürümü:** Safety 4.0 Bootcamp — UKM-OPT  
**Çıktı Belgesi:** AI Fail-Safe Decision (YZ Hata Güvenliği Tasarımı)  
**Evidence Pack Karşılığı:** `Evidence Pack EP 03 — Safety Functions` & `EP 15 — AI Risk Management`  
**İlgili Standartlar & Mevzuat:** ISO/IEC 42001 (Yapay Zeka Yönetim Sistemi), AB Yapay Zeka Yasası (EU AI Act 2024/1689), AB Makine Tüzüğü (EU 2023/1230 Madde 10 & Ek III), ISO 13849-1

---

## 👨‍🏫 Mentor Notu: "Prediction ≠ Safety" (Tahmin Emniyet Değildir!)
Yazılım mühendisliğinde ve veri biliminde derin öğrenme modellerinin başarısı çoğunlukla doğruluk (accuracy), F1-skoru veya AUC metrikleriyle ölçülür. Örneğin %98.5 doğruluk oranına sahip bir optik budak ve kusur algılama modeli bir yazılımcı için "harika" sayılabilir.

Ancak **Kıdemli bir Emniyet Mühendisi (Safety Engineer)** gözüyle baktığımızda gerçek şudur:
> **"Tahmin, Emniyet Değildir (Prediction $\neq$ Safety)!"**

* **Determinizm vs Olasılık:** Fonksiyonel emniyet (ISO 13849 / IEC 62061), her çalışma döngüsünde kesin ve deterministik bir koruma talep eder. %98.5 doğruluk demek, her 100 parçadan birinde veya ikisinde modelin ölümcül bir hata yapabilmesi demektir. Bu oran endüstriyel emniyette asla kabul edilemez!
* **Mimari İlke:** Yapay zeka modeli akıllı bir optimizasyon karar motorudur; ancak **insan canını koruyan emniyet mekanizması YZ'nin ve IPC'nin tamamen DIŞINDA, bağımsız donanım ve kural tabanlı katmanlarda (Guardrails) tanımlanmalıdır.**

---

## 🏗️ YZ Emniyet ve Hata-Güvenli (Fail-Safe) Mimari Şeması

Aşağıdaki şema, UKM-OPT makinesinde görüntü işleme modelinden fiziksel servo tahriklere kadar uzanan çok katmanlı denetim ve yedekleme (fallback) mekanizmasını göstermektedir:

```mermaid
flowchart TB
    subgraph SENSING["1. Algılama ve Girdi Bütünlüğü (Input Verification)"]
        CAM["Endüstriyel Kamera<br/><i>(Ahşap / Profil Optik Tarama)</i>"]
        LIVENESS["Girdi Bütünlüğü Denetimi<br/><i>- Kare Sayacı & Zaman Damgası<br/>- Parlaklık / Histogram Analizi<br/>- Donuk Kare / Sahte Veri Tespiti</i>"]
        CAM --> LIVENESS
    end

    subgraph AI_CORE["2. Yapay Zeka Karar Katmanı (AI Engine / IPC)"]
        direction TB
        MODEL["Optimizasyon & Kusur Algılama Modeli<br/><i>(Boy Kesim Optimizasyonu & Budak Tespiti)</i>"]
        CONF{"Model Güven Skoru (Confidence)<br/>$\ge$ %95 Eşik Değeri mi?"}
        
        LIVENESS -->|Bütünlük Doğrulandı| MODEL
        LIVENESS -.->|Bozuk / Donmuş Kare| CORRUPT_ALARM["🚨 GİRDİ HATASI: Kesimi Durdur & Alarm Ver"]
        MODEL --> CONF
    end

    subgraph ARBITRATION["3. Bağımsız Güvenlik Katmanı (Safety Arbitration & Guardrails)"]
        direction TB
        HARD_LIMITS["Donanımsal & PLC Güvenlik Sınırları (Hard Limits)<br/><i>(YZ asla maksimum hız veya strok sınırını aşamaz)</i>"]
        SECONDARY_CHECK["Bağımsız İkinci Kontrol<br/><i>(Lazer Mesafe Sensörü & Enkoder Doğrulaması)</i>"]
        HUMAN_SAMPLE["Periyodik İnsan Numune Kontrolü (Human-in-the-Loop)"]
        
        CONF -- "EVET ($\ge$ %95)" --> SECONDARY_CHECK
        SECONDARY_CHECK --> HARD_LIMITS
        
        CONF -- "HAYIR (< %95)" --> LOW_CONF["Parçayı Iskartaya / İncelemeye Ayır"]
        LOW_CONF --> REPEAT_CHECK{"Art Arda Düşük Güven mi?"}
        REPEAT_CHECK -- "EVET" --> FALLBACK["FALLBACK: Klasik Deterministik<br/>Sabit Boy Kesim Moduna Geç"]
        REPEAT_CHECK -- "HAYIR" --> HUMAN_SAMPLE
    end

    subgraph EXECUTION["4. PLC ve Sahada Güvenli İcra"]
        direction TB
        PLC_CMD["PLC Kesim ve Besleme Komutu"]
        SAW_SERVO["Testere Bıçağı & Malzeme Besleme Motorları"]
        
        HARD_LIMITS --> PLC_CMD
        FALLBACK --> PLC_CMD
        PLC_CMD --> SAW_SERVO
    end

    subgraph SAFETY_INTERLOCK["5. Donanımsal Fonksiyonel Emniyet (Z1 - Tamamen Bağımsız)"]
        direction TB
        ISOLATED_SPLC["Safety PLC & Işık Perdesi & STO<br/><i>(YZ kararı ne olursa olsun insan yaklaşırsa gücü keser!)</i>"]
    end

    SAW_SERVO -.->|Fiziksel Güç Kesintisi (STO)| ISOLATED_SPLC

    classDef sens fill:#e1f5fe,stroke:#0288d1,stroke-width:2px;
    classDef ai fill:#fff8e1,stroke:#ffa000,stroke-width:2px;
    classDef arbit fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px;
    classDef exec fill:#e8f5e9,stroke:#388e3c,stroke-width:2px;
    classDef safety fill:#ffebee,stroke:#d32f2f,stroke-width:3px,color:#b71c1c;

    class SENSING,CAM,LIVENESS sens;
    class AI_CORE,MODEL,CONF ai;
    class ARBITRATION,HARD_LIMITS,SECONDARY_CHECK,HUMAN_SAMPLE,FALLBACK,LOW_CONF arbit;
    class EXECUTION,PLC_CMD,SAW_SERVO exec;
    class SAFETY_INTERLOCK,ISOLATED_SPLC,CORRUPT_ALARM safety;
```

---

## 🎯 Beş Temel Emniyet Prensibi

1. **Fail-Safe (Hata-Güvenli):** Sistem donanım, yazılım veya algoritma hatasında daima önceden tanımlanmış en emniyetli konuma (Safe State) geçmelidir.
2. **Human Override (İnsan Müdahalesi Önceliği):** Operatör, acil stop butonu veya acil mod seçici ile yapay zekanın tüm kararlarını anında hükümsüz kılabilmelidir (Human-in-the-loop / Human-on-the-loop).
3. **Continuous Monitoring (Sürekli Girdi ve Süreç İzleme):** Kamera görüntüsünün kare sürekliliği, aydınlatma stabilitesi ve donanımsal sensör verileri her çevrimde doğrulanmalıdır.
4. **Confidence Threshold (Güven Skoru Eşiği):** Yapay zeka tahmin güveni belirlenen eşik değerinin (ör. %95) altına düştüğünde parça otomatik işlenmez.
5. **Fallback (Yedek Deterministik Mod):** Yapay zeka modülü devre dışı kaldığında veya kararsızlaştığında makine durmak zorunda kalmaz; güvenli nominal hızda klasik kural tabanlı sabit boy kesim moduna geri döner.

---

## 📊 UKM-OPT Makinesi: Üç Temel Başarısızlık ve Güvenli Davranış Tablosu

| Başarısızlık Türü | Olası Neden / Siber Tehdit | Tanımlanan Güvenli Davranış (Safe Behavior) | İlgili Emniyet Kavramları |
| :--- | :--- | :--- | :--- |
| **1. Model Yanlış Sınıflandırdı**<br/>*(Kusurlu/Kısa parçaya OK dedi veya hatalı koordinat üretti)* | Yansıma, optik sapma, eğitilmemiş ahşap budak türü veya model ağırlıklarının siber manipülasyonu. | **Bağımsız İkinci Kontrol & Sabit Sınırlar:**<br/>- Boyut doğrulaması için lazer mesafe sensörü ve enkoder karşılaştırması yapılır.<br/>- YZ'nin PLC'ye verebileceği hız ve pozisyon sınırları PLC içinde donanımsal **sabit limitlerle (Hard Limits)** kısıtlanır; YZ asla bu sınırı aşamaz.<br/>- Vardiyada rastgele seçilen parçalarda operatör numune kontrolü zorunludur. | `monitoring`, `human override`, `fail-safe`, `bounded inputs` |
| **2. Girdi Manipüle Edildi / Bozuldu**<br/>*(Kamera görüntüsü dondu, karardı veya sahte kare enjekte edildi)* | Kamera lensinin tozlanması/kirlenmesi, aydınlatma arızası, video hattına ortadaki adam (MitM) saldırısı veya RTSP/GigE Vision akışı sahteciliği. | **Girdi Bütünlüğü Doğrulaması:**<br/>- Kare sayacı (frame counter), zaman damgası ve canlılık kontrolü (liveness test) yapılır.<br/>- Donuk/tekrarlanan kare veya aşırı parlaklık sapmasında kesim derhal durdurulur, parça ıskartaya (RED) atılır, sesli alarm verilir ve operatör onayı istenir. | `monitoring`, `fail-safe`, `data integrity` |
| **3. Confidence (Güven Skoru) Düştü**<br/>*(Model kararsız kaldı, güven skoru <%95)* | Karmaşık budak yapısı, çatlak formasyonunun net seçilememesi veya optik netlik kaybı. | **Otomatik Iskartalama ve Fallback:**<br/>- Güven eşiği altındaki parça kesilmeden ayırma konveyörüne alınır ve operatör onayına bırakılır.<br/>- Art arda 3 parça düşük güvenle gelirse makine optimizasyon modundan çıkar, nominal güvenli hıza düşer ve operatör müdahalesine kadar **deterministik sabit boy kesim moduna (fallback)** geçer. | `confidence threshold`, `human override`, `fallback` |

---

## 📝 Föy Soruları ve Mühendislik Analizleri

### Soru 1: Kusurlu veya yanlış boyuttaki bir parçaya YZ tarafından "OK" verilirse operatör ve makine nasıl korunur?
* **Cevap:** YZ modelinin çıktısı tek başına son karar verici olamaz. Sistemde YZ'den tamamen bağımsız çalışan donanımsal bir lazer mesafe sensörü ve besleme enkoderi bulunur. Bu iki sensörün fiziksel ölçümü ile YZ koordinatı uyuşmazsa kesim döngüsü başlatılmaz. Ayrıca YZ'nin robota veya besleme servolarına gönderebileceği hız ve strok değerleri, PLC yazılımında kilitli **mutlak güvenlik sınırlarını (Hard Boundaries)** asla aşamaz.

### Soru 2: Kamera girdisi manipüle edilirse veya donarsa sistem bunu nasıl algılar?
* **Cevap:** Görüntü işleme IPC'si video akışındaki her karede ardışık kare sayacını (frame sequence ID) ve donanımsal zaman damgasını (hardware timestamp) doğrular. Statik/donmuş kare tespit edildiğinde, histogram parlaklık değerleri aniden düştüğünde veya lensin aşırı kirlendiği saptandığında emniyet protokolü tetiklenir; malzeme besleme durdurulur ve operatör ekranına "Optik Veri Bütünlüğü Bozuldu" alarmı verilir.

### Soru 3: Confidence skoru düştüğünde üretim tamamen durmalı mıdır?
* **Cevap:** Hayır, fabrikanın gereksiz yere durması önlenmelidir. Eşik altındaki tekil parçalar ıskartaya/manuel inceleme istasyonuna yönlendirilir. Ancak düşük güven durumu sistematik hale gelirse (örneğin arka arkaya 3 parçada güven <%95 ise), sistem akıllı optimizasyon modunu askıya alarak kural tabanlı sabit boy kesim moduna (**Graceful Degradation / Fallback**) geçer ve operatörü uyarır.

---

## 📌 Evidence Pack Çıktı Özeti
* **Belge No:** `EP 03 (Safety Functions)` & `EP 15 (AI Risk Management & Governance)`
* **Durum:** Doldurulmuş & Tasarıma İşlenmiş
* **Kritik Çıkarım:** Yapay zeka ve derin öğrenme üretim verimliliğini maksimize eder; ancak emniyeti asla yapay zekaya emanet edemeyiz. Emniyet, YZ'yi dışarıdan kuşatan bağımsız sensörler, deterministik limitler ve donanımsal koruyucularla (Safety PLC) garanti edilir.
