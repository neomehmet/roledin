# AI Coding Agent Talimatları

Bu dosya Codex, Claude Code, Cursor, Copilot, Gemini CLI, Roo Code ve benzeri yapay zekâ kodlama ajanları için yüksek sinyal / düşük token rehberidir.

## Öncelikli Amaç

Kod geliştirirken tüm dosyaları baştan sona okumadan doğru bağlamı kullan:

1. Önce `README.md` dosyasını oku.
2. Veri şeması veya lookup ilişkisi gerekiyorsa `docs/DATA_CONTRACTS.md` dosyasını oku.
3. Graph/trace algoritması gerekiyorsa `docs/GRAPH_TRACE_ALGORITHM.md` dosyasını oku.
4. Refactor/performance isteniyorsa `docs/REFACTORING_ROADMAP.md` dosyasını oku.
5. Sonra yalnızca görevle doğrudan ilgili Python dosyasını aç.

## Kod Dosyalarının Rolü

- `couplateV2.py`: GIS shapefile -> spatial join -> lookup Excel üretimi.
- `import_dicts.py`: lookup Excel -> Python dictionary/DataFrame yükleme.
- `graph.py`: dictionary + DataFrame -> NetworkX graph + trace çıktısı.

## Kesinlikle Korunacak Davranışlar

- `ABB_INT_ID` ana kimliktir; ID değerleri mümkün olduğunca `int` kalmalıdır.
- Koordinatlar `PRECISION = 8` ile yuvarlanmış tuple olarak kullanılır.
- `start_end_2_edge_id` iki yönlüdür: `(node1,node2)` ve `(node2,node1)` aynı edge ID’ye gitmelidir.
- `NORMAL_ANA == "KAPALI"` ise ayırıcı durumu `1`, diğer durumlarda `0` kabul edilir.
- `edge_df.TYPE == "HAT"` olan edge’ler kablo/hat mesafesi hesabında kullanılır.
- `path_ids` DataFrame aşamasında string liste olarak gelebilir; güvenli dönüşüm için `ast.literal_eval` kullanılır.
- GIS değişmediği sürece `couplateV2.py` tekrar çalıştırılmamalı; mevcut Excel lookup dosyaları kullanılmalıdır.

## Değişiklik Yaparken Dikkat

- Geniş çaplı yeniden yazım yapma; önce küçük, test edilebilir değişiklik öner.
- Yeni davranış eklerken mevcut Excel çıktı isimlerini ve sheet isimlerini koru.
- Spatial join predicate değiştirirken (`within`, `intersects`, `touches`) gerekçeyi açık yaz.
- Graph traversal değişikliklerinde tek bara ve 2.5 bara mantığını ayrı ayrı koru.
- Hata yakalama eklerken sessizce veri kaybetme; `hata_loglari` veya `logging` ile raporla.
- Performans için önce `list.pop(0)` -> `collections.deque`, gereksiz string/list dönüşümleri, tekrar eden shortest path çağrıları ve Excel I/O darboğazlarını incele.

## Ajan İçin Dosya Okuma Stratejisi

### Eğer görev “trace sonucu yanlış” ise

Sıra:

1. `docs/GRAPH_TRACE_ALGORITHM.md`
2. `graph.py` içinde sadece şu fonksiyonlar:
   - `search_and_create_tree`
   - `preperation`
   - `baslangic_setter`
   - `pre_process`
   - `create_temp_bufferV2`
   - `filter_and_clear_bufferV2`
   - `prepare_connection`
   - `trace_merkez_ici_1_bara`
   - `trace_merkez_ici_2_5_bara`

### Eğer görev “lookup dosyası okunmuyor” ise

Sıra:

1. `docs/DATA_CONTRACTS.md`
2. `import_dicts.py` içinde:
   - `safe_literal_eval`
   - `_load_sheet_as_dict`
   - `load_and_unpack_all`
   - `load_dataframes_only`

### Eğer görev “GIS ilişkilendirmesi yanlış” ise

Sıra:

1. `docs/ARCHITECTURE.md`
2. `docs/DATA_CONTRACTS.md`
3. `couplateV2.py` içinde ilgili spatial join bölümü.

## Modernizasyon Hedefleri

Kısa vadeli güvenli iyileştirmeler:

- `couplateV2.py` için `main()` ve `if __name__ == "__main__"` yapısı kur.
- `graph.py` içinde global değişkenleri sınıf alanına taşı.
- `queue.pop(0)` yerine `collections.deque.popleft()` kullan.
- `get_path_ids()` içinde eksik edge için kontrollü hata üret.
- `print()` yerine `logging` kullan.
- Veri şeması validasyonu ekle.

Orta vadeli iyileştirmeler:

- Excel yerine büyük veriler için Parquet/Feather/SQLite/GeoPackage desteği ekle.
- Lookup sözlüklerini typed `dataclass` / `TypedDict` ile belgeleyip doğrula.
- Spatial join işlemlerini fonksiyonlara böl ve test edilebilir hale getir.
- NetworkX shortest path sonuçlarını cache’le veya merkez bazlı alt graph kullan.

## Patch Formatı

Kod değiştirirken cevapta şunları ver:

1. Değiştirilen dosyalar.
2. Hangi davranış korundu.
3. Hangi hata/performance problemi düzeltildi.
4. Nasıl test edileceği.
5. Riskli varsayımlar.
