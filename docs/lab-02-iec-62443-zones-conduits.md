# Laboratuvar 2: IEC 62443 Mini Workshop (Bölgeleme ve Geçiş Kanalları)
**Makine Adı:** UKM-OPT · Otomatik Boy Kesme Makinesi  
**Bootcamp Sürümü:** Safety 4.0 Bootcamp — UKM-OPT  
**Çıktı Belgesi:** Zone / Conduit Diagram  
**Evidence Pack Karşılığı:** `Evidence Pack EP 05 — Zone & Conduit`  
**İlgili Standartlar:** IEC 62443-3-2 (Risk Değerlendirmesi ve Bölgeleme), IEC 62443-3-3 (Sistem Güvenliği Gereksinimleri), IEC 62443-4-2 (Bileşen Güvenliği)

---

## 👨‍🏫 Mentor Notu: Zone & Conduit Mantığı
Geleneksel endüstriyel ağlarda en sık yapılan ölümcül hata **"Düz Ağ" (Flat Network)** tasarımıdır. Düz bir ağda ofis bilgisayarına gelen bir oltalama e-postası veya internete açık tek bir port, saldırganın doğrudan üretim PLC'sine veya Safety PLC'ye ulaşmasına izin verir.

IEC 62443 standardının özü şudur:
* **Zone (Bölge):** Benzer güvenlik kritikliği ve işletim hedefine sahip donanım ve yazılımların mantıksal/fiziksel olarak bir araya getirilmesi.
* **Conduit (Geçiş Kanalı):** İki farklı bölge arasındaki iletişimin aktığı tekil, filtrelenmiş, denetlenen ve kayıt altına alınan kapıdır.
* **Kural:** Her bölge sınırında bir conduit vardır. İki bölge birbirine doğrudan, kuralsız ve kontrolsüz bağlanamaz!

---

## 🏗️ Zone / Conduit Mimari Şeması

Aşağıdaki diyagramda UKM-OPT makinesinin IEC 62443 prensiplerine göre yapılandırılmış 5 ana bölgesi ve aralarındaki kontrollü geçiş kapıları (conduit) gösterilmiştir:

```mermaid
flowchart TB
    subgraph Z5["BÖLGE 5 (Z5): DIŞ AĞ & KURUMSAL (Enterprise / External Zone)"]
        INTERNET["Genel İnternet & Servis Sağlayıcı"]
        ERP["Ofis Ağı (ERP, E-Posta, Kurumsal IT)"]
        CLOUD["Bulut Telemetri Sunucuları"]
    end

    subgraph C54["Conduit 5-4: Dış Sınır Güvenlik Duvarı (Perimeter Firewall)"]
        direction LR
        FW1["Firewall: VPN Sonlandırma + MFA Zorunluluğu<br/><i>Doğrudan Port Yönlendirme ve Gelen Bağlantı YASAK</i>"]
    end

    subgraph Z4["BÖLGE 4 (Z4): BAKIM VE ERİŞİM DMZ'Sİ (Demilitarized Zone)"]
        JUMP["Jump Server / Bastion Host<br/><i>Kişisel Hesaplar + Tam Video/Komut Oturum Kaydı</i>"]
        PATCH["Yama & Güncelleme Dağıtım Sunucusu"]
    end

    subgraph C43["Conduit 4-3: DMZ - Denetim Güvenlik Duvarı (Internal Firewall)"]
        direction LR
        FW2["Firewall: Protokol İzin Listesi (Allow-List)<br/><i>Yalnızca Tanımlı Bakım Protokolleri (RDP/SSH/VNC Denetimli)</i>"]
    end

    subgraph Z3["BÖLGE 3 (Z3): OPTİMİZASYON & DENETİM (Supervisory & Optimization Zone)"]
        IPC_CAM["Görüntü İşleme & Kamera IPC'si<br/><i>Optik Boy Ölçüm Motoru</i>"]
        HMI["Operatör Paneli (HMI)"]
        ENG_ST["Mühendislik İstasyonu (Engineering Workstation)"]
        SCADA["SCADA İstemcisi"]
    end

    subgraph C32["Conduit 3-2: Denetim - Saha İletişim Filtresi"]
        direction LR
        FW3["Endüstriyel Filtre / VLAN Yönlendirici<br/><i>Yalnızca Üretim Verisi (OPC UA, Modbus TCP)</i>"]
    end

    subgraph Z2["BÖLGE 2 (Z2): MAKİNE KONTROL BÖLGESİ (Control Zone)"]
        PLC["Standart Üretim PLC'si"]
        DRIVES["Servo Sürücüler & İnvertörler"]
        MOTORS["Konveyör Motorları & Pnömatik Valfler"]
        
        PLC <--> DRIVES
        DRIVES --> MOTORS
    end

    subgraph C21["Conduit 2-1: Emniyet Sınır İzolasyonu (Safety Isolation Conduit)"]
        direction LR
        ISOLATION["Sıkı Filtrelenmiş / Fiziksel İzolasyon<br/><i>YALNIZCA Durum Okuma (Status Read)<br/>UZAKTAN YAZMA / PROGRAMLAMA KESİNLİKLE YASAK!</i>"]
    end

    subgraph Z1["BÖLGE 1 (Z1): FONKSİYONEL EMNİYET BÖLGESİ (Safety Zone - SL 4)"]
        SPLC["Safety PLC (Çift Kanallı Emniyet Mantığı)"]
        LIGHT["Işık Perdeleri & Kapak Switchleri"]
        ESTOP["Acil Stop Butonları"]
        STO_ACT["STO Modülü & Emniyet Kontaktörleri"]
        
        LIGHT --> SPLC
        ESTOP --> SPLC
        SPLC --> STO_ACT
    end

    %% Bağlantılar
    INTERNET ==> C54
    ERP ==> C54
    C54 ==> Z4
    Z4 ==> C43
    C43 ==> Z3
    Z3 ==> C32
    C32 ==> Z2
    Z2 <==> C21
    C21 <==> Z1

    classDef z5 fill:#eceff1,stroke:#607d8b,stroke-width:2px;
    classDef z4 fill:#fff8e1,stroke:#ffa000,stroke-width:2px;
    classDef z3 fill:#e1f5fe,stroke:#0288d1,stroke-width:2px;
    classDef z2 fill:#e8f5e9,stroke:#388e3c,stroke-width:2px;
    classDef z1 fill:#ffebee,stroke:#d32f2f,stroke-width:3px;
    classDef conduit fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px,stroke-dasharray: 4 4;

    class Z5 z5;
    class Z4 z4;
    class Z3 z3;
    class Z2 z2;
    class Z1 z1;
    class C54,C43,C32,C21 conduit;
```

