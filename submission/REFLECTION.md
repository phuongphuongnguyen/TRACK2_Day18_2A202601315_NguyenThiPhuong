# Reflection — Day 18 Lakehouse Lab

**Anti-Pattern: "Maintenance Jobs Are a Luxury, Not a Requirement"**

NB6 và NB7 lộ ra rằng việc **skip maintenance không phải là tiết kiệm — nó là lỗi định thời.**

NB6 đo được: `VACUUM` không dọn orphan từ job crash (chưa từng vào transaction log) → 3 file stranded trên đĩa mãi mãi. NB6 cũng bắt gặp `expire_snapshots` của Iceberg xoá 0 byte dữ liệu từ S3 dù đã xoá 17/20 snapshot — metadata vẫn phình ra 2 tỷ bytes. Nếu toàn bộ bảng LLM observability 1B request/ngày bị như vậy: 10 GB/ngày × 365 ngày × $0.023/GB = **$84/năm chỉ để lưu metadata xác chết.**

NB7 phát hiện thêm: Delta không cố định chiều vector → `fixed_size_list[256]` đọc lại thành `list<float>` bất định. External index stale vẫn trả về dữ liệu đã xoá khỏi bảng (lifecycle bug). Tuần lộn như vậy tự động sinh bug trong training.

NB2/NB5 đo: small-file problem tạo 527 file nhỏ, pruning phải quét 100% trước khi filter → **5× tỷ lệ amplification.**

Team dễ bị cái này nhất vì maintenance **vẫn chạy mà logs lặng im** — mọi người sẽ tưởng OK cho đến khi bill tăng hoặc model accuracy rơi.

