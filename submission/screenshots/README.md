# Tài Liệu Minh Chứng Cấu Trúc Lưu Trữ & Transaction Log Lakehouse

Tài liệu này cung cấp đầy đủ các minh chứng kỹ thuật phục vụ việc chấm điểm theo [rubric.md](../../rubric.md):
1. **Cấu trúc lưu trữ vật lý trên đĩa (`_lakehouse/`)** bao gồm các tầng Bronze, Silver, Gold, Scratch và Iceberg Catalog.
2. **Chi tiết cấu trúc và giải thích Transaction Log của Delta Lake (`_delta_log/00000000000000000000.json`)**.

---

## 1. Cấu Trúc Lưu Trữ Vật Lý Của Lakehouse (`_lakehouse/`)

###  Sơ Đồ Phân Cấp Kiến Trúc Tổng Quan:

```text
_lakehouse/
├── blobs/                        # Lưu trữ 200 file nhị phân phi cấu trúc (Multimodal Blobs, dung lượng 12.5 MB) phục vụ cho NB7
├── bronze/                       # TẦNG BRONZE: Dữ liệu thô ban đầu (Raw, Append-only)
│   ├── agent_traces/             # Log vết thực thi thô của AI Agent (1,578 steps / 300 sessions cho NB8)
│   ├── docs_multimodal/          # Tài liệu đa phương thức kèm đường dẫn trỏ tới blobs (NB7)
│   └── llm_calls_raw/            # 200,000 dòng log API gọi LLM thô ở định dạng JSON (NB4)
├── silver/                       # TẦNG SILVER: Dữ liệu đã làm sạch, chuẩn hóa schema và phân vùng
│   ├── agent_trajectories/       # Trajectory đã parse JSON, deduplicate, phân vùng theo agent_version (NB8)
│   ├── docs_embedded/            # Văn bản kèm vector embeddings (kích thước 256 chiều, hỗ trợ int8) (NB7)
│   ├── llm_calls/                # 190,052 dòng đã loại bỏ trùng lặp (dedup), phân vùng theo date (NB4)
│   └── training_corpus_governed/ # Tập ngữ liệu phân chia theo 4 rổ EU AI Act Art. 10 (NB8)
├── gold/                         # TẦNG GOLD: Dữ liệu tổng hợp KPI phục vụ trực tiếp cho báo cáo và AI Training
│   ├── agent_performance/        # Tổng hợp reward và số bước trung bình theo từng chính sách agent (NB8)
│   └── llm_daily_metrics/        # Chỉ số KPI hàng ngày: Latency p50/p95, Cost USD, Error Rate (8 ngày × 3 model) (NB4)
├── iceberg/                      # APACHE ICEBERG CATALOGS: Control Plane quản lý metadata
│   ├── nb5/                      # Catalog SQLite + Warehouse cho NB5 (Hidden Partitioning & Schema Evolution)
│   ├── nb6/                      # Catalog SQLite + Warehouse cho NB6 (Quản lý Snapshot Expiry & Maintenance)
│   └── nb8/                      # Catalog SQLite + Warehouse cho NB8 (Phân vùng theo EU AI Act Art. 10)
└── scratch/                      # VÙNG THỰC NGHIỆM ĐO ĐẠC: Các bảng phục vụ benchmark
    ├── customers_tt/             # Bảng kiểm chứng Time Travel (5 versions), Upsert MERGE và RESTORE (NB3)
    ├── events_smallfiles/        # Bảng tái hiện vấn đề 200 small-files và đo lường Z-ORDER (NB2)
    ├── maintenance_delta/        # Bảng thực nghiệm dọn dẹp VACUUM và bẫy Orphan Files (NB6)
    ├── users_delta/              # Bảng kiểm chứng Schema Enforcement & Schema Evolution (NB1)
    ├── emb_f32 / emb_int8/       # Bảng so sánh quantization vector float32 vs int8 (NB7)
    └── docs_cdf/                 # Bảng kiểm chứng Change Data Feed (NB7)
```

---

###  Toàn Bộ Cây Thư Mục Thực Tế (`tree _lakehouse/` — 101 Directories, 1152 Files):

<details>
<summary><b> Nhấn vào đây để mở rộng / thu gọn toàn bộ cây thư mục chi tiết (1152 files)</b></summary>