---

## 📝 Görev A — Varlıkların Bölgelere Ayrılması ve Analizi

### Varlık Listesi:
`PLC` · `HMI` · `Robot Kontrolörü / Sürücüler` · `Safety PLC` · `SCADA İstemcisi` · `Mühendislik İstasyonu` · `Uzak Servis Erişimi` · `Ofis Ağı (ERP/E-posta)` · `Optik Kamera ve IPC`

### 1. Hangi varlıklar aynı bölgede? Bölgeleri adlandırıp gruplandırın.
* **Bölge 1 (Z1 - Emniyet Bölgesi):** Safety PLC, optik ışık perdeleri, acil stop butonları, koruyucu kapak interlockları ve STO kontaktörleri.
* **Bölge 2 (Z2 - Makine Kontrol Bölgesi):** Standart üretim PLC'si, servo sürücüler, invertörler ve besleme aktüatörleri.
* **Bölge 3 (Z3 - Optimizasyon ve Denetim Bölgesi):** Kamera ve optik görüntü işleme ünitesi (IPC), HMI operatör dokunmatik paneli, SCADA istemcisi ve mühendislik iş istasyonu.
* **Bölge 4 (Z4 - Bakım & Erişim DMZ'si):** Jump Server (Bastion Host), oturum kayıt sunucusu ve yerel güncelleme havuzu.
* **Bölge 5 (Z5 - Dış Ağ & Kurumsal Bölge):** Kurumsal ofis ağı (ERP, MES, kurumsal e-posta), internet omurgası, bulut telemetrisi ve harici uzak servis erişim kanalları.

### 2. Safety PLC'yi hangi bölgeye koydunuz — neden?
* **Cevap:** **Z1 Emniyet Bölgesine (Safety Zone)** yerleştirildi. Çünkü Safety PLC insan canını doğrudan koruyan en kritik varlıktır. Tesisin üretim kontrol ağında veya gözetim ağında bir siber ihlal meydana gelse dahi, bu ihlalin emniyet sistemine sıçramasını engellemek için Safety PLC bağımsız ve en yüksek güvenlik seviyesine (Security Level 4) sahip izole bir bölgede konumlandırılmalıdır. Uzak bakım modülü ise kesinlikle emniyet bölgesinde veya üretim kontrol ağında değil; aradaki **Z4 Bakım DMZ'sinde** bulunmalı, doğrudan internete veya emniyet ağına temas etmemelidir.

### 3. Bölgeler arası geçişler (conduit) nerede? Her geçişte ne kontrol edilecek?
* **Cevap:**
  1. **Z5 (Dış Ağ) $\leftrightarrow$ Z4 (DMZ):** IPSec/SSL VPN sonlandırma, kurumsal kimlik denetimi ve çok faktörlü doğrulama (MFA) kontrol edilir. Doğrudan port yönlendirme engellenir.
  2. **Z4 (DMZ) $\leftrightarrow$ Z3 (Denetim):** Yalnızca tanımlı jump server üzerinden yetkilendirilmiş protokollerle (RDP/SSH/VNC) geçişe izin verilir; izin listesi (allow-list) dışındaki tüm trafik filtrelenir ve oturum tam kayıt altına alınır.
  3. **Z3 (Denetim) $\leftrightarrow$ Z2 (Kontrol):** Yalnızca operasyonel endüstriyel protokoller (OPC UA, Modbus TCP, PROFINET) denetlenir; rastgele internet veya ofis protokolleri geçemez.
  4. **Z2 (Kontrol) $\leftrightarrow$ Z1 (Emniyet):** Yalnızca emniyet durum okuma (Safe State, hata kodları vb. salt-okunur veriler) sinyallerine izin verilir. Safety PLC'ye uzaktan program yükleme, firmware güncelleme veya mantık parametresi yazma yolu conduit üzerinde **tamamen kilitlidir**. Bu işlemler yalnızca sahada, fiziksel mühendislik anahtarı ile yapılır.

---

## 🔒 Görev B — Uzak Erişim Karar Tablosu (6 Satır)

Aşağıdaki kararlar, UKM-OPT makinesinde yetkisiz müdahaleleri önlemek ve siber dayanıklılık mevzuatlarını (CRA & NIS2) karşılamak için seçilmiştir:

| Karar Başlığı | Mevcut Seçenekler | Seçimimiz | Teknik ve Emniyet Gerekçesi |
| :--- | :--- | :--- | :--- |
| **1. Aracı?** | Doğrudan / Jump server | **Jump server** | Makine ve kontrolörler internete asla doğrudan açılmaz. Tek ve merkezi olarak izlenen, güvenlik duvarı arkasında bir ara nokta (bastion host) kullanılır. |
| **2. Tünel?** | Açık / VPN | **VPN** | Dışarıdan gelen tüm veri trafiği güçlü şifreli tünellerden (IPSec/WireGuard/TLS) geçer. Açık port taraması yapan otomatik internet botları makineyi keşfedemez. |
| **3. Kimlik?** | Parola / MFA | **MFA (Çok Faktörlü Doğrulama)** | Tek başına statik parola yeterli değildir; çalınan, sızan veya tahmin edilen şifrelerin yetkisiz girişe yol açması engellenir. |
| **4. Hesap?** | Paylaşılan / Kişisel | **Kişisel Hesap** | Ortak kullanılan genel hesaplar (`admin/admin`, `service`) yasaklanır. Her teknisyenin kendi tekil hesabı olur; kimin, ne zaman, hangi parametreyi değiştirdiği izlenebilir. |
| **5. Kayıt?** | Yok / Tam oturum kaydı | **Tam oturum kaydı** | Uzak bağlantı esnasında yapılan tüm terminal komutları, fare tıklamaları ve ekran oturumu adli bilişim (forensics) ve kaza sonrası kök neden analizi için arşivlenir. |
| **6. Zamanlama?** | Sürekli açık / Varsayılan kapalı | **Varsayılan kapalı** | 7/24 internete açık kapı sürekli bir saldırı yüzeyidir. Erişim yalnızca arıza/bakım anında talep edilir; yetkili onayıyla belirli bir süre (ör. 2 saat) için açılır ve süre bitince otomatik kapanır. |

---

## 📌 Evidence Pack Çıktı Özeti
* **Belge No:** EP 05
* **Durum:** Doldurulmuş & Doğrulamaya Hazır
* **Kritik Çıkarım:** Güvenli uzaktan erişim mimarisi, kontrol sistemlerini doğrudan internete açmak yerine DMZ ve Jump Server üzerinden kademelendirilerek kurgulanmalıdır. Emniyet bölgesi (Z1), diğer tüm ağlardan en sıkı izolasyon kurallarıyla ayrılmalıdır.
