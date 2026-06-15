# Graph ve Trace Algoritması Rehberi

Bu dosya özellikle `graph.py` üzerinde çalışacak ajanlar için hazırlanmıştır.

## Graph Sınıfı Yaşam Döngüsü

```python
g = Graph()
g.init_dicts()
g.make_graph()
g.search_and_create_tree(start_hucre_id)
```

### `init_dicts()`

`import_dicts.load_and_unpack_all()` çağırır ve Excel lookup dosyalarını belleğe alır.

Önemli alanlar:

- `self.m2h`: merkez -> hücre listesi.
- `self.h2a`: hücre -> ayırıcı listesi.
- `self.ayirici_id_2_coord`: ayırıcı ID -> koordinat.
- `self.ayirici_coord_2_ayirici_id_dict`: koordinat -> ayırıcı ID.
- `self.ayirici_coord_2_baglanti_coord_dict`: ayırıcı koordinatı -> bağlantı start/end koordinatları.
- `self.start_end_2_edge_id`: iki koordinat arası edge ID.
- `self.hat_edge_ids`: `TYPE == "HAT"` edge ID seti.
- `self.hat_mesafe_dict`: HAT ID -> uzunluk.
- `self.hucre_2_ayirici_top`, `self.hucre_2_ayirici_bottom`: 2.5 bara mantığı için top/bottom ayırıcılar.
- `self.hucre_tek_bara_top_ayirici`, `self.hucre_tek_bara_bottom_ayirici`: tek bara merkezler için top/bottom ayırıcılar.

### `make_graph()`

- `point_df["NODE_ID"]` değerlerini node olarak ekler.
- Ayırıcı koordinatlarını ayrıca node olarak ekler.
- `edge_df[["NODE_1", "NODE_2", "EDGE_ID"]]` ile edge ekler.
- Edge attribute olarak `name = EDGE_ID` kullanılır.

## Trace Ana Akışı

`search_and_create_tree(start_hucre_id)` başlangıç hücresinden trace başlatır.

Akış:

1. `preperation(start_hucre_id)` çağrılır.
2. Başlangıç hücresindeki uygun top/bottom ayırıcılar belirlenir.
3. Her başlangıç ayırıcısı için ayrı queue ve visited set oluşturulur.
4. Queue’dan ayırıcı koordinatı alınır.
5. `pre_process()` ile downstream bağlantıya giden path adayları üretilir.
6. `create_temp_bufferV2()` path adaylarını alt merkez / sonlanan hat olarak sınıflandırır.
7. `filter_and_clear_bufferV2()` en kısa veya en uzun path’i seçer ve HAT dışı edge ID’leri temizler.
8. `prepare_connection()` bağlantı satırını tabloya ekler ve gerekiyorsa alt merkez içi trace için yeni queue elemanları üretir.
9. Sonuçlar Excel’e yazılır.

## Başlangıç Hazırlığı

### `ayirici_list_finder(hucre_id)`

Merkezin tek bara veya 2.5 bara olmasına göre doğru top/bottom ayırıcı dictionary’sini seçer.

- Tek bara merkez: `hucre_tek_bara_top_ayirici`, `hucre_tek_bara_bottom_ayirici`
- 2.5 bara merkez: `hucre_2_ayirici_top`, `hucre_2_ayirici_bottom`

### `baslangic_setter(...)`

Başlangıçta trace’in hangi ayırıcı koordinatından başlayacağını belirler.

Mantık:

- Tek bara merkezde aynı hücrede iki ayırıcı varsa top -> bottom geçişi tabloya 0 mesafe ile eklenir.
- Tek ayırıcılı hücrede aynı ayırıcı hem top hem bottom kabul edilir.
- 2.5 bara merkezde bottom ayırıcı path içindeyse başlangıç bottom’dan devam eder; değilse source ayırıcıdan devam eder.

## Path Üretme ve Filtreleme

### `pre_process(source_node)`

