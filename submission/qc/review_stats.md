# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 507; đối tượng sau duyệt: 506; ngưỡng accepted: IoU >= 0.9
- Đã **loại 22 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 488 | 488/507 = 96.3% | 507 box máy |
| edited (sửa) | 3 | 3/507 = 0.6% | 507 box máy |
| deleted (xoá) | 16 | 16/507 = 3.2% | 507 box máy |
| added (vẽ thêm) | 15 | 15/506 = 3.0% | 506 đối tượng sau duyệt |

Trong số deleted + added, **5 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 10.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 267 | 0 | 2 | 3 |
| bicycle | 6 | 0 | 3 | 1 |
| motorbike | 138 | 1 | 0 | 2 |
| car | 54 | 0 | 2 | 1 |
| bus | 3 | 0 | 0 | 2 |
| truck | 7 | 0 | 0 | 1 |
| three_wheeler | 13 | 2 | 9 | 5 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 20 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 0 | 73 | 20 | 0.0% (KTC 95%: 0–5%) | 0.0% (KTC 95%: 0–16%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 2/12 = 16.7% (KTC 95%: 5–45%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - missing: 1/3
  - extra: 0/3
  - wrong_class: 0/3
  - loose: 1/3
  - lọt: img_035.jpg (extra), img_023.jpg (wrong_class), img_014.jpg (missing), img_022.jpg (missing), img_031.jpg (loose), img_005.jpg (loose), img_034.jpg (wrong_class), img_013.jpg (wrong_class), img_021.jpg (extra), img_010.jpg (extra)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
