# Bài Phản Ánh: Lakehouse Anti-Patterns Trong Thực Tế

**Anti-Pattern Nguy Hiểm Nhất: Incomplete Maintenance (Dọn Dẹp Nửa Vời)**

Trong streaming và pipeline AI/ML, rủi ro lớn nhất là **Incomplete Maintenance** — lầm tưởng chỉ chạy `VACUUM` hoặc `expire_snapshots` là dữ liệu đã sạch.

Thực tế, streaming vi mô hoặc job crash để lại file Parquet "mồ côi" (orphan files) trên storage. Như chứng minh ở NB6, `VACUUM` của Delta chỉ xóa file đã commit trong `_delta_log/` bị tombstone; hoàn toàn mù trước file rác uncommitted. Tương tự, `expire_snapshots` của Iceberg chỉ đổi metadata snapshot, không xóa file dữ liệu vật lý.

Thiếu job quét rác bằng thuật toán hiệu tập hợp (Set Difference) giữa metadata và storage, dung lượng cloud sẽ âm thầm phình to, lãng phí chi phí. Đồng thời, log không được checkpoint và compact làm giảm tốc độ đọc. Do đó, bảo trì bắt buộc kết hợp đồng thời: Compaction, Clustering, Snapshot Expiry và Orphan Cleanup tự động.
