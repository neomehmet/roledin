# Veri Sözleşmeleri ve Lookup Şemaları

Bu dosya, ajanların ve geliştiricilerin hangi dosyanın hangi veriyi ürettiğini hızlı anlaması için hazırlanmıştır.

## Girdi GIS Dosyaları

Varsayılan dosya yolları `couplateV2.py` içinde sabittir:

| Değişken | Dosya | Beklenen geometri | Ana kolonlar |
|---|---|---|---|
| `MERKEZ_PATH` | `LGC_MERKEZ.shp` | Polygon | `ABB_INT_ID`, `geometry` |
| `HAT_PATH` | `H_ENERJI_NAKIL_HATTI.shp` | LineString | `ABB_INT_ID`, `GERILIM`, `UZUNLUK`, `geometry` |
| `HUCRE_PATH` | `T_HUCRE.shp` | Polygon | `ABB_INT_ID`, `GERILIM`, `ALTTIP`, `geometry` |
| `BARA_PATH` | `T_OG_BARA.shp` | LineString | `ABB_INT_ID`, `GERILIM`, `BARA_NO`, `geometry` |
| `KESICI_PATH` | `T_OG_KESICI.shp` | Point | `ABB_INT_ID`, `GERILIM`, `geometry` |
| `AYIRICI_PATH` | `T_OG_AYIRICI.shp` | Point | `ABB_INT_ID`, `GERILIM`, `ALTTIP`, `NORMAL_ANA`, `geometry` |
| `T_OG_KABLO_BAGLANTISI_PATH` | `T_OG_KABLO_BAGLANTISI.shp` | LineString/geometry | `ABB_INT_ID`, `GERILIM`, `geometry` |

## Temel Filtreler

- Hücre, bara, hat, kesici, ayırıcı ve bağlantı için `GERILIM == "34.5 kV"` filtrelenir.
- Ayırıcı tarafında `ALTTIP` içinde `TOPRAKLI` geçen kayıtlar çıkarılır.
- Ayırıcı durumunda `NORMAL_ANA.upper() == "KAPALI"` ise lookup değeri `1`, aksi halde `0` olur.

## Koordinat Sözleşmesi

- `PRECISION = 8` kullanılır.
- LineString verilerinde:
  - `START_POINT = (round(x0, 8), round(y0, 8))`
  - `END_POINT = (round(xn, 8), round(yn, 8))`
- Point verilerinde geometri doğrudan yuvarlanmış `Point` olur.
- Graph node’ları çoğunlukla `(x, y)` tuple’dır.

## `couplateV2.py` Excel Çıktıları

| Dosya | İçerik |
|---|---|
| `bara_lookup_dicts.xlsx` | Bara merkez/hücre/geometri/ayırıcı/kesici ilişkileri. |
| `ayirici_lookup_dicts.xlsx` | Ayırıcı hücre/merkez/geometri/bara/bağlantı/durum ilişkileri. |
| `kesici_lookup_dicts.xlsx` | Kesici hücre/merkez/geometri/ayırıcı/bara/bağlantı ilişkileri. |
| `hucre_lookup_dicts.xlsx` | Hücre merkez/geometri/ayırıcı/kesici/bara/bağlantı/top-bottom/coupling/çıkış ilişkileri. |
| `merkez_lookup_dicts.xlsx` | Merkez geometri/hücre/ayırıcı/kesici/bara/bağlantı/tek-bara ilişkileri. |
| `baglanti_lookup_dicts.xlsx` | Bağlantı hücre/merkez/geometri/kesici/ayırıcı/bara/start-end ilişkileri. |
| `edge_lookup_dicts.xlsx` | `start_end_2_edge_id` ve `hat_mesafe` sözlükleri. |
| `edge_df.xlsx` | Graph edge tablosu: `EDGE_ID`, `NODE_1`, `NODE_2`, `TYPE`. |
| `point_df.xlsx` | Graph node tablosu: `NODE_ID`. |
| `all_segment.xlsx` | Segment tablosu: `START_POINT`, `END_POINT`, `ABB_INT_ID`. |
| `bara_2_baglanti.xlsx` | Bara-bağlantı spatial join sonucu. |
| `baglanti_processed.xlsx` | Merkez/hücre bilgisi eklenmiş bağlantı verisi. |
| `ayirici_anahtar_status.xlsx` | Ayırıcı anahtar durumları. |

## Lookup Dictionary Adları

`import_dicts.load_and_unpack_all()` aşağıdaki anahtarları döndürür. Ajanlar mümkünse bu isimleri korumalıdır.