```text
khanhhuy@MacBook-Air-cua-Nguyen-84 Day18-Track2-Lakehouse-Lab-2A202601591-NguyenBaKhanhHuy % tree _lakehouse/
_lakehouse/
├── blobs
│   ├── frame_0000.bin
│   ├── frame_0001.bin
│   ├── frame_0002.bin
│   ├── frame_0003.bin
│   ├── frame_0004.bin
│   ├── frame_0005.bin
│   ├── frame_0006.bin
│   ├── frame_0007.bin
│   ├── frame_0008.bin
│   ├── frame_0009.bin
│   ├── frame_0010.bin
│   ├── frame_0011.bin
│   ├── frame_0012.bin
│   ├── frame_0013.bin
│   ├── frame_0014.bin
│   ├── frame_0015.bin
│   ├── frame_0016.bin
│   ├── frame_0017.bin
│   ├── frame_0018.bin
│   ├── frame_0019.bin
│   ├── frame_0020.bin
│   ├── frame_0021.bin
│   ├── frame_0022.bin
│   ├── frame_0023.bin
│   ├── frame_0024.bin
│   ├── frame_0025.bin
│   ├── frame_0026.bin
│   ├── frame_0027.bin
│   ├── frame_0028.bin
│   ├── frame_0029.bin
│   ├── frame_0030.bin
│   ├── frame_0031.bin
│   ├── frame_0032.bin
│   ├── frame_0033.bin
│   ├── frame_0034.bin
│   ├── frame_0035.bin
│   ├── frame_0036.bin
│   ├── frame_0037.bin
│   ├── frame_0038.bin
│   ├── frame_0039.bin
│   ├── frame_0040.bin
│   ├── frame_0041.bin
│   ├── frame_0042.bin
│   ├── frame_0043.bin
│   ├── frame_0044.bin
│   ├── frame_0045.bin
│   ├── frame_0046.bin
│   ├── frame_0047.bin
│   ├── frame_0048.bin
│   ├── frame_0049.bin
│   ├── frame_0050.bin
│   ├── frame_0051.bin
│   ├── frame_0052.bin
│   ├── frame_0053.bin
│   ├── frame_0054.bin
│   ├── frame_0055.bin
│   ├── frame_0056.bin
│   ├── frame_0057.bin
│   ├── frame_0058.bin
│   ├── frame_0059.bin
│   ├── frame_0060.bin
│   ├── frame_0061.bin
│   ├── frame_0062.bin
│   ├── frame_0063.bin
│   ├── frame_0064.bin
│   ├── frame_0065.bin
│   ├── frame_0066.bin
│   ├── frame_0067.bin
│   ├── frame_0068.bin
│   ├── frame_0069.bin
│   ├── frame_0070.bin
│   ├── frame_0071.bin
│   ├── frame_0072.bin
│   ├── frame_0073.bin
│   ├── frame_0074.bin
│   ├── frame_0075.bin
│   ├── frame_0076.bin
│   ├── frame_0077.bin
│   ├── frame_0078.bin
│   ├── frame_0079.bin
│   ├── frame_0080.bin
│   ├── frame_0081.bin
│   ├── frame_0082.bin
│   ├── frame_0083.bin
│   ├── frame_0084.bin
│   ├── frame_0085.bin
│   ├── frame_0086.bin
│   ├── frame_0087.bin
│   ├── frame_0088.bin
│   ├── frame_0089.bin
│   ├── frame_0090.bin
│   ├── frame_0091.bin
│   ├── frame_0092.bin
│   ├── frame_0093.bin
│   ├── frame_0094.bin
│   ├── frame_0095.bin
│   ├── frame_0096.bin
│   ├── frame_0097.bin
│   ├── frame_0098.bin
│   ├── frame_0099.bin
│   ├── frame_0100.bin
│   ├── frame_0101.bin
│   ├── frame_0102.bin
│   ├── frame_0103.bin
│   ├── frame_0104.bin
│   ├── frame_0105.bin
│   ├── frame_0106.bin
│   ├── frame_0107.bin
│   ├── frame_0108.bin
│   ├── frame_0109.bin
│   ├── frame_0110.bin
│   ├── frame_0111.bin
│   ├── frame_0112.bin
│   ├── frame_0113.bin
│   ├── frame_0114.bin
│   ├── frame_0115.bin
│   ├── frame_0116.bin
│   ├── frame_0117.bin
│   ├── frame_0118.bin
│   ├── frame_0119.bin
│   ├── frame_0120.bin
│   ├── frame_0121.bin
│   ├── frame_0122.bin
│   ├── frame_0123.bin
│   ├── frame_0124.bin
│   ├── frame_0125.bin
│   ├── frame_0126.bin
│   ├── frame_0127.bin
│   ├── frame_0128.bin
│   ├── frame_0129.bin
│   ├── frame_0130.bin
│   ├── frame_0131.bin
│   ├── frame_0132.bin
│   ├── frame_0133.bin
│   ├── frame_0134.bin
│   ├── frame_0135.bin
│   ├── frame_0136.bin
│   ├── frame_0137.bin
│   ├── frame_0138.bin
│   ├── frame_0139.bin
│   ├── frame_0140.bin
│   ├── frame_0141.bin
│   ├── frame_0142.bin
│   ├── frame_0143.bin
│   ├── frame_0144.bin
│   ├── frame_0145.bin
│   ├── frame_0146.bin
│   ├── frame_0147.bin
│   ├── frame_0148.bin
│   ├── frame_0149.bin
│   ├── frame_0150.bin
│   ├── frame_0151.bin
│   ├── frame_0152.bin
│   ├── frame_0153.bin
│   ├── frame_0154.bin
│   ├── frame_0155.bin
│   ├── frame_0156.bin
│   ├── frame_0157.bin
│   ├── frame_0158.bin
│   ├── frame_0159.bin
│   ├── frame_0160.bin
│   ├── frame_0161.bin
│   ├── frame_0162.bin
│   ├── frame_0163.bin
│   ├── frame_0164.bin
│   ├── frame_0165.bin
│   ├── frame_0166.bin
│   ├── frame_0167.bin
│   ├── frame_0168.bin
│   ├── frame_0169.bin
│   ├── frame_0170.bin
│   ├── frame_0171.bin
│   ├── frame_0172.bin
│   ├── frame_0173.bin
│   ├── frame_0174.bin
│   ├── frame_0175.bin
│   ├── frame_0176.bin
│   ├── frame_0177.bin
│   ├── frame_0178.bin
│   ├── frame_0179.bin
│   ├── frame_0180.bin
│   ├── frame_0181.bin
│   ├── frame_0182.bin
│   ├── frame_0183.bin
│   ├── frame_0184.bin
│   ├── frame_0185.bin
│   ├── frame_0186.bin
│   ├── frame_0187.bin
│   ├── frame_0188.bin
│   ├── frame_0189.bin
│   ├── frame_0190.bin
│   ├── frame_0191.bin
│   ├── frame_0192.bin
│   ├── frame_0193.bin
│   ├── frame_0194.bin
│   ├── frame_0195.bin
│   ├── frame_0196.bin
│   ├── frame_0197.bin
│   ├── frame_0198.bin
│   └── frame_0199.bin
├── bronze
│   ├── agent_traces
│   │   ├── _delta_log
│   │   │   └── 00000000000000000000.json
│   │   └── part-00000-3401dd51-3a4f-4407-9a83-6bc4f6ece855-c000.snappy.parquet
│   ├── docs_multimodal
│   │   ├── _delta_log
│   │   │   └── 00000000000000000000.json
│   │   └── part-00000-6fb69eed-3f03-461f-99ec-ad76ddd75c6c-c000.snappy.parquet
│   └── llm_calls_raw
│       ├── _delta_log
│       │   └── 00000000000000000000.json
│       └── part-00000-6dc5a775-2863-4c44-aaf9-37c2c8a6c2a6-c000.snappy.parquet
├── gold
│   ├── agent_performance
│   │   ├── _delta_log
│   │   │   └── 00000000000000000000.json
│   │   └── part-00000-e940df57-ec60-4d0a-9d26-3381f0f595d7-c000.snappy.parquet
│   └── llm_daily_metrics
│       ├── _delta_log
│       │   ├── 00000000000000000000.json
│       │   └── 00000000000000000001.json
│       ├── date=2026-04-01
│       │   ├── part-00000-5bacfd25-8c74-4fd1-bfc3-73fa83e82a75-c000.zstd.parquet
│       │   └── part-00000-aa9b2c44-990f-40d7-8402-be74df9498fd-c000.snappy.parquet
│       ├── date=2026-04-02
│       │   ├── part-00000-286b3dc2-3ab4-4465-a160-4f29eb02009e-c000.zstd.parquet
│       │   └── part-00000-a34cc199-c628-4ac4-ac02-468fe5bcd31e-c000.snappy.parquet
│       ├── date=2026-04-03
│       │   ├── part-00000-6b2ac3b6-c038-4132-af24-f278609cce4c-c000.snappy.parquet
│       │   └── part-00000-e5340650-f117-4cf4-a0aa-44fa9fb56221-c000.zstd.parquet
│       ├── date=2026-04-04
│       │   ├── part-00000-40890586-38ae-405a-bbec-5af29f692a09-c000.snappy.parquet
│       │   └── part-00000-d20601b9-250e-443d-9d02-e03e6ab2973a-c000.zstd.parquet
│       ├── date=2026-04-05
│       │   ├── part-00000-05d61635-377e-4763-a88a-571498ab3911-c000.snappy.parquet
│       │   └── part-00000-71237f73-1f21-421e-b7fa-75a612106b81-c000.zstd.parquet
│       ├── date=2026-04-06
│       │   ├── part-00000-9a252f53-0fa1-44ed-a8e5-e52aa5910f5d-c000.snappy.parquet
│       │   └── part-00000-bdb1c45e-895b-4afe-9504-2c0336913a6b-c000.zstd.parquet
│       ├── date=2026-04-07
│       │   ├── part-00000-3c0ea89c-b7d3-47dc-831f-7f342d864c04-c000.zstd.parquet
│       │   └── part-00000-fd20b5af-29a2-4724-9bfa-f9e1bb8fcda2-c000.snappy.parquet
│       └── date=2026-04-08
│           ├── part-00000-0e79ab0e-e2b1-4c66-ad86-174924a69865-c000.zstd.parquet
│           └── part-00000-f079c7c0-973c-46de-b4ec-3e5ad7632197-c000.snappy.parquet
├── iceberg
│   ├── nb5
│   │   ├── catalog.db
│   │   └── warehouse
│   │       └── lake
│   │           └── llm_events
│   │               ├── data
│   │               │   ├── ts_day=2026-08-01
│   │               │   │   └── 00000-0-f63858e3-c841-493a-bd91-a120edb7a3cc.parquet
│   │               │   ├── ts_day=2026-08-02
│   │               │   │   └── 00000-0-3e47457e-77b0-4f5a-885a-5b185681a4a6.parquet
│   │               │   ├── ts_day=2026-08-03
│   │               │   │   └── 00000-0-c5e6090c-edfc-47ca-aa8f-39dc4052d265.parquet
│   │               │   ├── ts_day=2026-08-04
│   │               │   │   └── 00000-0-31c8dc4d-ec14-482d-b6b5-b1203bb4f702.parquet
│   │               │   ├── ts_day=2026-08-05
│   │               │   │   └── 00000-0-b0acf3fd-b0ca-4ffa-9e8b-c7e5e8d85242.parquet
│   │               │   ├── ts_day=2026-08-06
│   │               │   │   └── 00000-0-10c58ee9-3e74-4d76-b9fa-777daa259a3e.parquet
│   │               │   ├── ts_day=2026-08-07
│   │               │   │   └── 00000-0-115e097a-d602-4283-8638-9839080f91fe.parquet
│   │               │   ├── ts_day=2026-08-08
│   │               │   │   └── 00000-0-71f6a9d8-f38c-4489-b764-4e7cedf9d102.parquet
│   │               │   ├── ts_day=2026-08-09
│   │               │   │   └── 00000-0-8a6d8b21-283a-43d5-aab2-9c8a0ae4d561.parquet
│   │               │   ├── ts_day=2026-08-10
│   │               │   │   └── 00000-0-877ad72f-43b3-43b4-ae8d-5ac59894e6cf.parquet
│   │               │   └── ts_day=2026-08-11
│   │               │       ├── model_id=claude-haiku-4-5
│   │               │       │   └── 00000-0-713cc85f-7191-49fa-a4d4-46872fae3ceb.parquet
│   │               │       ├── model_id=claude-opus-4-7
│   │               │       │   └── 00000-2-713cc85f-7191-49fa-a4d4-46872fae3ceb.parquet
│   │               │       └── model_id=claude-sonnet-4-6
│   │               │           └── 00000-1-713cc85f-7191-49fa-a4d4-46872fae3ceb.parquet
│   │               └── metadata
│   │                   ├── 00000-a6fe0aa9-db74-4514-abf0-0e03e72f97bc.metadata.json
│   │                   ├── 00001-57722d80-d5d7-4e44-a456-d1ac2242179e.metadata.json
│   │                   ├── 00002-7a97fc78-8f14-427a-8459-1082c4996962.metadata.json
│   │                   ├── 00003-32c2744a-c2c0-4883-9a16-0fd0a1f91ff1.metadata.json
│   │                   ├── 00004-fa9c60e4-0cd4-4a4a-ba6e-e97620a3f9b3.metadata.json
│   │                   ├── 00005-d3a432b2-3cca-4668-8dae-79a820c87d62.metadata.json
│   │                   ├── 00006-398feef6-8fd2-4f92-a6a5-c0b2931e945f.metadata.json
│   │                   ├── 00007-86f9165d-52ad-4a83-a800-4b653bbe5b5b.metadata.json
│   │                   ├── 00008-0add6c4e-2921-4436-a79a-fb0657f2a4d4.metadata.json
│   │                   ├── 00009-9a208dd7-00c2-4faa-84be-388ce71d11e5.metadata.json
│   │                   ├── 00010-a60c3309-e93b-445c-a9e1-1f68296ec752.metadata.json
│   │                   ├── 00011-7dfd487c-2c4e-4ea3-a91b-ddc48b283757.metadata.json
│   │                   ├── 00012-3fa0c2dd-4c94-4286-bb67-b3caaa9f7340.metadata.json
│   │                   ├── 00013-abb45179-0738-4afa-b884-fc62d2da80b4.metadata.json
│   │                   ├── 00014-058da884-fa39-4aaa-ab6f-e51b047a82c5.metadata.json
│   │                   ├── 00015-0647ce75-8f3d-46ee-9421-7b27a7ecc48a.metadata.json
│   │                   ├── 10c58ee9-3e74-4d76-b9fa-777daa259a3e-m0.avro
│   │                   ├── 115e097a-d602-4283-8638-9839080f91fe-m0.avro
│   │                   ├── 31c8dc4d-ec14-482d-b6b5-b1203bb4f702-m0.avro
│   │                   ├── 3e47457e-77b0-4f5a-885a-5b185681a4a6-m0.avro
│   │                   ├── 713cc85f-7191-49fa-a4d4-46872fae3ceb-m0.avro
│   │                   ├── 71f6a9d8-f38c-4489-b764-4e7cedf9d102-m0.avro
│   │                   ├── 877ad72f-43b3-43b4-ae8d-5ac59894e6cf-m0.avro
│   │                   ├── 8a6d8b21-283a-43d5-aab2-9c8a0ae4d561-m0.avro
│   │                   ├── b0acf3fd-b0ca-4ffa-9e8b-c7e5e8d85242-m0.avro
│   │                   ├── c5e6090c-edfc-47ca-aa8f-39dc4052d265-m0.avro
│   │                   ├── f63858e3-c841-493a-bd91-a120edb7a3cc-m0.avro
│   │                   ├── snap-1947765882817198608-0-115e097a-d602-4283-8638-9839080f91fe.avro
│   │                   ├── snap-2806445558200743297-0-b0acf3fd-b0ca-4ffa-9e8b-c7e5e8d85242.avro
│   │                   ├── snap-3466613127732320838-0-713cc85f-7191-49fa-a4d4-46872fae3ceb.avro
│   │                   ├── snap-3575761812987287237-0-3e47457e-77b0-4f5a-885a-5b185681a4a6.avro
│   │                   ├── snap-4552815560186933861-0-31c8dc4d-ec14-482d-b6b5-b1203bb4f702.avro
│   │                   ├── snap-4718236542096047875-0-f63858e3-c841-493a-bd91-a120edb7a3cc.avro
│   │                   ├── snap-5274560472635953004-0-71f6a9d8-f38c-4489-b764-4e7cedf9d102.avro
│   │                   ├── snap-5342189727303138027-0-877ad72f-43b3-43b4-ae8d-5ac59894e6cf.avro
│   │                   ├── snap-540435076532952181-0-10c58ee9-3e74-4d76-b9fa-777daa259a3e.avro
│   │                   ├── snap-6787058596693684044-0-c5e6090c-edfc-47ca-aa8f-39dc4052d265.avro
│   │                   └── snap-7218330053158871770-0-8a6d8b21-283a-43d5-aab2-9c8a0ae4d561.avro
│   ├── nb6
│   │   ├── catalog.db
│   │   └── warehouse
│   │       └── lake
│   │           └── maint
│   │               ├── data
│   │               │   ├── 00000-0-112096d0-1f4f-46b1-9a94-0e5bbff4c558.parquet
│   │               │   ├── 00000-0-1f4b7bc2-7116-4d97-a16f-7ad6773d1813.parquet
│   │               │   ├── 00000-0-2772adf7-046d-45e6-ad70-617b891fa068.parquet
│   │               │   ├── 00000-0-2d39c487-2c61-4165-b61c-f67e849a8efa.parquet
│   │               │   ├── 00000-0-4ac4a2b4-07be-4a0b-b799-869746b46d50.parquet
│   │               │   ├── 00000-0-62f1f10d-39ec-4254-92a4-3290690daaac.parquet
│   │               │   ├── 00000-0-630b89b0-367b-4ed4-8aa5-b9655d4add48.parquet
│   │               │   ├── 00000-0-692164e1-ff56-48bd-8f29-1501bfc5a6b0.parquet
│   │               │   ├── 00000-0-72c182b8-6c60-4105-ae5a-9b072c7e2b3d.parquet
│   │               │   ├── 00000-0-817fee01-3eb0-4273-991a-04f4978d0862.parquet
│   │               │   ├── 00000-0-9087b6fc-e47a-4e44-aa9e-77a15c455cac.parquet
│   │               │   ├── 00000-0-9d842861-ebb5-4b2d-a7d2-ad8e24af9763.parquet
│   │               │   ├── 00000-0-a2818425-b79e-48d7-91d1-ed4ae2d9740e.parquet
│   │               │   ├── 00000-0-b084f226-e85b-4456-9c4b-7fbce5fe9a47.parquet
│   │               │   ├── 00000-0-b8b9fcd5-1ee8-496f-8c55-28665656840a.parquet
│   │               │   ├── 00000-0-ba5ec5c6-db31-4137-a015-3c1aedc9ca4a.parquet
│   │               │   ├── 00000-0-d2d51b18-089e-4020-9b80-6461b32e5753.parquet
│   │               │   ├── 00000-0-dac3aa04-c4e7-472d-aed8-c6bb00852936.parquet
│   │               │   ├── 00000-0-e6f83680-b8a6-4afe-9aa6-b0be92fd9131.parquet
│   │               │   └── 00000-0-ed00e373-4bd3-447b-88f8-46d6d7259fbb.parquet
│   │               └── metadata
│   │                   ├── 00000-f5abe1b1-6d19-44af-b7fa-fc07a09c4842.metadata.json
│   │                   ├── 00001-6057d792-8e88-436c-a0c8-b5b72c4a0804.metadata.json
│   │                   ├── 00002-6c9381ba-c473-451e-95b3-029ae4ae39dd.metadata.json
│   │                   ├── 00003-7792574e-2c09-45cd-b84a-bd0682572656.metadata.json
│   │                   ├── 00004-3fc250d9-b61b-4465-bb51-528c03bd21d7.metadata.json
│   │                   ├── 00005-4cd74b18-aa67-43d0-8c59-c6b8a6d16240.metadata.json
│   │                   ├── 00006-2510f261-b894-43a8-bd33-2720daf9cc1b.metadata.json
│   │                   ├── 00007-b9774cdc-b792-4764-83af-d36d882ef235.metadata.json
│   │                   ├── 00008-b1ce2790-d54b-4725-8282-4062d8b89aaa.metadata.json
│   │                   ├── 00009-605b941f-1e77-4e2c-96ab-53d15a3220a8.metadata.json
│   │                   ├── 00010-d3f4cc4c-9cf4-4425-9c6b-480a73a3afd9.metadata.json
│   │                   ├── 00011-2b5baf38-f80e-47be-aa63-334153841ca8.metadata.json
│   │                   ├── 00012-846a6eb1-5af2-42e6-873d-051953dc11a3.metadata.json
│   │                   ├── 00013-de333c2d-ddf7-4478-acfa-988f3fc0027b.metadata.json
│   │                   ├── 00014-000ec6c1-c5eb-483a-8f48-15883707b775.metadata.json
│   │                   ├── 00015-7e63ad0c-09e5-4370-a13b-9b19cd0e9253.metadata.json
│   │                   ├── 00016-3b612de6-26fa-4e88-90b3-29a606373ad9.metadata.json
│   │                   ├── 00017-84f35807-e266-4178-8402-e7b1a7fcf138.metadata.json
│   │                   ├── 00018-75fac2d6-81a8-4c47-ae69-0979bb34993c.metadata.json
│   │                   ├── 00019-f61c3eb0-9723-436e-9011-92d0bc73928a.metadata.json
│   │                   ├── 00020-ea1b2145-b7df-42c8-82c9-536a12ca7b16.metadata.json
│   │                   ├── 00021-443c899e-d49a-48e1-b671-f0e3ce8e1b20.metadata.json
│   │                   ├── 112096d0-1f4f-46b1-9a94-0e5bbff4c558-m0.avro
│   │                   ├── 1f4b7bc2-7116-4d97-a16f-7ad6773d1813-m0.avro
│   │                   ├── 2772adf7-046d-45e6-ad70-617b891fa068-m0.avro
│   │                   ├── 2d39c487-2c61-4165-b61c-f67e849a8efa-m0.avro
│   │                   ├── 4ac4a2b4-07be-4a0b-b799-869746b46d50-m0.avro
│   │                   ├── 62f1f10d-39ec-4254-92a4-3290690daaac-m0.avro
│   │                   ├── 630b89b0-367b-4ed4-8aa5-b9655d4add48-m0.avro
│   │                   ├── 692164e1-ff56-48bd-8f29-1501bfc5a6b0-m0.avro
│   │                   ├── 72c182b8-6c60-4105-ae5a-9b072c7e2b3d-m0.avro
│   │                   ├── 817fee01-3eb0-4273-991a-04f4978d0862-m0.avro
│   │                   ├── 9087b6fc-e47a-4e44-aa9e-77a15c455cac-m0.avro
│   │                   ├── 9d842861-ebb5-4b2d-a7d2-ad8e24af9763-m0.avro
│   │                   ├── a2818425-b79e-48d7-91d1-ed4ae2d9740e-m0.avro
│   │                   ├── b084f226-e85b-4456-9c4b-7fbce5fe9a47-m0.avro
│   │                   ├── b8b9fcd5-1ee8-496f-8c55-28665656840a-m0.avro
│   │                   ├── ba5ec5c6-db31-4137-a015-3c1aedc9ca4a-m0.avro
│   │                   ├── d2d51b18-089e-4020-9b80-6461b32e5753-m0.avro
│   │                   ├── dac3aa04-c4e7-472d-aed8-c6bb00852936-m0.avro
│   │                   ├── e6f83680-b8a6-4afe-9aa6-b0be92fd9131-m0.avro
│   │                   ├── ed00e373-4bd3-447b-88f8-46d6d7259fbb-m0.avro
│   │                   ├── snap-1459279953996412553-0-e6f83680-b8a6-4afe-9aa6-b0be92fd9131.avro
│   │                   ├── snap-1973660318390246770-0-1f4b7bc2-7116-4d97-a16f-7ad6773d1813.avro
│   │                   └── snap-804833600524163404-0-630b89b0-367b-4ed4-8aa5-b9655d4add48.avro
│   └── nb8
│       ├── catalog.db
│       └── warehouse
│           └── lake
│               └── trajectories
│                   ├── data
│                   │   └── 00000-0-cb517006-226c-4e48-8ee3-2fac04fc385b.parquet
│                   └── metadata
│                       ├── 00000-5149c95b-3190-4aa2-9db4-7db85bd73d4b.metadata.json
│                       ├── 00001-08b50ea1-91c5-479e-a19f-f609ac546b5a.metadata.json
│                       ├── cb517006-226c-4e48-8ee3-2fac04fc385b-m0.avro
│                       └── snap-3070913163837874509-0-cb517006-226c-4e48-8ee3-2fac04fc385b.avro
├── scratch
│   ├── customers_tt
│   │   ├── _delta_log
│   │   │   ├── 00000000000000000000.json
│   │   │   ├── 00000000000000000001.json
│   │   │   ├── 00000000000000000002.json
│   │   │   ├── 00000000000000000003.json
│   │   │   └── 00000000000000000004.json
│   │   ├── part-00000-5a3c9248-c8d8-4f5f-b014-b149e824eaed-c000.snappy.parquet
│   │   ├── part-00000-794991ad-3d37-429c-a466-29811bfab656-c000.snappy.parquet
│   │   ├── part-00000-89c12133-da92-4deb-8ae0-ac14ff0fb02b-c000.snappy.parquet
│   │   └── part-00000-a48add20-7ff2-4814-bbcc-51c5290d5233-c000.snappy.parquet
│   ├── docs_cdf
│   │   ├── _change_data
│   │   │   └── part-00000-aa1f3aef-828e-4189-9c01-dca882ec85d0-c000.zstd.parquet
│   │   ├── _delta_log
│   │   │   ├── 00000000000000000000.json
│   │   │   └── 00000000000000000001.json
│   │   ├── part-00000-47def7e3-b485-477b-859e-5d7d55d6d87a-c000.zstd.parquet
│   │   └── part-00000-ec321c32-ce16-49d6-a3c3-6be431c727fc-c000.snappy.parquet
│   ├── docs_intable
│   │   ├── _delta_log
│   │   │   ├── 00000000000000000000.json
│   │   │   └── 00000000000000000001.json
│   │   ├── part-00000-8b82d1f2-5351-4226-ad01-53fd7324be43-c000.zstd.parquet
│   │   └── part-00000-b36ca361-fbb3-4b32-a276-2b90154be3ae-c000.snappy.parquet
│   ├── emb_f32
│   │   ├── _delta_log
│   │   │   └── 00000000000000000000.json
│   │   └── part-00000-1280960b-3202-4838-9f74-56861bb8bbca-c000.snappy.parquet
│   ├── emb_int8
│   │   ├── _delta_log
│   │   │   └── 00000000000000000000.json
│   │   └── part-00000-1d11e9e2-9bf9-40c0-87b0-9adf0f2f77ae-c000.snappy.parquet
│   ├── events_smallfiles
│   │   ├── _delta_log
│   │   │   ├── 00000000000000000000.json
│   │   │   ├── 00000000000000000001.json
│   │   │   ├── 00000000000000000002.json
│   │   │   ├── 00000000000000000003.json
│   │   │   ├── 00000000000000000004.json
│   │   │   ├── 00000000000000000005.json
│   │   │   ├── 00000000000000000006.json
│   │   │   ├── 00000000000000000007.json
│   │   │   ├── 00000000000000000008.json
│   │   │   ├── 00000000000000000009.json
│   │   │   ├── 00000000000000000010.json
│   │   │   ├── 00000000000000000011.json
│   │   │   ├── 00000000000000000012.json
│   │   │   ├── 00000000000000000013.json
│   │   │   ├── 00000000000000000014.json
│   │   │   ├── 00000000000000000015.json
│   │   │   ├── 00000000000000000016.json
│   │   │   ├── 00000000000000000017.json
│   │   │   ├── 00000000000000000018.json
│   │   │   ├── 00000000000000000019.json
│   │   │   ├── 00000000000000000020.json
│   │   │   ├── 00000000000000000021.json
│   │   │   ├── 00000000000000000022.json
│   │   │   ├── 00000000000000000023.json
│   │   │   ├── 00000000000000000024.json
│   │   │   ├── 00000000000000000025.json
│   │   │   ├── 00000000000000000026.json
│   │   │   ├── 00000000000000000027.json
│   │   │   ├── 00000000000000000028.json
│   │   │   ├── 00000000000000000029.json
│   │   │   ├── 00000000000000000030.json
│   │   │   ├── 00000000000000000031.json
│   │   │   ├── 00000000000000000032.json
│   │   │   ├── 00000000000000000033.json
│   │   │   ├── 00000000000000000034.json
│   │   │   ├── 00000000000000000035.json
│   │   │   ├── 00000000000000000036.json
│   │   │   ├── 00000000000000000037.json
│   │   │   ├── 00000000000000000038.json
│   │   │   ├── 00000000000000000039.json
│   │   │   ├── 00000000000000000040.json
│   │   │   ├── 00000000000000000041.json
│   │   │   ├── 00000000000000000042.json
│   │   │   ├── 00000000000000000043.json
│   │   │   ├── 00000000000000000044.json
│   │   │   ├── 00000000000000000045.json
│   │   │   ├── 00000000000000000046.json
│   │   │   ├── 00000000000000000047.json
│   │   │   ├── 00000000000000000048.json
│   │   │   ├── 00000000000000000049.json
│   │   │   ├── 00000000000000000050.json
│   │   │   ├── 00000000000000000051.json
│   │   │   ├── 00000000000000000052.json
│   │   │   ├── 00000000000000000053.json
│   │   │   ├── 00000000000000000054.json
│   │   │   ├── 00000000000000000055.json
│   │   │   ├── 00000000000000000056.json
│   │   │   ├── 00000000000000000057.json
│   │   │   ├── 00000000000000000058.json
│   │   │   ├── 00000000000000000059.json
│   │   │   ├── 00000000000000000060.json
│   │   │   ├── 00000000000000000061.json
│   │   │   ├── 00000000000000000062.json
│   │   │   ├── 00000000000000000063.json
│   │   │   ├── 00000000000000000064.json
│   │   │   ├── 00000000000000000065.json
│   │   │   ├── 00000000000000000066.json
│   │   │   ├── 00000000000000000067.json
│   │   │   ├── 00000000000000000068.json
│   │   │   ├── 00000000000000000069.json
│   │   │   ├── 00000000000000000070.json
│   │   │   ├── 00000000000000000071.json
│   │   │   ├── 00000000000000000072.json
│   │   │   ├── 00000000000000000073.json
│   │   │   ├── 00000000000000000074.json
│   │   │   ├── 00000000000000000075.json
│   │   │   ├── 00000000000000000076.json
│   │   │   ├── 00000000000000000077.json
│   │   │   ├── 00000000000000000078.json
│   │   │   ├── 00000000000000000079.json
│   │   │   ├── 00000000000000000080.json
│   │   │   ├── 00000000000000000081.json
│   │   │   ├── 00000000000000000082.json
│   │   │   ├── 00000000000000000083.json
│   │   │   ├── 00000000000000000084.json
│   │   │   ├── 00000000000000000085.json
│   │   │   ├── 00000000000000000086.json
│   │   │   ├── 00000000000000000087.json
│   │   │   ├── 00000000000000000088.json
│   │   │   ├── 00000000000000000089.json
│   │   │   ├── 00000000000000000090.json
│   │   │   ├── 00000000000000000091.json
│   │   │   ├── 00000000000000000092.json
│   │   │   ├── 00000000000000000093.json
│   │   │   ├── 00000000000000000094.json
│   │   │   ├── 00000000000000000095.json
│   │   │   ├── 00000000000000000096.json
│   │   │   ├── 00000000000000000097.json
│   │   │   ├── 00000000000000000098.json
│   │   │   ├── 00000000000000000099.checkpoint.parquet
│   │   │   ├── 00000000000000000099.json
│   │   │   ├── 00000000000000000100.json
│   │   │   ├── 00000000000000000101.json
│   │   │   ├── 00000000000000000102.json
│   │   │   ├── 00000000000000000103.json
│   │   │   ├── 00000000000000000104.json
│   │   │   ├── 00000000000000000105.json
│   │   │   ├── 00000000000000000106.json
│   │   │   ├── 00000000000000000107.json
│   │   │   ├── 00000000000000000108.json
│   │   │   ├── 00000000000000000109.json
│   │   │   ├── 00000000000000000110.json
│   │   │   ├── 00000000000000000111.json
│   │   │   ├── 00000000000000000112.json
│   │   │   ├── 00000000000000000113.json
│   │   │   ├── 00000000000000000114.json
│   │   │   ├── 00000000000000000115.json
│   │   │   ├── 00000000000000000116.json
│   │   │   ├── 00000000000000000117.json
│   │   │   ├── 00000000000000000118.json
│   │   │   ├── 00000000000000000119.json
│   │   │   ├── 00000000000000000120.json
│   │   │   ├── 00000000000000000121.json
│   │   │   ├── 00000000000000000122.json
│   │   │   ├── 00000000000000000123.json
│   │   │   ├── 00000000000000000124.json
│   │   │   ├── 00000000000000000125.json
│   │   │   ├── 00000000000000000126.json
│   │   │   ├── 00000000000000000127.json
│   │   │   ├── 00000000000000000128.json
│   │   │   ├── 00000000000000000129.json
│   │   │   ├── 00000000000000000130.json
│   │   │   ├── 00000000000000000131.json
│   │   │   ├── 00000000000000000132.json
│   │   │   ├── 00000000000000000133.json
│   │   │   ├── 00000000000000000134.json
│   │   │   ├── 00000000000000000135.json
│   │   │   ├── 00000000000000000136.json
│   │   │   ├── 00000000000000000137.json
│   │   │   ├── 00000000000000000138.json
│   │   │   ├── 00000000000000000139.json
│   │   │   ├── 00000000000000000140.json
│   │   │   ├── 00000000000000000141.json
│   │   │   ├── 00000000000000000142.json
│   │   │   ├── 00000000000000000143.json
│   │   │   ├── 00000000000000000144.json
│   │   │   ├── 00000000000000000145.json
│   │   │   ├── 00000000000000000146.json
│   │   │   ├── 00000000000000000147.json
│   │   │   ├── 00000000000000000148.json
│   │   │   ├── 00000000000000000149.json
│   │   │   ├── 00000000000000000150.json
│   │   │   ├── 00000000000000000151.json
│   │   │   ├── 00000000000000000152.json
│   │   │   ├── 00000000000000000153.json
│   │   │   ├── 00000000000000000154.json
│   │   │   ├── 00000000000000000155.json
│   │   │   ├── 00000000000000000156.json
│   │   │   ├── 00000000000000000157.json
│   │   │   ├── 00000000000000000158.json
│   │   │   ├── 00000000000000000159.json
│   │   │   ├── 00000000000000000160.json
│   │   │   ├── 00000000000000000161.json
│   │   │   ├── 00000000000000000162.json
│   │   │   ├── 00000000000000000163.json
│   │   │   ├── 00000000000000000164.json
│   │   │   ├── 00000000000000000165.json
│   │   │   ├── 00000000000000000166.json
│   │   │   ├── 00000000000000000167.json
│   │   │   ├── 00000000000000000168.json
│   │   │   ├── 00000000000000000169.json
│   │   │   ├── 00000000000000000170.json
│   │   │   ├── 00000000000000000171.json
│   │   │   ├── 00000000000000000172.json
│   │   │   ├── 00000000000000000173.json
│   │   │   ├── 00000000000000000174.json
│   │   │   ├── 00000000000000000175.json
│   │   │   ├── 00000000000000000176.json
│   │   │   ├── 00000000000000000177.json
│   │   │   ├── 00000000000000000178.json
│   │   │   ├── 00000000000000000179.json
│   │   │   ├── 00000000000000000180.json
│   │   │   ├── 00000000000000000181.json
│   │   │   ├── 00000000000000000182.json
│   │   │   ├── 00000000000000000183.json
│   │   │   ├── 00000000000000000184.json
│   │   │   ├── 00000000000000000185.json
│   │   │   ├── 00000000000000000186.json
│   │   │   ├── 00000000000000000187.json
│   │   │   ├── 00000000000000000188.json
│   │   │   ├── 00000000000000000189.json
│   │   │   ├── 00000000000000000190.json
│   │   │   ├── 00000000000000000191.json
│   │   │   ├── 00000000000000000192.json
│   │   │   ├── 00000000000000000193.json
│   │   │   ├── 00000000000000000194.json
│   │   │   ├── 00000000000000000195.json
│   │   │   ├── 00000000000000000196.json
│   │   │   ├── 00000000000000000197.json
│   │   │   ├── 00000000000000000198.json
│   │   │   ├── 00000000000000000199.checkpoint.parquet
│   │   │   ├── 00000000000000000199.json
│   │   │   ├── 00000000000000000200.json
│   │   │   ├── 00000000000000000201.json
│   │   │   ├── 00000000000000000202.json
│   │   │   ├── 00000000000000000203.checkpoint.parquet
│   │   │   ├── 00000000000000000203.json
│   │   │   └── _last_checkpoint
│   │   ├── part-00000-2ee30389-35e8-47f7-abe9-e5735fa1df93-c000.zstd.parquet
│   │   ├── part-00001-2ee30389-35e8-47f7-abe9-e5735fa1df93-c000.zstd.parquet
│   │   ├── part-00002-2ee30389-35e8-47f7-abe9-e5735fa1df93-c000.zstd.parquet
│   │   ├── part-00003-2ee30389-35e8-47f7-abe9-e5735fa1df93-c000.zstd.parquet
│   │   ├── part-00004-2ee30389-35e8-47f7-abe9-e5735fa1df93-c000.zstd.parquet
│   │   ├── part-00005-2ee30389-35e8-47f7-abe9-e5735fa1df93-c000.zstd.parquet
│   │   ├── part-00006-2ee30389-35e8-47f7-abe9-e5735fa1df93-c000.zstd.parquet
│   │   ├── part-00007-2ee30389-35e8-47f7-abe9-e5735fa1df93-c000.zstd.parquet
│   │   ├── part-00008-2ee30389-35e8-47f7-abe9-e5735fa1df93-c000.zstd.parquet
│   │   └── part-00009-2ee30389-35e8-47f7-abe9-e5735fa1df93-c000.zstd.parquet
│   ├── media_inline
│   │   ├── _delta_log
│   │   │   └── 00000000000000000000.json
│   │   └── part-00000-98286f9a-d87c-4c9f-8ab6-2574b3cf06d2-c000.snappy.parquet
│   ├── media_pointer
│   │   ├── _delta_log
│   │   │   └── 00000000000000000000.json
│   │   └── part-00000-ec8c4c0a-2140-4eea-ac38-c1c403c77104-c000.snappy.parquet
│   ├── users_delta
│   │   ├── _delta_log
│   │   │   ├── 00000000000000000000.json
│   │   │   └── 00000000000000000001.json
│   │   ├── part-00000-990587b0-9522-406e-83cb-e11d092e382a-c000.snappy.parquet
│   │   └── part-00000-b12851b1-c938-417d-9755-fe0b905d7299-c000.snappy.parquet
│   └── vector_index_external
│       ├── _delta_log
│       │   └── 00000000000000000000.json
│       └── part-00000-c5e80dec-8693-456a-9008-85d8be7a5285-c000.snappy.parquet
└── silver
    ├── agent_trajectories
    │   ├── _delta_log
    │   │   ├── 00000000000000000000.json
    │   │   └── 00000000000000000001.json
    │   ├── agent_version=policy-v2
    │   │   ├── part-00000-211e9daa-592e-4b8c-8498-847bfd7d833c-c000.snappy.parquet
    │   │   └── part-00000-31ad7148-d837-41cd-92bd-5787497f934f-c000.snappy.parquet
    │   └── agent_version=policy-v3
    │       └── part-00000-4e237fce-058a-435d-ba3f-ae0e7caa211b-c000.snappy.parquet
    ├── llm_calls
    │   ├── _delta_log
    │   │   └── 00000000000000000000.json
    │   ├── date=2026-04-01
    │   │   └── part-00000-3d64c52f-8ecb-4f27-96ec-451e6cad522f-c000.snappy.parquet
    │   ├── date=2026-04-02
    │   │   └── part-00000-2412d400-6e56-4d17-80c8-9bddab4e021c-c000.snappy.parquet
    │   ├── date=2026-04-03
    │   │   └── part-00000-3ee8ef76-c454-49d0-95ad-cae9a057fe54-c000.snappy.parquet
    │   ├── date=2026-04-04
    │   │   └── part-00000-27c6b587-44ea-49fc-86bf-0fbfd498e498-c000.snappy.parquet
    │   ├── date=2026-04-05
    │   │   └── part-00000-bc703acb-3498-4eff-a0fe-986fa24b9cf7-c000.snappy.parquet
    │   ├── date=2026-04-06
    │   │   └── part-00000-63aa899e-dccf-47b7-aced-db4305003579-c000.snappy.parquet
    │   ├── date=2026-04-07
    │   │   └── part-00000-9bf4c7ca-2a4f-42b1-b8b7-4613adcbb9ea-c000.snappy.parquet
    │   └── date=2026-04-08
    │       └── part-00000-e081f606-0eab-4551-ab12-f9f3bf7a5c50-c000.snappy.parquet
    └── training_corpus_governed
        ├── _delta_log
        │   ├── 00000000000000000000.json
        │   └── 00000000000000000001.json
        ├── provenance_bucket=UNCLASSIFIED
        │   ├── part-00000-15d04b3b-7d2a-4080-bb5e-0761fb7f747b-c000.zstd.parquet
        │   └── part-00000-53887e50-f220-48af-9889-236709200afe-c000.snappy.parquet
        ├── provenance_bucket=licensed
        │   ├── part-00000-69ea2368-ca4c-48f4-8132-58d073048f39-c000.zstd.parquet
        │   └── part-00000-d3e6f586-d7c2-4bbc-8b77-6f82240ef82c-c000.snappy.parquet
        ├── provenance_bucket=public_domain
        │   └── part-00000-c6e1e44e-116b-4972-baf0-da9ea0f4158c-c000.snappy.parquet
        ├── provenance_bucket=scraped_optout_checked
        │   ├── part-00000-25a84432-b18c-4775-8d1b-8a28326d0d5e-c000.zstd.parquet
        │   └── part-00000-e9a96409-e8d7-40cc-ae86-2b7d1c117f5e-c000.snappy.parquet
        └── provenance_bucket=synthetic
            ├── part-00000-25c023e8-4a8c-4e74-b8e6-77a48afc0a10-c000.zstd.parquet
            └── part-00000-2fba6f24-9874-4fcc-8ce8-eb77a923a26d-c000.snappy.parquet

101 directories, 1152 files
```

