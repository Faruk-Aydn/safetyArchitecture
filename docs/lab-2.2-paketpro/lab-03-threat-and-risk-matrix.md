# Lab 2.2 · Lab 3: Threat & Risk + Risk Matrix — PaketPro 3000
**Çıktı:** Risk Register · **Evidence Pack:** EP 04

Skor = L × S × E. Kritik 18–27 · Yüksek 9–17 · Orta 4–8 · Düşük 1–3. **S=3 ise skor düşük olsa da öne alınır.**

| # | Senaryo (kim → kapı → zincir → fiziksel sonuç) | L | S | E | Skor | Öncelik |
|---|---|:-:|:-:|:-:|:-:|---|
| 1 | Fırsatçı dış → internete açık uzak bakım modülü + admin/admin → PLC ağına ve Safety PLC'ye ulaşır, ışık perdesi muting/çözünürlük parametresini değiştirir → film sıkışmasını gideren operatörün eli bıçağa/çeneye girer, amputasyon | 3 | 3 | 3 | 27 | **Kritik** |
| 2 | Meraklı içeriden → şifresiz port 80 HMI web arayüzü → çene sıcaklığı ve çevrim hızı reçetesini değiştirir → aşırı ısınan çene filmi yakar, duman/yangın başlangıcı ve yanık | 3 | 2 | 2 | 12 | Yüksek |
| 3 | Tedarik zinciri → servis dizüstüsünden kilitsiz USB ile zararlı yazılım → programlama aracı üzerinden kapak kilidi gecikmesini kısaltır → bıçak durmadan kapak açılır, kesik | 1 | 3 | 1 | 3 | Düşük → **S=3 kuralıyla öne alınır** |

1. **En yüksek öncelik:** Senaryo 1 (27) — internetten herkesin bulabildiği, varsayılan parolalı bir kapı doğrudan amputasyon riskine bağlanıyor.
2. **S=3 olup skoru düşük kalan:** Senaryo 3 (skor 3). "Düşük" sütununda kaybolmaması için register'da S'ye göre üstte, "S3 — insan zararı" etiketiyle tutulur; USB port kilidi ve servis dizüstüsü kontrolü 90 günlük plana yazılır.
