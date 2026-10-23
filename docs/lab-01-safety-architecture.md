# Laboratuvar 1: Safety Architecture (Emniyet Mimarisi)
**Makine Adı:** UKM-OPT · Otomatik Boy Kesme Makinesi  
**Bootcamp Sürümü:** Safety 4.0 Bootcamp — UKM-OPT  
**Çıktı Belgesi:** Safety Function Map  
**Evidence Pack Karşılığı:** `Evidence Pack EP 03 — Safety Functions`  
**İlgili Standartlar & Mevzuat:** ISO 13849-1, IEC 62061, IEC 61508, AB Makine Tüzüğü (2023/1230)

---

## 👨‍🏫 Mentor Notu: Temel Mühendislik Yaklaşımı
Endüstriyel otomasyonda altın kural şudur: **Emniyet (Safety) kazaya karşı, Güvenlik (Security) ise kasta karşı korur.**  
Bir siber güvenlik açığı, makinenin kontrol mantığını değiştirip emniyet fonksiyonunu etkisiz kılarsa, bu artık basit bir bilişim zafiyeti değil; **doğrudan insan hayatını tehdit eden bir emniyet bulgusudur.**

Bu laboratuvarda yaptığımız temel mimari ayrım:
* **Normal Kontrol Yolu (Üretim):** Sensörler $\rightarrow$ Standart PLC $\rightarrow$ Servo Sürücüler $\rightarrow$ Aktüatörler.
* **Safety Yolu (Koruma - Donanımsal Olarak AYRI):** E-Stop / Işık Perdesi $\rightarrow$ Safety PLC $\rightarrow$ STO / Kontaktör $\rightarrow$ Safe State (Güvenli Durum).
* **Dijital Kesişim Noktası ("Sarı Çizgi"):** IPC, uzak bakım veya mühendislik istasyonunun Safety PLC konfigürasyonuna veya sürücü parametrelerine dijital olarak temas ettiği riskli hat.

---

## 🗺️ Safety Function Map (Emniyet Fonksiyonu Haritası)

Aşağıdaki mimari şema, UKM-OPT makinesinde üretim akışı ile bağımsız emniyet zincirinin ayrımını ve siber-fiziksel risk noktasını göstermektedir:

```mermaid
flowchart TB
    subgraph EXTERNAL["Dış Dünya & Uzak Erişim"]
        REMOTE["Uzak Servis / Bakım"]
        ENG["Mühendislik İstasyonu"]
    end

    subgraph NORMAL_PATH["Normal Kontrol Yolu (Üretim Hattı)"]
        direction TB
        CAM["Kamera & Görüntü İşleme (IPC)<br/><i>Optik Boy Ölçümü & Optimizasyon</i>"]
        SENS_NORM["Konveyör / Pozisyon Sensörleri"]
        PLC_NORM["Standart Üretim PLC'si<br/><i>Kesim Sıralaması ve Hareket Yönetimi</i>"]
        DRIVE["Servo Sürücüler & İnvertörler"]
        ACT_NORM["Besleme Konveyörü & Malzeme İtici"]
        
        CAM -->|Boy & Kesim Koordinatları| PLC_NORM
        SENS_NORM -->|Malzeme Geldi/Geçti| PLC_NORM
        PLC_NORM -->|Hız ve Pozisyon Komutları| DRIVE
        DRIVE -->|Tahrik Gücü| ACT_NORM
    end

    subgraph YELLOW_LINE["Dijital Kesişim Noktası ('Sarı Çizgi')"]
        REMOTE -.->|Parametre / Firmware Değişikliği| PLC_NORM
        ENG -.->|Yapılandırma & Kalibrasyon| CAM
        ENG -.->|Güvensiz Erişim / Bypass Riski| SPLC
    end

    subgraph SAFETY_PATH["Safety Yolu (Fonksiyonel Emniyet · Donanımsal Olarak AYRI)"]
        direction TB
        ESTOP["Acil Stop (E-Stop) Butonları"]
        LIGHT_CURTAIN["Optik Işık Perdesi (Koruyucu Bariyer)"]
        INTERLOCK["Emniyet Switchleri & Kapak Kilitleri"]
        
        SPLC["Safety PLC (Çift Kanallı, PLr/SIL Sertifikalı)<br/><i>Checksum Korumalı Mantık Denetimi</i>"]
        
        STO["STO (Safe Torque Off) & Emniyet Kontaktörleri"]
        SAFE_STATE(["GÜVENLİ DURUM (Safe State)<br/>- Kesici bıçak enerjisi kesilmiş<br/>- Besleme tahriki durdurulmuş<br/>- Beklenmedik yeniden başlama engellenmiş"])
        
        ESTOP -->|Çift Kanal Emniyet Sinyali| SPLC
        LIGHT_CURTAIN -->|İhlal Tespiti (OSSD Çıkışı)| SPLC
        INTERLOCK -->|Kapak Açıldı Sinyali| SPLC
        
        SPLC -->|Emniyet Açma / STO Tetikleme| STO
        STO -->|Fiziksel Güç Kesintisi| SAFE_STATE
    end

    classDef normal fill:#e1f5fe,stroke:#0288d1,stroke-width:2px,color:#01579b;
    classDef safety fill:#ffebee,stroke:#d32f2f,stroke-width:2px,color:#b71c1c;
    classDef yellow fill:#fff9c4,stroke:#fbc02d,stroke-width:2px,stroke-dasharray: 5 5,color:#f57f17;
    classDef safeState fill:#c8e6c9,stroke:#388e3c,stroke-width:3px,color:#1b5e20;

    class CAM,SENS_NORM,PLC_NORM,DRIVE,ACT_NORM normal;
    class ESTOP,LIGHT_CURTAIN,INTERLOCK,SPLC,STO safety;
    class YELLOW_LINE yellow;
    class SAFE_STATE safeState;
```