### Bara

- `bara_2_hucre_dict`
- `bara_2_bara_geometry_dict`
- `bara_2_merkez_dict`
- `bara_2_ayirici_dict`
- `bara_2_kesici_dict`

### Ayırıcı

- `ayirici_2_hucre_dict`
- `ayirici_2_ayirici_geometry_dict`
- `ayirici_2_merkez_dict`
- `ayirici_2_kesici_dict`
- `ayirici_2_bara_dict`
- `ayirici_2_baglanti_dict`
- `ayirici_coord_2_baglanti_coord_dict`
- `ayirici_coord_2_merkez_dict`
- `ayirici_coord_2_ayirici_id_dict`
- `ayirici_2_normal_ana_dict`
- `ayirici_2_bara_no_dict`

### Kesici

- `kesici_2_hucre_dict`
- `kesici_2_kesici_geometry_dict`
- `kesici_2_merkez_dict`
- `kesici_2_ayirici_dict`
- `kesici_2_bara_dict`
- `kesici_2_baglanti_dict`

### Hücre

- `hucre_2_merkez_dict`
- `hucre_2_hucre_geometry_dict`
- `hucre_2_ayiricilar_abb_id_dict`
- `hucre_2_kesiciler_abb_id_dict`
- `hucre_2_baralar_abb_id_dict`
- `hucre_2_baglantilar_abb_id_dict`
- `hucre_name_dict`
- `hucre_2_ayirici_top_dict`
- `hucre_2_ayirici_bottom_dict`
- `hucre_2_coupling_bays_dict`
- `hucre_2_cikis_hucreleri_dict`
- `hucre_tek_bara_top_ayirici_dict`
- `hucre_tek_bara_bottom_ayirici_dict`

### Merkez

- `merkez_2_merkez_geometry_dict`
- `merkez_2_hucre_dict`
- `merkez_2_ayiricilar_abb_id_dict`
- `merkez_2_kesiciler_abb_id_dict`
- `merkez_2_baralar_abb_id_dict`
- `merkez_2_baglantilar_abb_id_dict`
- `merkez_tek_bara_mi_dict`
- `merkez_hucre_length_dict` — `import_dicts.py` içinde türetilir.

### Bağlantı

- `baglanti_2_hucre_dict`
- `baglanti_2_baglanti_geometry_dict`
- `baglanti_2_merkez_dict`
- `baglanti_2_kesiciler_abb_id_dict`
- `baglanti_2_ayiricilar_abb_id_dict`
- `baglanti_2_baralar_abb_id_dict`
- `baglanti_2_baglanti_end_start_coord_dict`

### Edge / Node

- `start_end_2_edge_id`
- `hat_mesafe_dict`
- `node_to_edge_dict`
- `point_df`
- `all_segments_df`
- `edge_df`
- `bara_2_baglanti_df`
- `ayirici_anahtar_status_df`

## Graph Edge Sözleşmesi

`edge_df` kolonları:

| Kolon | Anlam |
|---|---|
| `EDGE_ID` | Fiziksel GIS elemanının `ABB_INT_ID` değeri. |
| `NODE_1` | Başlangıç koordinat tuple’ı. |
| `NODE_2` | Bitiş koordinat tuple’ı. |
| `TYPE` | `HAT`, `BARA` veya `BAGLANTI`. |

## Trace Sonuç Sözleşmesi

`graph.py` trace çıktısı şu kolonlara sahiptir:

| Kolon | Anlam |
|---|---|
| `from_ayirici` | Kaynak ayırıcı ID. |
| `to_ayirici` | Hedef ayırıcı ID. |
| `length` | HAT edge uzunluklarının toplamı. |
| `enh_ids` | Path üzerinde bulunan hat/kablo edge ID’leri. |
| `anahtar_durumu` | Kaynak ayırıcının normal anahtar durumu. |

## Veri Doğrulama Kontrol Listesi

Kod geliştiren ajanlar yeni özellik eklemeden önce şunları kontrol etmelidir:

- Lookup dosyası mevcut mu?
- Sheet adı 31 karakter sınırına takılıyor mu?
- `Key` ve `Value` kolonları var mı?
- ID’ler float olarak gelmişse int’e dönmüş mü?
- Tuple koordinatlar string olarak kaldı mı?
- WKT geometri gerçekten `shapely` nesnesine dönmeli mi, yoksa string kalması yeterli mi?
- `start_end_2_edge_id` iki yönlü mü?
- `edge_df.TYPE` değerleri beklenen üç tipten biri mi?
