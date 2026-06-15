# Kod Geliştirme İstek Şablonu

Ajanlara görev verirken aşağıdaki formatı kullanmak token tüketimini ve yanlış dosya okuma ihtiyacını azaltır.

## Şablon

```text
Görev tipi: [GIS ilişkilendirme / lookup okuma / graph trace / performans / refactor / hata düzeltme]
İlgili dosya: [couplateV2.py / import_dicts.py / graph.py / emin değilim]
Beklenen davranış:
Mevcut hata veya problem:
Örnek input:
Örnek output:
Korunması gereken davranışlar:
Çıktı formatı: [kod bloğu / patch / açıklama / test]
```

## Örnek 1 — Trace Hatası

```text
Görev tipi: graph trace hata düzeltme
İlgili dosya: graph.py
Problem: 2.5 bara merkezde bağlama hücresi üzerinden çıkış ayırıcıları bazen yanlış eşleşiyor.
Beklenen davranış: Giriş barasından kuplaj hücresine, kuplajdan farklı baraya ve çıkış hücresindeki aynı bara numaralı top ayırıcıya geçilmeli.
Korunması gereken davranışlar: Tek bara merkez trace mantığı bozulmamalı. Output kolonları aynı kalmalı.
Çıktı formatı: değiştirilecek fonksiyonların tamamını ver ve test öner.
```

## Örnek 2 — Performans

```text
Görev tipi: performans
İlgili dosya: graph.py
Problem: search_and_create_tree büyük merkezlerde yavaş çalışıyor.
Beklenen davranış: Aynı sonuçları üretmeli, Excel çıktı kolonları değişmemeli.
Korunması gereken davranışlar: path_ids sadece HAT edge ID'leri içermeli.
Çıktı formatı: minimal patch ver, riskleri açıkla.
```

## Örnek 3 — GIS Şema Değişikliği

```text
Görev tipi: GIS ilişkilendirme
İlgili dosya: couplateV2.py
Problem: Yeni shapefile’da GERILIM kolonu bazı katmanlarda GERILIM_SEV olarak geliyor.
Beklenen davranış: Eski ve yeni kolon adları desteklensin.
Korunması gereken davranışlar: 34.5 kV filtresi değişmemeli.
Çıktı formatı: fonksiyonlaştırılmış çözüm öner.
```