- Source graph içinde mi kontrol eder.
- Source ayırıcıya ait bağlantı start/end koordinatlarını bulur.
- `nx.single_source_shortest_path(..., cutoff=50)` ile aday path’leri üretir.

### `create_temp_bufferV2(...)`

Her path için:

- Path bağlantı start/end koordinatlarından geçiyor mu?
- Path uzunluğu en az 4 node mu?
- Target farklı bir merkeze mi ait?
- Target ayırıcı mı?
- Path içinde `HAT` edge var mı?

Sınıflandırma:

- `is_in_ayirici and include_cable` -> `alt_merkez`
- `not is_in_ayirici and include_cable` -> `sonlanan_hat`
- Aksi halde ignore.

### `filter_and_clear_bufferV2(...)`

- `alt_merkez` için en kısa path seçilir.
- `sonlanan_hat` için en uzun path seçilir.
- `path_ids` içinden sadece `HAT` edge ID’leri bırakılır.

## Bağlantı Tablosu Üretme

### `prepare_connection(...)`

Trace sonucunu tabloya ekler:

```text
(from_ayirici, to_ayirici, mesafe, path_ids, anahtar_durumu)
```

Sonra alt merkeze göre karar verir:

- Eğer `filter_param == "sonlanan_hat"`: tabloya ekle ve devam etme.
- Eğer `filter_param == "alt_merkez"` ve hedef merkez root merkez ise: tabloya ekle ve devam etme.
- Eğer hedef merkez root değilse:
  - Tek bara merkez: `trace_merkez_ici_1_bara(...)`
  - 2.5 bara merkez: `trace_merkez_ici_2_5_bara(...)`

## Merkez İçi Trace

### Tek Bara — `trace_merkez_ici_1_bara(...)`

- Giriş hücresinin bottom ve top ayırıcılarını bulur.
- Çıkış hücrelerini listeler.
- Giriş top ayırıcısından çıkış hücresi top/bottom ayırıcılarına kısa yol arar.
- Merkez içi geçişleri 0 mesafe ile tabloya ekler.
- Dış hatta devam edecek ayırıcı koordinatlarını queue’ya döndürür.

### 2.5 Bara — `trace_merkez_ici_2_5_bara(...)`

İki senaryo uygular:

1. **Aynı bara üzerinden doğrudan geçiş:** giriş top ayırıcısı ile aynı bara numarasındaki çıkış ayırıcısı eşleştirilir.
2. **Bağlama/kuplaj hücresi üzerinden geçiş:** giriş barasından bağlama hücresine, oradan farklı baraya ve çıkış hücresine geçiş kurulur.

## Bilinen Zayıf Noktalar

Ajanlar bu alanlarda dikkatli olmalıdır:

- `queue` listesi `pop(0)` ile kullanılıyor; büyük trace için `deque` daha uygundur.
- `connection_and_length`, `fot_text`, `count` globaldir; instance alanına taşınmalıdır.
- `preperation()` bazı hata durumlarında liste yerine `(None, None)` döndürebilir; bu, caller tarafında unpack hatası doğurabilir. Güvenli dönüş `[]` olmalıdır.
- `get_line_id()` boş bırakılmıştır.
- `get_ayirici_ids_in_path()` ve `create_connection_node_list()` tamamlanmamış/yanlış görünüyor.
- `path_ids` bazen liste, bazen string liste olarak kullanılıyor; veri tipi standartlaştırılmalıdır.
- `start_end_2_edge_id[(path[i], path[i+1])]` doğrudan erişim KeyError üretebilir; `.get()` + hata logu daha güvenlidir.

## Test İçin Minimum Senaryolar

1. Tek ayırıcılı tek bara hücre.
2. İki ayırıcılı tek bara hücre.
3. 2.5 bara merkezde aynı bara üzerinden çıkış.
4. 2.5 bara merkezde bağlama hücresi üzerinden çıkış.
5. Sonlanan hat.
6. Root merkeze geri dönen path.
7. Bağlantı koordinatı olmayan ayırıcı.
8. HAT edge içermeyen path.
