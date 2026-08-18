# Tài Liệu Minh Chứng Cấu Trúc Lưu Trữ & Transaction Log Lakehouse

Tài liệu này cung cấp đầy đủ các minh chứng kỹ thuật phục vụ việc chấm điểm theo [rubric.md](../../rubric.md):
1. **Cấu trúc lưu trữ vật lý trên đĩa (`_lakehouse/`)** bao gồm các tầng Bronze, Silver, Gold, Scratch và Iceberg Catalog.
2. **Chi tiết cấu trúc và giải thích Transaction Log của Delta Lake (`_delta_log/00000000000000000000.json`)**.

---

## 1. Cấu Trúc Lưu Trữ Vật Lý Của Lakehouse (`_lakehouse/`)

Cây thư mục dưới đây thể hiện toàn bộ dữ liệu thực tế được sinh ra sau khi hoàn thành 8 Notebooks:

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
│   └── llm_calls/                # 190,052 dòng đã loại bỏ trùng lặp (dedup), phân vùng theo date (NB4)
├── gold/                         # TẦNG GOLD: Dữ liệu tổng hợp KPI phục vụ trực tiếp cho báo cáo và AI Training
│   ├── agent_policy_gold/        # Tổng hợp reward và số bước trung bình theo từng chính sách agent (NB8)
│   └── llm_daily_metrics/        # Chỉ số KPI hàng ngày: Latency p50/p95, Cost USD, Error Rate (8 ngày × 3 model) (NB4)
├── iceberg/                      # APACHE ICEBERG CATALOGS: Control Plane quản lý metadata
│   ├── nb5_catalog.db            # Catalog SQLite cho NB5 (Hidden Partitioning & Schema Evolution)
│   ├── nb6_catalog.db            # Catalog SQLite cho NB6 (Quản lý Snapshot Expiry & Maintenance)
│   └── nb8_catalog.db            # Catalog SQLite cho NB8 (Phân vùng theo EU AI Act Art. 10)
└── scratch/                      # VÙNG THỰC NGHIỆM ĐO ĐẠC: Các bảng phục vụ benchmark
    ├── customers_tt/             # Bảng kiểm chứng Time Travel (5 versions), Upsert MERGE và RESTORE (NB3)
    ├── events_smallfiles/        # Bảng tái hiện vấn đề 200 small-files và đo lường Z-ORDER (NB2)
    ├── maintenance_delta/        # Bảng thực nghiệm dọn dẹp VACUUM và bẫy Orphan Files (NB6)
    └── users_delta/              # Bảng kiểm chứng Schema Enforcement & Schema Evolution (NB1)