</details>

---

## 2. Chi Tiết File Transaction Log (`_delta_log/00000000000000000000.json`)

Dưới đây là toàn bộ nội dung file commit đầu tiên (`version 0`) của bảng `scratch/users_delta`:

```json
{"commitInfo":{"timestamp":1787021514708,"operation":"WRITE","operationParameters":{"mode":"Overwrite"},"engineInfo":"delta-rs:py-1.6.2","operationMetrics":{"execution_time_ms":2,"num_added_files":1,"num_added_rows":3,"num_partitions":0,"num_removed_files":0},"clientVersion":"delta-rs.py-1.6.2"}}
{"protocol":{"minReaderVersion":1,"minWriterVersion":2}}
{"metaData":{"id":"e9848eb4-7f63-4191-b5dc-4c276df8b20e","name":null,"description":null,"format":{"provider":"parquet","options":{}},"schemaString":"{\"type\":\"struct\",\"fields\":[{\"name\":\"id\",\"type\":\"long\",\"nullable\":true,\"metadata\":{}},{\"name\":\"name\",\"type\":\"string\",\"nullable\":true,\"metadata\":{}},{\"name\":\"age\",\"type\":\"long\",\"nullable\":true,\"metadata\":{}},{\"name\":\"city\",\"type\":\"string\",\"nullable\":true,\"metadata\":{}}]}","partitionColumns":[],"createdTime":1787021514706,"configuration":{}}}
{"add":{"path":"part-00000-990587b0-9522-406e-83cb-e11d092e382a-c000.snappy.parquet","partitionValues":{},"size":1384,"modificationTime":1787021514708,"dataChange":true,"stats":"{\"numRecords\":3,\"minValues\":{\"city\":\"Danang\",\"age\":25,\"name\":\"alice\",\"id\":1},\"maxValues\":{\"name\":\"charlie\",\"id\":3,\"city\":\"Hanoi\",\"age\":35},\"nullCount\":{\"name\":0,\"age\":0,\"city\":0,\"id\":0}}","tags":null,"baseRowId":null,"defaultRowCommitVersion":null,"clusteringProvider":null}}
```

