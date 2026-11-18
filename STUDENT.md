# Thông tin học viên

- Họ và tên: Lê Trung Tá
- MSHV: 2A202603213
- Lớp (L2 / L3 / khác): L2
- Tài khoản GitHub: trugta
- Repo bài nộp (URL): https://github.com/trugta/K5-L23-DAY08-2A202603213-LeTrungTa-AIPrelabel.git
- Commit chốt bài (hash): submit
- Máy: Windows, chạy script bằng PowerShell. Chạy SAM 2.1 Base trên Google Colab. GPU của phiên chạy chưa được xác nhận.
- CVAT (Docker): 2.74.1

## Kết quả tóm tắt

Trên 24 ảnh, thống kê đã loại 22 đối tượng nằm quanh các bẫy trước khi tính bốn trạng thái. Nguồn: `submission/qc/review_stats.md`.

| Trạng thái | Số lượng / mẫu số | Tỉ lệ |
|---|---:|---:|
| accepted – giữ nguyên | 488 / 507 box máy | 96,3% |
| edited – sửa box | 3 / 507 box máy | 0,6% |
| deleted – xóa box | 16 / 507 box máy | 3,2% |
| added – vẽ thêm | 15 / 506 đối tượng sau duyệt | 3,0% |

- **So với gold.** Trên 4 ảnh gán tay, máy có 0 TP, 73 FP và 20 FN. Precision đạt 0/73 = 0% và recall đạt 0/20 = 0%.
- **Bắt bẫy.** Em phát hiện 2/12 bẫy (16,7%), gồm 1/3 bẫy thiếu vật và 1/3 bẫy box lỏng. Em bỏ lọt cả 3/3 bẫy box thừa và 3/3 bẫy sai lớp.
- **Thời gian.** Gán tay mất 8 phút cho 4 ảnh, trung bình 2 phút/ảnh. Duyệt có pre-label mất 15 phút cho 24 ảnh, trung bình 0,625 phút/ảnh (37,5 giây/ảnh). Nguồn: `submission/time_log.csv`.
- **Lỗi QC.** Chờ số ảnh có lỗi / tổng số ảnh kiểm và số đối tượng lỗi / tổng số đối tượng kiểm từ `submission/qc/qc_report_auto.md`.
- **Quyết định tạm thời: Hold.** Tỉ lệ bắt bẫy 2/12 thấp hơn ngưỡng 80% của bài. Em cần duyệt lại lô, ưu tiên box thừa và sai lớp, rồi chốt quyết định sau khi có kết quả QC.
