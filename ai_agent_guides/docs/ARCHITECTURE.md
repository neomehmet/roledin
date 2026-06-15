# Mimari Açıklama

## Sistem Amacı

Bu proje, elektrik dağıtım/iletim şebekesine ait GIS saha verilerinden fiziksel ekipman ilişkilerini çıkarır. Amaç, “hangi ayırıcı/kesici hangi hücrede, hangi bara ve kablo bağlantısı ile ilişkili, hangi merkezden hangi merkeze giden hat üzerinde yer alıyor?” gibi sorguları hızlı ve tekrar kullanılabilir hale getirmektir.

## Katmanlar

### 1. GIS Ön İşleme Katmanı — `couplateV2.py`

Bu katman ham shapefile verilerini okur:

- `LGC_MERKEZ.shp`: merkez/substation poligonları.
- `T_HUCRE.shp`: hücre/bay poligonları.
- `T_OG_BARA.shp`: bara çizgileri.
- `H_ENERJI_NAKIL_HATTI.shp`: hat/kablo çizgileri.
- `T_OG_KESICI.shp`: kesici noktaları.
- `T_OG_AYIRICI.shp`: ayırıcı noktaları.
- `T_OG_KABLO_BAGLANTISI.shp`: kablo bağlantısı çizgi/nokta geometrileri.

Ana işlemler:

1. 34.5 kV filtreleme yapılır.
2. Topraklı ayırıcılar dışarıda bırakılır.
3. Koordinatlar `PRECISION = 8` ile normalize edilir.
4. Çizgisel verilerden `START_POINT` ve `END_POINT` çıkarılır.
5. `edge_df` ve `point_df` üretilir.
6. Spatial join ile ekipman ilişkileri kurulur.
7. Lookup dictionary grupları Excel dosyalarına yazılır.
8. Hücre içindeki ayırıcılar bağlantı elemanına uzaklığa göre `top` / `bottom` olarak sınıflandırılır.

### 2. Lookup Yükleme Katmanı — `import_dicts.py`

Bu katman, `couplateV2.py` tarafından üretilen Excel dosyalarını tekrar ağır spatial join yapmadan okur.

Ana sorumluluklar:

- `Key` / `Value` sheet yapısını dictionary’ye dönüştürmek.
- Excel’den gelen string liste/sözlük/tuple değerlerini Python nesnelerine dönüştürmek.
- WKT geometri stringlerini `shapely` nesnesine çevirmek.
- `edge_df`, `point_df`, `all_segments_df`, `bara_2_baglanti_df`, `ayirici_anahtar_status_df` gibi DataFrame’leri yüklemek.

### 3. Graph ve Trace Katmanı — `graph.py`

Bu katman lookup dictionary’leri yükler, `NetworkX` graph oluşturur ve ayırıcı/hücre üzerinden trace yapar.

Ana sorumluluklar:

- `Graph.init_dicts()` ile lookup verilerini belleğe almak.
- `Graph.make_graph()` ile `point_df` + `edge_df` kullanarak graph oluşturmak.
- Ayırıcı koordinatlarını graph node’u olarak eklemek.
- Başlangıç hücresinden trace başlatmak.
- Hat/kablo içeren path’leri filtrelemek.
- Alt merkeze, sonlanan hatta ve merkez içi geçişlere göre bağlantı tablosu üretmek.
- Sonuçları `*-connection_and_length.xlsx` ve `*-just_between_bays.xlsx` olarak dışa aktarmak.

## Modüller Arası Bağımlılık

```text
couplateV2.py
    └── Excel lookup ve DataFrame çıktıları üretir
            └── import_dicts.py bu çıktıları okur
                    └── graph.py import_dicts.load_and_unpack_all() ile veriyi kullanır
```

`graph.py`, doğrudan `couplateV2.py` import etmez. Bu doğru bir mimari karardır; ağır GIS ön işleme ile hızlı graph sorgusu ayrılmıştır.

## Fiziksel Model Kavramları

| Kavram | Kod adı | GIS tipi | Açıklama |
|---|---|---|---|
| Merkez | `merkez` | polygon | Substation / dağıtım merkezi. |
| Hücre | `hucre` | polygon | Bay / fider hücresi. |
| Bara | `bara` | line | Hücre içindeki bara iletkeni. |
| Hat/Kablo | `hat` / `HAT` | line | Merkezler arası bağlantı / kablo. |
| Kablo bağlantısı | `baglanti` | line/point | Hücreden hatta geçiş bağlantısı. |
| Ayırıcı | `ayirici` | point | İzolatör/disconnector. |
| Kesici | `kesici` | point | Circuit breaker. |

## Tasarım Kararları

### Spatial join ilişkileri Excel’e dökülüyor

Bunun nedeni, GIS/spatial join işlemlerinin maliyetli olmasıdır. Bir kez ilişkilendirilen veriler Excel lookup olarak saklanır ve sonraki çalışmalarda hızlıca dictionary’ye çevrilir.

### Graph node’ları koordinat tuple’larıdır

Graph yapısında node kimliği çoğunlukla `(x, y)` koordinat tuple’ıdır. Ayırıcı ID’leri, bu koordinatlar üzerinden graph node’una bağlanır.

### Hat mesafesi sadece `HAT` edge’lerinden hesaplanır

Merkez içi bara/bağlantı geçişleri genelde 0 mesafe veya lojik geçiş olarak tutulur. Fiziksel kablo uzunluğu için `edge_df.TYPE == "HAT"` filtrelenir.

## Kritik Mimari Riskler

1. `couplateV2.py` import edildiğinde tüm GIS işlemleri çalışır. Bu dosya ileride `main()` fonksiyonuna alınmalıdır.
2. `graph.py` içinde global değişkenler (`connection_and_length`, `fot_text`, `count`) vardır. Bunlar sınıf içine taşınmalıdır.
3. Excel, büyük veri için pratik ama yavaş olabilir. Veri büyürse Parquet/SQLite/GeoPackage düşünülmelidir.
4. Spatial join predicate seçimleri (`within`, `intersects`, `touches`) saha verisinin geometrik kalitesine çok bağlıdır.
5. Koordinat yuvarlama precision değeri değişirse tüm lookup ve graph eşleşmeleri etkilenir.
