# Elektrik Şebekesi GIS–Graph Projesi Rehberi

Bu depo, elektrik iletim/dağıtım şebekesine ait GIS saha verilerini okuyup fiziksel ekipman ilişkilerini çıkarır, bu ilişkileri Excel + Python dictionary formatında saklar ve daha sonra bu verilerden tek hat diyagramı mantığında bir `NetworkX` graph yapısı oluşturur.

Bu rehber hem insan geliştiriciler hem de yapay zekâ kodlama ajanları için yazılmıştır. Amaç, kod üzerinde geliştirme yapılırken tüm dosyaların baştan sona okunmasını azaltmak, doğru dosyaya hızlı yönlendirmek ve mimari kararları kaybetmemektir.

## Kısa Özet

Proje 3 ana Python dosyasından oluşur:

| Dosya | Ana görev | Ne zaman okunmalı? |
|---|---|---|
| `couplateV2.py` | GIS shapefile verilerini okur, spatial join işlemleriyle ekipman ilişkilerini kurar, lookup Excel dosyalarını üretir. | GIS veri kaynağı, spatial join, dictionary üretimi veya Excel çıktı şeması değişecekse. |
| `import_dicts.py` | `couplateV2.py` tarafından üretilmiş Excel lookup dosyalarını okuyup Python dictionary/DataFrame nesnelerine dönüştürür. | Excel lookup okuma, tip dönüşümü, hızlı sorgu veya veri yükleme iyileştirilecekse. |
| `graph.py` | Lookup dictionary’leri kullanarak NetworkX graph oluşturur ve hücre/ayırıcı üzerinden trace algoritmasını çalıştırır. | Trace algoritması, graph performansı, mesafe hesabı, merkez içi tek bara / 2.5 bara akışı değişecekse. |

## Ana Veri Akışı

```text
GIS shapefile verileri
    │
    ▼
couplateV2.py
    - 34.5 kV filtreleme
    - spatial join ilişkileri
    - top/bottom ayırıcı sınıflandırması
    - node/edge tablo üretimi
    │
    ▼
Excel lookup dosyaları
    - *_lookup_dicts.xlsx
    - edge_df.xlsx
    - point_df.xlsx
    - all_segment.xlsx
    - bara_2_baglanti.xlsx
    - ayirici_anahtar_status.xlsx
    │
    ▼
import_dicts.py
    - Excel -> dict/DataFrame
    - string/list/tuple/WKT dönüşümleri
    │
    ▼
graph.py
    - NetworkX graph
    - feeder / bay / substation trace
    - connection_and_length çıktıları
```

## GIS Veri Tipleri

Projede 3 tip fiziksel GIS verisi vardır:

1. **Noktasal veriler:** ayırıcı, kesici.
2. **Çizgisel / 2 boyutlu veriler:** tel, kablo, hat, bara, kablo bağlantısı.
3. **Poligon / alan verileri:** hücre ve merkez. Kullanıcı notunda “3 boyutlu polygon” olarak ifade edilmiştir; kodda `shapely`/`geopandas` geometri alanı olarak işlenmektedir.

## Çalıştırma Sırası

GIS dosyaları veya spatial ilişki mantığı değiştiyse:

```bash
python couplateV2.py
```

Bu komut lookup Excel dosyalarını üretir. Daha sonra graph/trace çalıştırmak için:

```bash
python graph.py
```

Programatik kullanım:

```python
from graph import Graph

g = Graph()
g.init_dicts()      # Excel lookup dosyalarını import_dicts üzerinden yükler
g.make_graph()      # NetworkX graph oluşturur
g.search_and_create_tree(start_hucre_id=304374)
```

## Geliştirici İçin Hızlı Karar Tablosu

| Görev | İlk bakılacak rehber | İlk bakılacak kod |
|---|---|---|
| GIS ilişkilendirme değişikliği | `docs/DATA_CONTRACTS.md`, `docs/ARCHITECTURE.md` | `couplateV2.py` |
| Excel lookup okuma hatası | `docs/DATA_CONTRACTS.md` | `import_dicts.py` |
| Trace yanlış sonuç veriyor | `docs/GRAPH_TRACE_ALGORITHM.md` | `graph.py` |
| Performans iyileştirme | `docs/REFACTORING_ROADMAP.md` | ilgili modül |
| Kod ajanı ile geliştirme | `AGENTS.md` | sadece görevle ilgili dosya |

## Önemli Mimari Kural

`couplateV2.py` ağır GIS/spatial join ön işleme katmanıdır. Her çalıştırmada yeniden çalıştırılması gerekmez. GIS verisi değişmediği sürece ajanlar ve geliştiriciler doğrudan `import_dicts.py` + `graph.py` tarafında çalışmalıdır.
