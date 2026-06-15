# Modernizasyon ve Refactoring Yol Haritası

Bu dosya, kodun daha yüksek performanslı, daha hatasız ve modern standartlarda çalışması için önerileri önceliklendirir.

## Öncelik 0 — Davranışı Bozmadan Güvenlik

Bu adımlar en düşük riskli iyileştirmelerdir.

### 1. `couplateV2.py` import-time side effect kaldırılmalı

Şu anda dosya import edildiğinde GIS okuma ve Excel üretme işlemleri çalışır. Bu risklidir.

Öneri:

```python
def main():
    # mevcut işlem akışı
    ...

if __name__ == "__main__":
    main()
```

Daha iyi yapı:

- `load_gis_layers(paths)`
- `build_edge_tables(layers)`
- `run_spatial_joins(layers)`
- `build_lookup_dicts(processed)`
- `export_lookup_dicts(groups, output_dir)`

### 2. `graph.py` global state temizlenmeli

Mevcut global değişkenler:

- `fot_text`
- `count`
- `connection_and_length`

Bunlar `Graph` instance alanı olmalıdır:

```python
self.connection_and_length = []
self.debug_text_buffer = []
self.debug_count = 0
```

### 3. Queue performansı iyileştirilmeli

Mevcut kullanım:

```python
queue = [source_node_coord]
ayirici_coord = queue.pop(0)
```

Öneri:

```python
from collections import deque
queue = deque([source_node_coord])
ayirici_coord = queue.popleft()
```

Bu değişiklik büyük trace işlemlerinde ciddi performans kazancı sağlar.

### 4. `path_ids` veri tipi standartlaştırılmalı

Şu anda bazı yerlerde liste, bazı yerlerde string liste kullanılıyor.

Öneri:

- İç işlem boyunca `list[int]` kalsın.
- Sadece Excel export aşamasında string’e çevrilsin.

### 5. Kontrollü edge lookup eklenmeli

Mevcut kullanım doğrudan KeyError riski taşır:

```python
start_end_2_edge_id[(path[i], path[i + 1])]
```

Öneri:

```python
edge_id = start_end_2_edge_id.get((path[i], path[i + 1]))
if edge_id is None:
    # hata logla ve path'i geçersiz say
```

## Öncelik 1 — Veri Kalitesi ve Test Edilebilirlik

### 1. Şema validasyonu eklenmeli

Her shapefile için gerekli kolonlar kontrol edilmeli:

- `ABB_INT_ID`
- `GERILIM`
- `geometry`
- dosyaya özel kolonlar: `ALTTIP`, `NORMAL_ANA`, `UZUNLUK`, `BARA_NO`

Minimum basit validasyon:

```python
def require_columns(df, required, layer_name):
    missing = set(required) - set(df.columns)
    if missing:
        raise ValueError(f"{layer_name} eksik kolonlar: {missing}")
```

### 2. Spatial join sonuçları ölçülmeli

Her join sonrası şu metrikler loglanmalı:

- toplam kayıt sayısı
- eşleşmeyen kayıt sayısı
- duplicate eşleşme sayısı
- beklenmeyen one-to-many sayısı

### 3. Regression test verisi oluşturulmalı

Küçük bir yapay GIS/Excel örneği ile şu testler yazılmalı:

- `safe_literal_eval()` dönüşümleri.
- `load_and_unpack_all()` minimum Excel seti.
- `make_graph()` node/edge sayıları.
- `get_path_ids()` iki yönlü edge mapping.
- `trace_merkez_ici_1_bara()` ve `trace_merkez_ici_2_5_bara()` küçük graph üzerinde.

## Öncelik 2 — Performans

### 1. Excel darboğazı azaltılmalı

Excel insan tarafından incelenebilir olduğu için faydalıdır, ama büyük veride yavaştır.

Alternatifler:

- Parquet: hızlı kolon bazlı okuma/yazma.
- Feather: hızlı Python içi veri aktarımı.
- SQLite: lookup sorguları ve indeksleme.
- GeoPackage: GIS geometrileri için daha doğal saklama.

Önerilen geçiş:

1. Excel çıktıları korunur.
2. Yanına `cache/lookup_store.parquet` veya `cache/lookup_store.sqlite` eklenir.
3. `import_dicts.py` önce hızlı cache’i, yoksa Excel’i okur.

### 2. Shortest path tekrarları azaltılmalı

`nx.shortest_path` ve `nx.single_source_shortest_path` birçok kez çalışabilir.

İyileştirmeler:

- Aynı kaynak node için path sonucunu cache’le.
- Merkez içi trace için alt graph oluştur.
- `cutoff` değerini parametre yap.
- `HAT` içeren path kontrolünü edge-level adjacency ile hızlandır.

### 3. DataFrame içinde string parse azaltılmalı

`ast.literal_eval()` pahalıdır. İç veride listeleri liste olarak tutmak daha hızlıdır.

## Öncelik 3 — Mimari Temizlik

### 1. Config yapısı

Sabit path ve ayarları ayrı config’e taşı:

```python
@dataclass
class ProjectPaths:
    gis_dir: Path
    output_dir: Path
    merkez_path: Path
    hat_path: Path
    ...
```

### 2. LookupStore sınıfı

`dicts["..."]` şeklindeki serbest erişim yerine typed yapı:

```python
@dataclass
class LookupStore:
    ayirici_2_hucre: dict[int, int]
    ayirici_id_2_coord: dict[int, tuple[float, float]]
    edge_df: pd.DataFrame
    point_df: pd.DataFrame
```

### 3. Logging standardı

`print()` yerine:

```python
import logging
logger = logging.getLogger(__name__)
```

Log seviyeleri:

- `INFO`: normal adımlar.
- `WARNING`: eksik ama tolere edilebilir veri.
- `ERROR`: işlem sonucunu bozabilecek veri.
- `DEBUG`: path/trace detayları.

## Öncelik 4 — Algoritmik Sağlamlık

### 1. CRS ve buffer güvenliği

`_AYIRICI_BARA_TOL = 1e-6` derece cinsinden kullanılıyor. Eğer veri coğrafi koordinat sistemindeyse bu yaklaşık bir toleranstır ama her bölgede aynı metre karşılığına gelmeyebilir.

Daha doğru yaklaşım:

1. Verinin CRS bilgisini kontrol et.
2. Projected CRS’e dönüştür.
3. Metre cinsinden buffer uygula.
4. Sonuçları gerekirse orijinal CRS’e döndür.

### 2. BARA_NO normalize mantığı

Mevcut mantıkta `str(BARA_NO)[0]` kullanılır. Eğer bara numaraları `10`, `11` gibi çok haneli olursa hatalı eşleşme oluşabilir.

Öneri:

- Regex ile gerçek bara grup numarasını çıkar.
- Veri sözlüğü varsa ona göre normalize et.
- Test ekle.

### 3. Trace döngüleri için güvenli limit

Visited set var, ancak queue büyümesi ve tekrar eden merkez içi geçişler için ek koruma iyi olur:

- maksimum queue boyutu
- maksimum trace derinliği
- aynı `(from_ayirici, to_ayirici)` bağlantısını tekrar eklememe

## Tavsiye Edilen Refactor Sırası

1. Test eklemeden büyük refactor yapma.
2. Önce `import_dicts.py` testlerini yaz; en izole modül budur.
3. Sonra `graph.py` queue/global state düzeltmelerini yap.
4. Daha sonra `couplateV2.py` fonksiyonlara bölünsün.
5. En son Excel dışı hızlı cache formatı eklensin.
