# Lab 2.1 · Lab 3: Threat & Risk + Risk Matrix — Şişeleme Hattı
**Çıktı:** Risk Register · **Evidence Pack:** EP 04 — Risk Assessment

Skor = L × S × E (her biri 1–3). **Kritik** 18–27 · **Yüksek** 9–17 · **Orta** 4–8 · **Düşük** 1–3. **Kural:** S=3 ise skor düşük olsa da senaryo öne alınır.

| # | Senaryo (kim → kapı → zincir → fiziksel sonuç) | L | S | E | Skor | Öncelik |
|---|---|:-:|:-:|:-:|:-:|---|
| 1 | Fırsatçı dış → internete doğrudan açık uzak servis erişimi (paylaşılan parola) → PLC/robot kontrolörüne ulaşır, bakım modunda hareket komutu veya hız parametresi değiştirir → hücrede sıkışan şişeyi alan operatörü robot/kapama kafası yaralar | 2 | 3 | 3 | 18 | **Kritik** |
| 2 | Hedefli → ofis e-postasına oltalama → düz ağ üzerinden mühendislik istasyonu → Safety PLC'de ışık perdesi muting'i genişletir → koruma devredeyken el kapama kafasına girer, ezilme/amputasyon | 1 | 3 | 2 | 6 | Orta → **S=3 kuralıyla öne alınır** |
| 3 | Meraklı içeriden → paylaşılan HMI hesabı → gazlı ürün reçetesinde dolum basıncını artırır → şişe patlaması, cam parçasıyla operatörde kesik | 2 | 2 | 1 | 4 | Orta |

1. **En yüksek öncelik:** Senaryo 1 — internetten herkesin denediği bir kapı, zayıf parola ve robot hücresinde insan zararı bir arada.
2. **S=3 olup skoru düşük kalan:** Senaryo 2 (skor 6). Register'da skora göre değil önce S'ye göre sıralanır, "S3 — insan zararı" etiketiyle kırmızı işaretlenir ve yönetim özetinde ayrı satır olarak gösterilir.