### 🔬 Phân Tích & Giải Thích Ý Nghĩa Từng Dòng JSON:

1. **Dòng 1 — `commitInfo` (Thông tin giao dịch)**:
   * `operation: "WRITE"`: Thao tác thực hiện là ghi dữ liệu mới.
   * `operationParameters: {"mode": "Overwrite"}`: Ghi đè toàn bộ dữ liệu của phiên bản trước.
   * `operationMetrics`: Chứa các chỉ số đo lường hiệu năng (`execution_time_ms: 2`, ghi thêm 1 file `num_added_files: 1`, nạp 3 bản ghi `num_added_rows: 3`).

2. **Dòng 2 — `protocol` (Giao thức Delta Lake)**:
   * Quy định phiên bản tối thiểu mà client cần hỗ trợ để đọc (`minReaderVersion: 1`) và ghi (`minWriterVersion: 2`).

3. **Dòng 3 — `metaData` (Định nghĩa Schema & Bảng)**:
   * `id`: Định danh duy nhất dạng UUID của bảng.
   * `schemaString`: Định nghĩa chính xác cấu trúc kiểu dữ liệu (`id: long`, `name: string`, `age: long`, `city: string`). Đây chính là căn cứ để Delta Lake thực hiện **Schema Enforcement** nhằm chặn đứng các bản ghi sai kiểu dữ liệu (ví dụ: `age="thirty"`).

4. **Dòng 4 — Hành động `add` (Thêm Data File Parquet)**:
   * `path`: Tên file dữ liệu Parquet thực tế trên đĩa (`part-00000-...snappy.parquet`).
   * `size: 1384`: Dung lượng file tính bằng bytes.
   * `stats`: Chứa thống kê cốt lõi gồm số dòng (`numRecords: 3`), giá trị nhỏ nhất `minValues` (`age: 25, id: 1, city: Danang`) và lớn nhất `maxValues` (`age: 35, id: 3, city: Hanoi`).
   * **Tầm quan trọng của `stats`**: Khi có truy vấn `WHERE age > 40`, công cụ truy vấn chỉ cần đọc file JSON này, thấy `maxValues.age = 35 < 40` là **tự động bỏ qua (skip/prune)** file Parquet này mà không cần tốn chi phí đọc đĩa (I/O).

---

## 3. Các Lệnh Tự Kiểm Chứng Trên Terminal:

```bash
# Xem cây thư mục lưu trữ
tree _lakehouse/

# Kiểm tra nội dung transaction log
cat _lakehouse/scratch/users_delta/_delta_log/00000000000000000000.json
```