```


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
│   │   │   └── _last_checkpoint
│   │   ├── part-00000-00daf0b3-493d-4dd1-b260-bae270cb7465-c000.snappy.parquet
│   │   ├── part-00000-010424c0-6158-4129-89bc-79999ca5d4a7-c000.snappy.parquet
│   │   ├── part-00000-01a4f1f0-73df-4683-8151-fa01aa6db232-c000.snappy.parquet
│   │   ├── part-00000-03ce3571-0e23-42ff-b286-ce6d92521823-c000.zstd.parquet
│   │   ├── part-00000-05b1546d-c6d0-4f85-b904-d2a7c6823ff7-c000.snappy.parquet
│   │   ├── part-00000-05b6f6a7-76e3-4944-a0fa-0c128c6ccb6b-c000.snappy.parquet
│   │   ├── part-00000-05d9da2e-ab14-4907-bb92-fa3e7b26a771-c000.zstd.parquet
│   │   ├── part-00000-06bebb5b-39f3-4130-9682-007cf5a806a1-c000.snappy.parquet
│   │   ├── part-00000-06fe1f78-cbe9-4987-aff5-b5afeef567d9-c000.snappy.parquet
│   │   ├── part-00000-07a33040-2a98-4d92-b8fc-eeafb473e83f-c000.snappy.parquet
│   │   ├── part-00000-08517d06-c1c1-4602-9bf9-26f3040df550-c000.snappy.parquet
│   │   ├── part-00000-094c07cc-ea81-4f10-b13d-fa4a9a56e301-c000.snappy.parquet
│   │   ├── part-00000-0a7e203b-4df8-448a-a946-bdd155440d87-c000.snappy.parquet
│   │   ├── part-00000-0a9b4be2-f556-4e79-8855-ab6a41dff7ea-c000.snappy.parquet
│   │   ├── part-00000-0b35083e-8c50-4482-a281-bb2cce530541-c000.snappy.parquet
│   │   ├── part-00000-0d7b1576-12b1-4762-b563-776289d85241-c000.snappy.parquet
│   │   ├── part-00000-0eb9cf84-f093-41df-8735-35b8c79f4998-c000.snappy.parquet
│   │   ├── part-00000-0f06ec8c-c377-4caf-aa54-477b7c816aa1-c000.snappy.parquet
│   │   ├── part-00000-0f7c4ad2-3803-4d7b-a36a-c2c8ba1e4b04-c000.snappy.parquet
│   │   ├── part-00000-11aa1e6b-bfbf-4ad9-9f01-0882698046e4-c000.zstd.parquet
│   │   ├── part-00000-1210a194-b30a-49fd-9b69-6b554d44abe8-c000.zstd.parquet
│   │   ├── part-00000-12bbfeb6-1854-4880-865e-02921af50406-c000.snappy.parquet
│   │   ├── part-00000-13df259f-dc90-4072-a69d-27fa56b24e53-c000.snappy.parquet
│   │   ├── part-00000-14d4930d-fbd3-4cc8-aa22-bb94f371a9e8-c000.zstd.parquet
│   │   ├── part-00000-1532b67e-3efd-481f-948f-b2fb7a24db33-c000.snappy.parquet
│   │   ├── part-00000-169ae3ea-f678-431b-9be7-6368b016ef8b-c000.snappy.parquet
│   │   ├── part-00000-16fb795c-76b6-4f1e-b91b-a3b3ea320bd2-c000.snappy.parquet
│   │   ├── part-00000-173dc634-ce11-44ed-be7d-d71f5d80d9b9-c000.snappy.parquet
│   │   ├── part-00000-17c8fdf2-614e-4e37-a78a-494941c891e6-c000.zstd.parquet
│   │   ├── part-00000-191dde55-c626-4933-87ca-e6920da2a189-c000.snappy.parquet
│   │   ├── part-00000-197e2047-33d8-40a4-8f97-e39dccb044c2-c000.snappy.parquet
│   │   ├── part-00000-1adfbf37-2e00-432a-83ba-46080c6f28bc-c000.snappy.parquet
│   │   ├── part-00000-1c43acb0-51eb-41f3-b465-c965ed1e3ac8-c000.snappy.parquet
│   │   ├── part-00000-1e216684-b82d-4a47-84a6-8efcedecef51-c000.snappy.parquet
│   │   ├── part-00000-1e98581f-625e-404f-9460-a34fcbb22dcb-c000.snappy.parquet
│   │   ├── part-00000-1ef7a74b-3689-4163-846d-3bc4898d8c2d-c000.snappy.parquet
│   │   ├── part-00000-2010eba1-6ff3-4eb8-a768-dc2c04a553aa-c000.zstd.parquet
│   │   ├── part-00000-20d984fd-a4ed-4da9-8c0d-c7c2d9f33e6f-c000.snappy.parquet
│   │   ├── part-00000-21a0054b-13a1-43b9-8a98-b3f2dee7672d-c000.zstd.parquet
│   │   ├── part-00000-22a3b96e-4564-46f9-be1c-195e0657bfaf-c000.zstd.parquet
│   │   ├── part-00000-23ea1ef9-f280-48db-be31-6c8d4cf279d0-c000.snappy.parquet
│   │   ├── part-00000-24d162d6-2e83-48ea-94b2-e4ab426d5458-c000.zstd.parquet
│   │   ├── part-00000-257d0ed9-f53f-4a06-bbc3-bff1bd6e2741-c000.snappy.parquet
│   │   ├── part-00000-265fe74e-bf8b-445d-a29b-fbc378c6381b-c000.snappy.parquet
│   │   ├── part-00000-269ee5c6-6d63-4bd0-bb04-89d7a843edea-c000.snappy.parquet
│   │   ├── part-00000-26b06a0e-86b4-4774-b5a2-f89400619b1c-c000.snappy.parquet
│   │   ├── part-00000-26b0c193-8a8c-4174-a5ee-0a96a03fec2f-c000.zstd.parquet
│   │   ├── part-00000-26c3d233-82f1-4409-9774-6cc8e7f76936-c000.snappy.parquet
│   │   ├── part-00000-27238852-867b-4f95-8525-13818df3e199-c000.snappy.parquet
│   │   ├── part-00000-28124b2f-7b06-4900-98ba-e5ecdb6431fd-c000.snappy.parquet
│   │   ├── part-00000-287fd427-83ee-4a68-a84e-be7e9bd4ad1b-c000.snappy.parquet
│   │   ├── part-00000-28826c61-99fc-4a57-860d-6fff7f0155c0-c000.snappy.parquet
│   │   ├── part-00000-28d29731-3076-4f59-8873-e283973ec83a-c000.zstd.parquet
│   │   ├── part-00000-291a2517-1b43-497a-9e67-d4e0ab09e2c7-c000.snappy.parquet
│   │   ├── part-00000-2931813b-80d0-40c4-92a2-ae939bc97312-c000.snappy.parquet
│   │   ├── part-00000-29c3e3e8-1604-4717-93d0-d1a6ecbf5247-c000.snappy.parquet
│   │   ├── part-00000-29f75539-0227-4821-b185-e94c1f9cfde9-c000.zstd.parquet
│   │   ├── part-00000-2a70cbeb-b0b0-4fe7-a267-3428c35c9514-c000.zstd.parquet
│   │   ├── part-00000-2b1d15a6-ffb4-40de-8ce1-53d6f9401673-c000.snappy.parquet
│   │   ├── part-00000-2b56d333-df40-4bdf-baf0-b43bb3d5ead3-c000.snappy.parquet
│   │   ├── part-00000-2b970046-fed1-4131-9f2c-de4def861dc2-c000.snappy.parquet
│   │   ├── part-00000-2c566ae0-39bc-4ce2-aa6b-f77abd7e2d44-c000.snappy.parquet
│   │   ├── part-00000-2ded58a0-7c47-40fb-9da0-4e75dac9de7c-c000.snappy.parquet
│   │   ├── part-00000-2ec92660-134f-4b8a-b9d7-f78c7dc67f8c-c000.snappy.parquet
│   │   ├── part-00000-3020a6b5-2b5e-4cbe-85ca-319948e70001-c000.snappy.parquet
│   │   ├── part-00000-31f146cf-5645-4870-87a9-1ff39c663c8c-c000.snappy.parquet
│   │   ├── part-00000-33cfd2a3-8b54-43eb-a25c-93579a8b7e52-c000.snappy.parquet
│   │   ├── part-00000-3438bb1c-484e-4bfc-9e53-49c6dd8f474f-c000.snappy.parquet
│   │   ├── part-00000-346e1786-74a9-4196-8572-6853f047933c-c000.snappy.parquet
│   │   ├── part-00000-34be6b10-a1b7-4fb5-a806-a4600404054b-c000.zstd.parquet
│   │   ├── part-00000-3501f157-252c-4436-ac9c-3519490501f5-c000.snappy.parquet
│   │   ├── part-00000-3754c9b5-46f1-4f42-9b87-f627d8450d9f-c000.snappy.parquet
│   │   ├── part-00000-375a8330-82fe-4d0f-9489-5f614342ce53-c000.zstd.parquet
│   │   ├── part-00000-38a19673-824c-47fb-a34d-a86351fb1ef4-c000.snappy.parquet
│   │   ├── part-00000-38c6e079-d55e-4884-ac15-6f8db45fe8c7-c000.zstd.parquet
│   │   ├── part-00000-3981a5b3-2eed-4e61-8405-97b0286d8b5a-c000.snappy.parquet
│   │   ├── part-00000-3a39e20a-7040-4a42-b1b5-46dbed81fbad-c000.snappy.parquet
│   │   ├── part-00000-3d09fcd3-671c-4e7a-a0d4-741088b0010b-c000.snappy.parquet
│   │   ├── part-00000-3d53acda-8c9b-4d69-8a00-611bb940c0d2-c000.zstd.parquet
│   │   ├── part-00000-3d580a41-e409-452c-9ec2-87d09414c42d-c000.snappy.parquet
│   │   ├── part-00000-3d602d44-daa3-444b-a093-44cdbe722072-c000.zstd.parquet
│   │   ├── part-00000-3ddac927-f76a-48f8-bec5-7d5b612f4024-c000.snappy.parquet
│   │   ├── part-00000-3ebfb50a-68a9-46db-b3b9-878f577b20f7-c000.snappy.parquet
│   │   ├── part-00000-3f553a93-a9a3-4566-9eff-a9bbe8ad0d78-c000.snappy.parquet
│   │   ├── part-00000-4023ed2d-7729-45a1-ae95-aeb77d6b459d-c000.snappy.parquet
│   │   ├── part-00000-40d441ec-19cd-4ce1-985b-a522e170e006-c000.snappy.parquet
│   │   ├── part-00000-41509fc1-0ace-4ddb-a85c-efff1d73920b-c000.snappy.parquet
│   │   ├── part-00000-419b43c1-11bf-46ff-97a3-bcb1086b80e5-c000.snappy.parquet
│   │   ├── part-00000-41b0f2e9-48f1-463c-8adf-8aa847025395-c000.zstd.parquet
│   │   ├── part-00000-42131f0f-0148-40a4-aec4-8a461a8f9b47-c000.snappy.parquet
│   │   ├── part-00000-424ee637-c9fe-43d6-a4e4-bcf1a24d8796-c000.snappy.parquet
│   │   ├── part-00000-42af8198-846c-4b18-88a0-d8a60cd7b5f8-c000.snappy.parquet
│   │   ├── part-00000-4414422b-23e6-4cc9-9e4d-f696685b8cbc-c000.snappy.parquet
│   │   ├── part-00000-44353863-0004-4ccd-8016-05ae67ad164f-c000.snappy.parquet
│   │   ├── part-00000-4574f698-5313-46a2-a161-9b34a4851afb-c000.snappy.parquet
│   │   ├── part-00000-476798da-382b-4423-a837-5957ffbe368d-c000.snappy.parquet
│   │   ├── part-00000-479a470e-8a01-4699-94ac-9dfe000ab68a-c000.zstd.parquet
│   │   ├── part-00000-47a45369-1933-43c9-9bbd-f8023b1f7b9a-c000.zstd.parquet
│   │   ├── part-00000-47c94033-a7b2-4f65-a0f0-245cdb51ef28-c000.snappy.parquet
│   │   ├── part-00000-48ea047a-c42a-4339-8caa-ea06969dd1be-c000.snappy.parquet
│   │   ├── part-00000-48ed773b-45dc-4a9f-a01c-cdfd7e9074d7-c000.snappy.parquet
│   │   ├── part-00000-49ad33b8-7792-4a7e-97e6-009641d10a9a-c000.snappy.parquet
│   │   ├── part-00000-4b06a45f-b93f-4e10-b080-795a5c9e78f6-c000.zstd.parquet
│   │   ├── part-00000-4cb2e734-03ef-474c-bf7b-6dcde47149a6-c000.snappy.parquet
│   │   ├── part-00000-4dfe0ea8-9125-4031-8ab5-2005cf981685-c000.snappy.parquet
│   │   ├── part-00000-4e0637b3-da2e-466d-951b-118a211b0eb4-c000.zstd.parquet
│   │   ├── part-00000-4e913854-0fba-484b-ba10-6d98abcdc880-c000.snappy.parquet
│   │   ├── part-00000-50b6a062-d1a1-4d56-ac88-0c2c5e1a41d9-c000.snappy.parquet
│   │   ├── part-00000-50d57c6a-da90-4002-8715-53857ba448a3-c000.snappy.parquet
│   │   ├── part-00000-513974dc-3669-45c0-bc0c-50751eb0144b-c000.zstd.parquet
│   │   ├── part-00000-52a628cc-cc51-4583-919a-91008002a744-c000.snappy.parquet
│   │   ├── part-00000-52ea1ec2-deb3-4a51-ac06-582cd4b74b29-c000.zstd.parquet
│   │   ├── part-00000-52eeaab1-dee1-41bd-91e7-ac182f77748b-c000.snappy.parquet
│   │   ├── part-00000-53f89c4b-086b-427a-b113-8f83ca756cf7-c000.snappy.parquet
│   │   ├── part-00000-54ed8736-348a-41af-aa15-dc9412dcce8b-c000.snappy.parquet
│   │   ├── part-00000-56841b71-cd48-4e93-96b3-3eb2966cede6-c000.snappy.parquet
│   │   ├── part-00000-5875d22b-b4df-4212-a52f-189ba2446168-c000.snappy.parquet
│   │   ├── part-00000-5a446c7b-a630-4d85-87c5-a0a817da7e90-c000.zstd.parquet
│   │   ├── part-00000-5d3b292c-bdbe-4304-b513-d5c0891749e2-c000.snappy.parquet
│   │   ├── part-00000-603898b6-eb4f-4d9d-96ac-af13b7417292-c000.snappy.parquet
│   │   ├── part-00000-60b5722c-6440-49ca-88dc-ee971601590d-c000.zstd.parquet
│   │   ├── part-00000-616a47fb-9676-4e24-a868-a7cf66e65b52-c000.snappy.parquet
│   │   ├── part-00000-61d2ac8b-a200-4b60-8526-58ea5ebdba4e-c000.snappy.parquet
│   │   ├── part-00000-63c3a2ae-d8dd-45ff-8cfc-d2fa981944d8-c000.snappy.parquet
│   │   ├── part-00000-63ccacc5-f340-4594-bcb6-e8d3228ba052-c000.zstd.parquet
│   │   ├── part-00000-6481b8f7-136e-4ef9-8e97-aa3d1dd688de-c000.snappy.parquet
│   │   ├── part-00000-6694d11b-b2f0-4a8e-9c31-d5700d586b1c-c000.snappy.parquet
│   │   ├── part-00000-66a721f6-84cb-4435-812f-6460f0851ffb-c000.snappy.parquet
│   │   ├── part-00000-66b56976-3e9a-4580-81f1-923bfad8e0cf-c000.zstd.parquet
│   │   ├── part-00000-687313b0-f09b-479c-a832-34f091c83384-c000.snappy.parquet
│   │   ├── part-00000-68dc9a86-8a07-4ca4-bc9a-165ec8d66e2f-c000.zstd.parquet
│   │   ├── part-00000-692c800f-d772-4870-80ea-000c071d2d7e-c000.zstd.parquet
│   │   ├── part-00000-69fbdc68-893f-48c8-8e91-0ad558e64c8b-c000.snappy.parquet
│   │   ├── part-00000-6a5aa2a4-f0af-4b4e-ad40-e6b0705791fa-c000.zstd.parquet
│   │   ├── part-00000-6a8b5696-e8ec-422c-bccf-b71ffca932a3-c000.zstd.parquet
│   │   ├── part-00000-6b671a2f-0651-4e93-9cff-c61096f0786f-c000.snappy.parquet
│   │   ├── part-00000-6dd98419-429f-423b-b6db-664d62b2dd51-c000.zstd.parquet
│   │   ├── part-00000-6ea0d9da-3942-4f87-91bc-3e07ca0547e8-c000.snappy.parquet
│   │   ├── part-00000-6eace636-e540-42c4-8acc-e472f3183ed1-c000.snappy.parquet
│   │   ├── part-00000-6eb7a3c7-0a94-453b-9f09-59fc6c4007aa-c000.snappy.parquet
│   │   ├── part-00000-6efea877-801e-4e84-8b0c-baaad3baf27d-c000.zstd.parquet
│   │   ├── part-00000-714b0875-0cdc-4362-9b2e-9d3a55f5cc6b-c000.snappy.parquet
│   │   ├── part-00000-7266a9f6-15b9-45d1-83ad-92af2a95ca84-c000.snappy.parquet
│   │   ├── part-00000-72b91522-a05e-4290-bb80-e68f223d80d8-c000.snappy.parquet
│   │   ├── part-00000-73c5cf47-a238-45c6-9a9e-de5d4e681ac1-c000.snappy.parquet
│   │   ├── part-00000-743d46a1-46c2-4d81-a080-9e207959871f-c000.snappy.parquet
│   │   ├── part-00000-74986b32-719f-4412-a9fe-e2eb608d080b-c000.zstd.parquet
│   │   ├── part-00000-74faeeab-b053-4d7d-8097-ec2dc9172603-c000.snappy.parquet
│   │   ├── part-00000-75cebe00-4ff4-42b7-a066-c5558177597d-c000.zstd.parquet
│   │   ├── part-00000-76621504-4782-46c1-95f1-a1e2905b1057-c000.snappy.parquet
│   │   ├── part-00000-773cfd81-504d-4cd7-ba6e-5a11825825c4-c000.snappy.parquet
│   │   ├── part-00000-778749c1-2d32-4cef-905d-fb8e0482204a-c000.snappy.parquet
│   │   ├── part-00000-779f7e65-f612-4025-ae38-af4bac234dcf-c000.zstd.parquet
│   │   ├── part-00000-7b647249-b19c-4195-a4d0-148cd093fdb6-c000.snappy.parquet
│   │   ├── part-00000-7c54e44b-de52-45a5-9223-32bdf542f17c-c000.snappy.parquet
│   │   ├── part-00000-7c753cb5-644a-474d-8ccc-2faed955aa2d-c000.snappy.parquet
│   │   ├── part-00000-81c8f60e-1be8-4f52-86eb-8e8029cd3272-c000.zstd.parquet
│   │   ├── part-00000-8354898e-1b36-4123-acbe-e4e3d538a8dd-c000.snappy.parquet
│   │   ├── part-00000-846338ed-6349-40ee-b558-4d0ce9c99ddd-c000.snappy.parquet
│   │   ├── part-00000-8574b611-e004-45f7-bb8a-1317b8727145-c000.snappy.parquet
│   │   ├── part-00000-857e3106-746e-404d-947a-7d107f16a3f1-c000.snappy.parquet
│   │   ├── part-00000-88cd5b0a-5df9-4a1d-9210-f09201a161bd-c000.zstd.parquet
│   │   ├── part-00000-890497a2-515b-47be-9fbc-e685e145827e-c000.zstd.parquet
│   │   ├── part-00000-892a9968-cc2c-43d2-9400-d7ef655d91c4-c000.snappy.parquet
│   │   ├── part-00000-894e4241-b588-4e11-ab35-71c8cfcff0c7-c000.snappy.parquet
│   │   ├── part-00000-8cc5b66e-77b2-44a1-ac84-e0ea158b25b9-c000.snappy.parquet
│   │   ├── part-00000-8e93bc23-7931-4a76-aeb8-4dceb86ae799-c000.snappy.parquet
│   │   ├── part-00000-8f03f88b-38d3-4727-a068-1b3f87f09fd2-c000.snappy.parquet
│   │   ├── part-00000-8fb75eda-7f8d-4827-b461-bee085dd2557-c000.snappy.parquet
│   │   ├── part-00000-905c5f76-e803-498c-b3ba-414ed75ae817-c000.zstd.parquet
│   │   ├── part-00000-908c357e-7d70-433e-9c4e-bf7a53f03ee9-c000.snappy.parquet
│   │   ├── part-00000-90b1424a-931d-4046-9db0-6a303f184961-c000.snappy.parquet
│   │   ├── part-00000-9116409c-77df-4488-b3c4-be316de2ae1b-c000.zstd.parquet
│   │   ├── part-00000-913915dc-90b6-4857-9f45-93dc4e611b7e-c000.snappy.parquet
│   │   ├── part-00000-95752950-35e4-453a-b464-a80652e77b9d-c000.snappy.parquet
│   │   ├── part-00000-9684a27d-de42-4de7-9ce7-68f0ef54f14b-c000.snappy.parquet
│   │   ├── part-00000-993dd902-1a80-46b7-b646-a1a7d49c686e-c000.snappy.parquet
│   │   ├── part-00000-99c2b453-f5c2-4ec9-805f-3612c62ff065-c000.snappy.parquet
│   │   ├── part-00000-99d961a9-422c-40f2-8287-265dad8c1f01-c000.snappy.parquet
│   │   ├── part-00000-9a0c0705-2825-4ece-bfa1-987da2f4558b-c000.zstd.parquet
│   │   ├── part-00000-9a0cb637-d0db-4d57-bc83-df0e40ebe85c-c000.snappy.parquet
│   │   ├── part-00000-9f4b2899-7f3f-4ff7-81a3-ac4d243f1301-c000.snappy.parquet
│   │   ├── part-00000-9f5b1685-4ae2-4078-86b5-cc503f2c5447-c000.zstd.parquet
│   │   ├── part-00000-9fb3cf99-14ec-4ba2-908b-d1033889cd41-c000.snappy.parquet
│   │   ├── part-00000-a33f7e8c-0a62-4423-9f01-9a71674c25db-c000.snappy.parquet
│   │   ├── part-00000-a3eaaf00-a1ef-43b4-b0cc-9c7a100ca0fe-c000.snappy.parquet
│   │   ├── part-00000-a4f10c3a-0fb4-4131-acc5-92a797c2e4f1-c000.snappy.parquet
│   │   ├── part-00000-a5696e85-4223-40b1-a023-7027df00efbb-c000.snappy.parquet
│   │   ├── part-00000-a581c9b2-2ca1-4cac-8ce6-dcb469646192-c000.zstd.parquet
│   │   ├── part-00000-a65fd0a6-faf9-4b90-8a6e-417fbd4fed8c-c000.zstd.parquet
│   │   ├── part-00000-a7989db6-8c5a-47cb-8544-5adcd86049ad-c000.zstd.parquet
│   │   ├── part-00000-a82ecbfb-f2f0-4518-b908-c413764449fc-c000.zstd.parquet
│   │   ├── part-00000-a88ad459-1bd5-4854-b418-4e19f145b04b-c000.snappy.parquet
│   │   ├── part-00000-a8a7f8e2-6793-4387-bcfd-0b1b7b7ee664-c000.snappy.parquet
│   │   ├── part-00000-aa0e556b-7f15-47a6-ba19-385ad60c5ab5-c000.snappy.parquet
│   │   ├── part-00000-ac338ecc-76a4-410a-934f-8fca72876bce-c000.snappy.parquet
│   │   ├── part-00000-af201325-7758-4a44-b459-b2d76a547496-c000.snappy.parquet
│   │   ├── part-00000-afbecce2-d18b-4c4a-80e4-f6d78d0ab41b-c000.zstd.parquet
│   │   ├── part-00000-b02197c3-be83-4faa-b026-e90b0163da75-c000.snappy.parquet
│   │   ├── part-00000-b43c9a36-f277-419f-941a-37e288ab237e-c000.snappy.parquet
│   │   ├── part-00000-b52c7a1d-8577-4188-b63f-f2b2795483bf-c000.zstd.parquet
│   │   ├── part-00000-b9760de1-585b-40c5-b70c-a257abe06c5a-c000.snappy.parquet
│   │   ├── part-00000-ba13e39a-05f2-412f-8f99-f2b23b64c68d-c000.zstd.parquet
│   │   ├── part-00000-bb0bab68-3646-4627-bf90-59bf8390d252-c000.snappy.parquet
│   │   ├── part-00000-bb9547e7-653f-419e-87ca-52ab477e2815-c000.snappy.parquet
│   │   ├── part-00000-bda6fedd-7284-4361-8de1-7dae89cb7f24-c000.snappy.parquet
│   │   ├── part-00000-c08849cc-2f42-4966-b10a-eba0f8702879-c000.zstd.parquet
│   │   ├── part-00000-c0f3ebed-5932-4762-bba3-71507ca7e744-c000.zstd.parquet
│   │   ├── part-00000-c1bd3c38-1705-467e-917a-58a4904029f9-c000.snappy.parquet
│   │   ├── part-00000-c1f920b8-c7b2-47b3-974f-7c8e971f036e-c000.snappy.parquet
│   │   ├── part-00000-c21a8a27-ce13-43ba-89c2-ade9317c2b9d-c000.snappy.parquet
│   │   ├── part-00000-c2a24f19-2b06-4392-b24a-665a477b12d7-c000.zstd.parquet
│   │   ├── part-00000-c2f40383-6178-47c0-b2ff-5ab30660ac2b-c000.snappy.parquet
│   │   ├── part-00000-c491c04d-7470-4f18-87bf-083be7731b6d-c000.zstd.parquet
│   │   ├── part-00000-c49c4b5f-eb8a-4afc-a34f-795a22d705fd-c000.snappy.parquet
│   │   ├── part-00000-c4d51646-3fa0-4429-a0de-a3fb3c260f86-c000.snappy.parquet
│   │   ├── part-00000-c52fe3e3-b7cc-4e80-bf58-4018ef480482-c000.snappy.parquet
│   │   ├── part-00000-c6193048-c3ea-44c4-842a-0840fa90fc76-c000.zstd.parquet
│   │   ├── part-00000-c65beeb4-3764-4fa0-b34b-a330aebb3413-c000.snappy.parquet
│   │   ├── part-00000-c9e66540-1bb7-4969-a9b2-58d9f8e58ace-c000.snappy.parquet
│   │   ├── part-00000-cc7a8406-f9d3-434f-a95c-184c9bef5589-c000.snappy.parquet
│   │   ├── part-00000-cf752113-3ffe-4280-a59e-dd9c31b64d92-c000.zstd.parquet
│   │   ├── part-00000-d26a4e68-d931-441b-b714-4aab4c2e890f-c000.snappy.parquet
│   │   ├── part-00000-d4be5677-39ab-433e-8475-23cce0100de0-c000.snappy.parquet
│   │   ├── part-00000-d67d7ae5-49e9-40a8-8ba1-307733b3e8e1-c000.snappy.parquet
│   │   ├── part-00000-d701ed5f-920d-49a1-8bb2-c7649c4e4b67-c000.snappy.parquet
│   │   ├── part-00000-d77dbeda-eca6-415a-becb-621901ff5c12-c000.snappy.parquet
│   │   ├── part-00000-d80f0756-4ab5-4295-81a9-82d634f8bc24-c000.snappy.parquet
│   │   ├── part-00000-d9718c97-9bb0-4af6-8b90-05e4782225af-c000.snappy.parquet
│   │   ├── part-00000-d9ac8a2c-fb22-41ef-952a-1d0bd23c05bd-c000.snappy.parquet
│   │   ├── part-00000-dbb4710f-106d-42b3-ad68-0b6904e2bbce-c000.zstd.parquet
│   │   ├── part-00000-dc0be853-f357-4dcb-84d0-d698b69b171c-c000.snappy.parquet
│   │   ├── part-00000-dc1fc6e7-b1ae-4432-a125-cd36cb08f038-c000.snappy.parquet
│   │   ├── part-00000-dc59128e-3d04-4bd4-906f-fb4b7a4d45bd-c000.zstd.parquet
│   │   ├── part-00000-de255216-aff8-4703-9ce0-4bbb869d6439-c000.snappy.parquet
│   │   ├── part-00000-de9be69a-81d9-426b-b177-e3c78cfdebc5-c000.snappy.parquet
│   │   ├── part-00000-df24f599-ab61-4361-b279-dbe3986bdec4-c000.snappy.parquet
│   │   ├── part-00000-df889feb-2b2e-482d-bfc8-5d05f6a793f4-c000.snappy.parquet
│   │   ├── part-00000-e011ff85-a329-4c1c-96b4-c5c5918d7a3b-c000.zstd.parquet
│   │   ├── part-00000-e08f96c5-9a42-4e33-a635-b29ba542eeb5-c000.zstd.parquet
│   │   ├── part-00000-e1e73f65-6150-4359-814e-d7a9b00dc87d-c000.snappy.parquet
│   │   ├── part-00000-e20373b3-b6e3-460d-885a-c68d244fe569-c000.zstd.parquet
│   │   ├── part-00000-e3013487-6e9e-4efc-97a6-32aea775c321-c000.zstd.parquet
│   │   ├── part-00000-e6cad3ff-ba9f-4019-9c49-36fbb7903058-c000.zstd.parquet
│   │   ├── part-00000-e8380133-e91f-47c5-aaaf-41fb01873a04-c000.snappy.parquet
│   │   ├── part-00000-ec983ca4-dcac-499f-9d6c-4ec52af486e8-c000.snappy.parquet
│   │   ├── part-00000-ecd59d3c-4efa-4e0c-97b0-db5ed0fcbc8e-c000.snappy.parquet
│   │   ├── part-00000-ed7fe1a3-fc67-4ac8-ae6a-8cb26b9259bd-c000.snappy.parquet
│   │   ├── part-00000-ee5b57cf-382f-4d84-919a-4b55e0eb05b5-c000.snappy.parquet
│   │   ├── part-00000-ee792730-9ca8-470d-8787-f53804a21672-c000.zstd.parquet
│   │   ├── part-00000-eeb06630-8cad-4143-928e-0e35cc348cf3-c000.snappy.parquet
│   │   ├── part-00000-eebe2ca8-0e20-4606-b2ae-4c6f5f68a2e9-c000.snappy.parquet
│   │   ├── part-00000-f18fd1b8-a797-4060-be8c-28401b2365bb-c000.snappy.parquet
│   │   ├── part-00000-f43483fd-82db-4bea-b4cc-dd2d057f1e02-c000.snappy.parquet
│   │   ├── part-00000-f459f0aa-b8d7-42d0-8d11-9ec8f8def94f-c000.zstd.parquet
│   │   ├── part-00000-f4ae121f-973c-475d-9504-0ab1a9a3c77a-c000.snappy.parquet
│   │   ├── part-00000-f4b359f9-776f-4c34-b6fe-392b7710acc1-c000.snappy.parquet
│   │   ├── part-00000-f72b06fe-5be6-4b47-af37-fad56a70d707-c000.snappy.parquet
│   │   ├── part-00000-f832d7a6-30a2-4d9d-a013-f9a5acffbc72-c000.snappy.parquet
│   │   ├── part-00000-f88d0a15-0bde-4b8f-96e7-794ef03972be-c000.snappy.parquet
│   │   ├── part-00000-f8d77df8-10ac-4b1a-89b4-75ff76701c9f-c000.snappy.parquet
│   │   ├── part-00000-fa11709a-d8e6-4f17-b0d7-30cfce8ea04e-c000.snappy.parquet
│   │   ├── part-00000-faca0792-2286-4bfb-9640-a9c9cb681142-c000.snappy.parquet
│   │   ├── part-00000-fb2c9189-d672-4113-8b38-35092a192e1b-c000.snappy.parquet
│   │   ├── part-00000-fd7a58d6-d46b-4895-9fd0-8f9c7833fa2f-c000.snappy.parquet
│   │   ├── part-00000-fe66f6f8-43e4-45c3-a9e2-ead93a4291ec-c000.snappy.parquet
│   │   ├── part-00000-febb84d7-2f8b-4c1e-9f5e-5fe6bb2c576f-c000.snappy.parquet
│   │   ├── part-00000-ff2486a9-493c-40be-a1d7-cc9e9a699b8c-c000.snappy.parquet
│   │   ├── part-00001-479a470e-8a01-4699-94ac-9dfe000ab68a-c000.zstd.parquet
│   │   ├── part-00002-479a470e-8a01-4699-94ac-9dfe000ab68a-c000.zstd.parquet
│   │   ├── part-00003-479a470e-8a01-4699-94ac-9dfe000ab68a-c000.zstd.parquet
│   │   ├── part-00004-479a470e-8a01-4699-94ac-9dfe000ab68a-c000.zstd.parquet
│   │   ├── part-00005-479a470e-8a01-4699-94ac-9dfe000ab68a-c000.zstd.parquet
│   │   ├── part-00006-479a470e-8a01-4699-94ac-9dfe000ab68a-c000.zstd.parquet
│   │   ├── part-00007-479a470e-8a01-4699-94ac-9dfe000ab68a-c000.zstd.parquet
│   │   ├── part-00008-479a470e-8a01-4699-94ac-9dfe000ab68a-c000.zstd.parquet
│   │   ├── part-00009-479a470e-8a01-4699-94ac-9dfe000ab68a-c000.zstd.parquet
│   │   ├── part-00010-479a470e-8a01-4699-94ac-9dfe000ab68a-c000.zstd.parquet
│   │   ├── part-00011-479a470e-8a01-4699-94ac-9dfe000ab68a-c000.zstd.parquet
│   │   ├── part-00012-479a470e-8a01-4699-94ac-9dfe000ab68a-c000.zstd.parquet
│   │   ├── part-00013-479a470e-8a01-4699-94ac-9dfe000ab68a-c000.zstd.parquet
│   │   ├── part-00014-479a470e-8a01-4699-94ac-9dfe000ab68a-c000.zstd.parquet
│   │   ├── part-00015-479a470e-8a01-4699-94ac-9dfe000ab68a-c000.zstd.parquet
│   │   ├── part-00016-479a470e-8a01-4699-94ac-9dfe000ab68a-c000.zstd.parquet
│   │   ├── part-00017-479a470e-8a01-4699-94ac-9dfe000ab68a-c000.zstd.parquet
│   │   ├── part-00018-479a470e-8a01-4699-94ac-9dfe000ab68a-c000.zstd.parquet
│   │   ├── part-00019-479a470e-8a01-4699-94ac-9dfe000ab68a-c000.zstd.parquet
│   │   ├── part-00020-479a470e-8a01-4699-94ac-9dfe000ab68a-c000.zstd.parquet
│   │   ├── part-00021-479a470e-8a01-4699-94ac-9dfe000ab68a-c000.zstd.parquet
│   │   ├── part-00022-479a470e-8a01-4699-94ac-9dfe000ab68a-c000.zstd.parquet
│   │   ├── part-00023-479a470e-8a01-4699-94ac-9dfe000ab68a-c000.zstd.parquet
│   │   ├── part-00024-479a470e-8a01-4699-94ac-9dfe000ab68a-c000.zstd.parquet
│   │   ├── part-00025-479a470e-8a01-4699-94ac-9dfe000ab68a-c000.zstd.parquet
│   │   ├── part-00026-479a470e-8a01-4699-94ac-9dfe000ab68a-c000.zstd.parquet
│   │   ├── part-00027-479a470e-8a01-4699-94ac-9dfe000ab68a-c000.zstd.parquet
│   │   ├── part-00028-479a470e-8a01-4699-94ac-9dfe000ab68a-c000.zstd.parquet
│   │   ├── part-00029-479a470e-8a01-4699-94ac-9dfe000ab68a-c000.zstd.parquet
│   │   ├── part-00030-479a470e-8a01-4699-94ac-9dfe000ab68a-c000.zstd.parquet
│   │   ├── part-00031-479a470e-8a01-4699-94ac-9dfe000ab68a-c000.zstd.parquet
│   │   ├── part-00032-479a470e-8a01-4699-94ac-9dfe000ab68a-c000.zstd.parquet
│   │   ├── part-00033-479a470e-8a01-4699-94ac-9dfe000ab68a-c000.zstd.parquet
│   │   ├── part-00034-479a470e-8a01-4699-94ac-9dfe000ab68a-c000.zstd.parquet
│   │   ├── part-00035-479a470e-8a01-4699-94ac-9dfe000ab68a-c000.zstd.parquet
│   │   ├── part-00036-479a470e-8a01-4699-94ac-9dfe000ab68a-c000.zstd.parquet
│   │   ├── part-00037-479a470e-8a01-4699-94ac-9dfe000ab68a-c000.zstd.parquet
│   │   ├── part-00038-479a470e-8a01-4699-94ac-9dfe000ab68a-c000.zstd.parquet
│   │   ├── part-00039-479a470e-8a01-4699-94ac-9dfe000ab68a-c000.zstd.parquet
│   │   ├── part-00040-479a470e-8a01-4699-94ac-9dfe000ab68a-c000.zstd.parquet
│   │   ├── part-00041-479a470e-8a01-4699-94ac-9dfe000ab68a-c000.zstd.parquet
│   │   ├── part-00042-479a470e-8a01-4699-94ac-9dfe000ab68a-c000.zstd.parquet
│   │   ├── part-00043-479a470e-8a01-4699-94ac-9dfe000ab68a-c000.zstd.parquet
│   │   ├── part-00044-479a470e-8a01-4699-94ac-9dfe000ab68a-c000.zstd.parquet
│   │   ├── part-00045-479a470e-8a01-4699-94ac-9dfe000ab68a-c000.zstd.parquet
│   │   ├── part-00046-479a470e-8a01-4699-94ac-9dfe000ab68a-c000.zstd.parquet
│   │   ├── part-00047-479a470e-8a01-4699-94ac-9dfe000ab68a-c000.zstd.parquet
│   │   ├── part-00048-479a470e-8a01-4699-94ac-9dfe000ab68a-c000.zstd.parquet
│   │   ├── part-00049-479a470e-8a01-4699-94ac-9dfe000ab68a-c000.zstd.parquet
│   │   ├── part-00050-479a470e-8a01-4699-94ac-9dfe000ab68a-c000.zstd.parquet
│   │   ├── part-00051-479a470e-8a01-4699-94ac-9dfe000ab68a-c000.zstd.parquet
│   │   ├── part-00052-479a470e-8a01-4699-94ac-9dfe000ab68a-c000.zstd.parquet
│   │   ├── part-00053-479a470e-8a01-4699-94ac-9dfe000ab68a-c000.zstd.parquet
│   │   └── part-00054-479a470e-8a01-4699-94ac-9dfe000ab68a-c000.zstd.parquet
│   ├── maint_events
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
ls -la _lakehouse/

# Kiểm tra nội dung transaction log
cat _lakehouse/scratch/users_delta/_delta_log/00000000000000000000.json
```
