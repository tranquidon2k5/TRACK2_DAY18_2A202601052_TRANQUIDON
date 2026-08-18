# Reflection: Top Lakehouse Anti-Patterns

**Anti-pattern team dễ vướng nhất:** **Expire Snapshots độc lập mà không dọn Orphan Files & chạy VACUUM.**

### Lý do:
Trong thực tế production, team thường thiết lập cron job tự động `expire_snapshots` để giảm số lượng snapshot của bảng Lakehouse (Iceberg/Delta) với kỳ vọng sẽ cắt giảm ngay chi phí lưu trữ Cloud Storage (S3/GCS).

Tuy nhiên, kết quả đo đạc từ **Notebook 6 (NB6)** đã chỉ ra một "bẫy" sản xuất rất phổ biến: `expire_snapshots` chỉ giải phóng các bản ghi metadata mà **không tự động xóa các data file (avro/parquet)** bị bỏ lại từ các job bị crash giữa chừng (chưa từng được commit vào transaction log). 

Nếu không kết hợp định kỳ giữa **Job 3 (Snapshot Expiry)** và **Job 4 (Orphan Cleanup/VACUUM)** thành một cặp song song, hệ thống vẫn chịu lãng phí chi phí dung lượng rất lớn dù số lượng snapshot hiển thị đã giảm từ 20 xuống 3. Đây là nguyên nhân hàng đầu khiến hóa đơn lưu trữ không giảm như kỳ vọng mà team rất khó phát hiện nếu chỉ nhìn vào metadata.