---

## 📝 Föy Soruları ve Analizler (Doldurulmuş Çıktı)

### Soru 1: Bu makinede tehlike nedir? İnsan neyden zarar görür?
* **Cevap:** UKM-OPT makinesinde kamera yanlış kesim koordinatı üretirse veya malzeme kılavuz yatağında kayarsa operatör sıkışan veya hatalı konumlanan parçayı düzeltmek için tehlikeli kesici bölgeye yaklaşabilir. Bu esnada kesici giyotin/dairesel testere bıçağı ve yüksek torklu malzeme besleme mekanizması operatörün el veya uzvunu kaparak ağır yaralanmaya veya amputasyona (uzuv kaybına) sebep olabilir.

### Soru 2: Safety function (emniyet fonksiyonu) hangisidir?
* **Cevap:** Koruyucu erişiminde (ışık perdesi ihlali veya koruyucu kapak/interlock açılması durumunda) tehlikeli kesim ve besleme hareketlerini Safe Torque Off (STO) ve emniyet kontaktörleri vasıtasıyla derhal durdurmak; enerji kesildikten sonra makinenin otomatik ve beklenmedik bir şekilde yeniden başlamasını engellemek.

### Soru 3: Safety-related control system nerede? Şemada işaretleyin.
* **Cevap:** Koruyucu algılayıcılar (ışık perdesi, kapak kilit switchleri, E-Stop) $\rightarrow$ emniyet mantığı (Safety PLC) $\rightarrow$ kesici ve besleme hareketini güvenli duruma getiren son kontrol elemanları (STO / Emniyet Kontaktörleri). Üst yolda bulunan kamera, IPC, normal üretim PLC'si ve HMI bu emniyet zincirinin donanımsal bir parçası değildir.

### Soru 4: Normal PLC ile Safety PLC farkı nedir?
* **Cevap:** 
  * **Normal PLC:** Üretim hızını, kesim optimizasyonunu ve reçeteleri yönetir. Tek kanallıdır (single channel); yazılımsal veya donanımsal bir arıza meydana geldiğinde bu hata sessiz kalabilir (silent failure) ve tehlikeli hareketi sürdürebilir.
  * **Safety PLC:** Çift kanallı (1oo2D / 2oo3 mimarisi), sürekli donanımsal self-test (öz-tanılama) yapan, saat frekansları ve sinyalleri çapraz doğrulanan sertifikalı bir cihazdır. En ufak bir uyumsuzlukta sistemi fail-safe (güvenli durum) konumuna geçirir. Yazılımı kriptografik imza (checksum) ile yetkisiz değişikliklere karşı kilitlenmiştir.

### Soru 5: PLr / SIL neden var? Bu fonksiyon için neden önemli?
* **Cevap:** Kesici ve hızlı tahrik kaynaklı ağır ve geri dönüşsüz yaralanma riskine karşı (ölüm, uzuv kaybı) gereken emniyet güvenilirlik performansını (Required Performance Level - PLr veya Safety Integrity Level - SIL) matematiksel olarak belirlemek için vardır. Kesici bölgeye operatörün sık erişimi ve yüksek tehlike seviyesi nedeniyle ISO 13849-1 standardı uyarınca tipik olarak en az **PL d (veya PL e)** / **SIL 2 - SIL 3** güvenilirlik düzeyi gereklidir.

### Soru 6: Safe state (güvenli durum) nedir?
* **Cevap:** Kesici bıçak tahrikinin ve tehlikeli besleme hareketlerinin enerjisinin fiziksel olarak kesildiği (STO aktif), dönen veya hareket eden kütlelerin durdurulduğu, hidrolik/pnömatik basınçların tahliye edildiği ve yetkili kişi tarafından manuel bilinçli bir "Reset" komutu verilmedikçe makinenin kendi kendine asla yeniden başlayamayacağı kilitli durumdur.

### Soru 7: Safety function hangi dijital bağımlılıklara sahip?
* **Cevap:** 
  1. Optik kamera ölçümü ve görüntü işleme algoritmaları,
  2. Servo motor pozisyon/enkoder geri besleme verileri,
  3. PLC hareket denetim parametreleri ve hız profilleri,
  4. Sürücü firmware ve parametre setleri,
  5. Safety PLC emniyet mantık yazılımı, ışık perdesi blanking/muting yapılandırması ve checksum değeri,
  6. ERP/MES üzerinden gelen iş emri ve kesim boyu verisi.

### Soru 8: Siber müdahale hangi noktadan bu fonksiyonu bozar?
* **Cevap:** Kesim koordinatını, hareket parametrelerini veya emniyet yapılandırmasını değiştirebilen yetkisiz erişim yolundan bozar. Özellikle uzaktan bakım modülü veya IPC üzerinden aynı endüstriyel ağa bağlı mühendislik istasyonuna erişilirse; ışık perdesi muting süreleri uzatılabilir, kapak interlockları yazılımsal olarak bypass edilebilir veya Safety PLC konfigürasyonu manipüle edilerek kağıt üzerinde var görünen emniyet fonksiyonu sahada tamamen işlevsiz hale getirilebilir.

---

## 📌 Evidence Pack Çıktı Özeti
* **Belge No:** EP 03
* **Durum:** Doldurulmuş & Taslaklandı (Doğrulama aşamasında)
* **Kritik Çıkarım:** Üretim hızı optimizasyonu amacıyla kamera veya IPC üzerinden emniyet yoluna yazılımsal bağlantı ("sarı çizgi") kurulmamalıdır. Emniyet zinciri donanımsal ve mantıksal olarak izole edilmelidir.
