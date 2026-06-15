# Antigravity IDE Kurulum ve Kullanım Rehberi

Bu proje Antigravity IDE ile geliştirilecekse mevcut `README.md`, `AGENTS.md`, `GEMINI.md` ve `docs/` dosyaları kullanılabilir. Antigravity için ek olarak `.agents/` klasörü de verilmiştir.

## Önerilen dosya yerleşimi

```text
project-root/
├── couplateV2.py
├── import_dicts.py
├── graph.py
├── README.md
├── AGENTS.md
├── GEMINI.md
├── docs/
│   ├── ARCHITECTURE.md
│   ├── DATA_CONTRACTS.md
│   ├── GRAPH_TRACE_ALGORITHM.md
│   ├── REFACTORING_ROADMAP.md
│   └── CHANGE_REQUEST_TEMPLATE.md
└── .agents/
    ├── agents.md
    ├── skills/
    │   ├── understand_project.md
    │   ├── modify_trace_algorithm.md
    │   ├── modify_lookup_loader.md
    │   ├── modify_spatial_join.md
    │   └── audit_patch.md
    └── workflows/
        ├── trace_fix.md
        ├── lookup_fix.md
        ├── spatial_join_change.md
        └── performance_refactor.md
```

## Antigravity için hangileri gerekli?

Minimum yeterli set:

1. `AGENTS.md`
2. `GEMINI.md`
3. `README.md`
4. `docs/ARCHITECTURE.md`
5. `docs/DATA_CONTRACTS.md`
6. `docs/GRAPH_TRACE_ALGORITHM.md`

Daha iyi Antigravity deneyimi için ek set:

1. `.agents/agents.md`
2. `.agents/skills/*.md`
3. `.agents/workflows/*.md`

## Neden ek `.agents/` klasörü var?

Antigravity agent-first çalıştığı için görevleri persona, skill ve workflow olarak ayırmak faydalıdır. Bu sayede agent her seferinde bütün kod tabanını okumaz; sadece ilgili rehber dosyasını, ilgili skill dosyasını ve görevle ilişkili Python fonksiyonlarını okur.

## Kullanım önerisi

Antigravity içinde şu şekilde prompt ver:

```text
/trace_fix
Başlangıç hücresi 304374 için trace sonucu hatalı. Mevcut davranışı bozmadan hatayı analiz et, minimal patch öner, değiştirdiğin fonksiyonları açıkla ve nasıl test edeceğimi yaz.
```

veya:

```text
/performance_refactor
Graph trace algoritmasında token ve runtime maliyetini azaltacak güvenli refactor yap. Öncelik: queue pop(0), tekrar eden shortest path çağrıları, gereksiz string/list dönüşümleri.
```

## Ajanlardan beklenen çıktı formatı

Her Antigravity görevi sonunda agent şunları üretmelidir:

1. Değişiklik planı
2. Değişen dosyalar
3. Korunan veri sözleşmeleri
4. Riskli varsayımlar
5. Test komutları veya manuel test adımları
6. Geri alma yöntemi

## Güvenlik ve veri koruma

- GIS verileri büyük ve hassas olabilir; agent gereksiz dosya taraması yapmamalıdır.
- `couplateV2.py` yalnızca GIS lookup üretimi gerekiyorsa çalıştırılmalıdır.
- Excel lookup şeması değiştirilecekse önce `docs/DATA_CONTRACTS.md` güncellenmelidir.
- `ABB_INT_ID`, koordinat hassasiyeti ve Excel sheet isimleri korunmalıdır.
