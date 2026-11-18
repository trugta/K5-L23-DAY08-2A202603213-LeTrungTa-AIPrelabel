# Báo cáo Day 08 — 2A202603213 – Lê Trung Tá

## 1. Model và recipe (Part 2–3)

Em dùng dự đoán YOLO26 cho sáu lớp giao thông phổ biến và YOLOE với prompt `three_wheeler` cho xe ba bánh. Em quét ngưỡng trên 4 ảnh gold với mục tiêu recall 90%. Kết quả trong `qc/threshold_sweep.md` dẫn đến cùng một lựa chọn cho cả 7 lớp: giữ ngưỡng mặc định 0,3.

| Lớp | Model theo cấu hình lab | Số vật trong 4 ảnh gold | Cơ sở chọn ngưỡng |
|---|---|---:|---|
| person | YOLO26m | 7 | Recall cao nhất trong dải xét đến 0,5 là 0/7. |
| bicycle | YOLO26m | 6 | Recall cao nhất là 0/6. |
| car | YOLO26m | 6 | Recall cao nhất là 0/6. |
| truck | YOLO26m | 1 | Chỉ có 1 mẫu, dưới mức tối thiểu 5 vật. |
| motorbike, bus | YOLO26m | 0 mỗi lớp | Gold không có ví dụ để chọn ngưỡng. |
| three_wheeler | YOLOE | 0 | Gold không có xe ba bánh để đánh giá prompt. |

Bốn ảnh gold có tổng cộng 20 vật thuộc các lớp trên. Việc chọn và đánh giá ngưỡng trên cùng 4 ảnh có thể làm kết quả lạc quan. Đặc biệt, recall bằng 0 cho người, xe đạp và ô tô đáng được kiểm tra bằng cách mở cùng một ảnh gold, đối chiếu tên ảnh, lớp và vị trí box giữa hai file.

Lệnh PowerShell tương ứng với cấu hình gộp của bài:

```powershell
python .\scripts\merge_prelabels.py ".\submission\labels\colab_outputs\work\prelabel\yoloe_all\pre.coco.json" ".\submission\labels\colab_outputs\work\prelabel\y26_all\pre.coco.json" --thr= --conf 0.3 --iou 0.6 --split "batch,gold" --out ".\work\prelabel\merged_review\pre.coco.json"
```

Em chọn SAM 2.1 Base (`sam2.1_b.pt`) để có thể chỉ định phần cần giữ hoặc loại bằng điểm dương và điểm âm trong Colab. Với detector, nếu box đã ôm đúng xe nhưng nhầm lớp thì đổi ngưỡng chỉ ảnh hưởng việc giữ hay bỏ box. Khi duyệt, em cần sửa trực tiếp tên lớp.

**Cần bổ sung từ Part 2:** đối chiếu cỡ model và lệnh thực chạy, ghi các `run_id` trong `model_log.csv`, rồi dẫn một box có điểm cao nhưng sai hoặc điểm thấp nhưng đúng từ ảnh minh họa của lần chạy đó.

## 2. AI error profile

| Trạng thái | Cách hiểu | Mẫu số để tính tỉ lệ |
|---|---|---|
| accepted | Giữ nguyên box máy | Tổng số box máy trên batch |
| edited | Sửa box máy | Tổng số box máy trên batch |
| deleted | Xóa box máy | Tổng số box máy trên batch |
| added | Bổ sung vật khi duyệt | Tổng số đối tượng sau duyệt trên batch |

**Chờ `qc/review_stats.md`:** điền số lượng, mẫu số và tỉ lệ thực tế cho bốn dòng trên. Lớp có nhiều lỗi cần kèm một tên ảnh và mô tả lỗi nhìn thấy. Với kết quả máy trên gold, ghi precision = TP/(TP+FP) và recall = TP/(TP+FN) theo thống kê của lần chạy.

## 3. Thiên kiến tự động hóa và ảnh bẫy

Bài lab cài 12 bẫy. Tỉ lệ accepted cho biết bao nhiêu box máy được giữ lại, còn số bẫy bắt được cho biết em nhận ra bao nhiêu lỗi đã cài. Nếu em chỉ xem từng box có sẵn thì một vật bị bỏ sót có thể vẫn lọt qua dù phần lớn box nhìn có vẻ hợp lý.

**Chờ kết quả:** ghi số bẫy bắt được trên 12, loại bẫy lọt và ảnh tương ứng từ `qc/review_stats.md`. Đối chiếu ảnh với checklist trước khi kết luận nguyên nhân.

## 4. Máy có đáng dùng không?

| Công việc | Ảnh | Tổng thời gian | Thời gian trung bình mỗi ảnh |
|---|---:|---:|---:|
| Gán tay gold | 4 | 8 phút | 120 giây |
| Duyệt batch | 24 | 15 phút | 37,5 giây |

Theo `time_log.csv`, mỗi ảnh batch mất ít hơn 82,5 giây so với mức trung bình của 4 ảnh gold, tương đương giảm khoảng 69% thời gian mỗi ảnh. Hai tập có thể khác nhau về mật độ xe và khi duyệt batch em đã quen cảnh sau phần gold. Em thấy pre-label đáng dùng để rút ngắn thời gian tạo nhãn ban đầu. Để quyết định chất lượng đầu ra có đạt hay không, em còn cần đối chiếu kết quả bắt bẫy và QC.

Ba quy tắc em dùng làm checklist cho lần duyệt tiếp theo:

1. Ẩn box và quét hết ảnh một lượt để tìm vật bị bỏ sót.
2. Bật lại box, lần lượt kiểm tra box thừa, nhãn lớp và bốn cạnh của từng box.
3. Với trường hợp khó phân biệt, đánh dấu `can_review` và ghi tên ảnh cùng lý do để kiểm tra lại theo guideline.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM | Số click đã ghi | Người lái, hàng hóa và lỗi | File kết quả dự kiến trong masks/ |
|---|---|---:|---|---|
| img_002 | SAM 2.1 Base | 2 | | tuktuk_img002.jpg |
| img_004 | SAM 2.1 Base | 3 |  | tuktuk_img004.jpg |
| img_013 | SAM 2.1 Base | 3 | Ghi nhận trong bảng ban đầu: mask gồm người lái. Cần đối chiếu phần hàng hóa trên ảnh cuối. | tuktuk_img013.jpg |

Trong lần thử lệnh mẫu trên `img_013.jpg`, điểm dương tại (600, 110) rơi vào bầu trời và vùng tô lan rộng trên nền trời. Ở ảnh có nhiều tuk-tuk đỗ cạnh nhau, một điểm dương rơi trên mặt đường và vùng tô xuất hiện quanh điểm đó. Hai lần thử cho thấy em cần kiểm tra vị trí chấm ngay trên ảnh sau mỗi lần chạy. Với xe ở tiền cảnh, thân xe và mái là hai vị trí dễ đặt điểm dương. Phần nền bị lấy nhầm là vị trí thích hợp để đặt điểm âm.

**Cần đối chiếu trước khi nộp:** mở ba ảnh mask cuối, xác nhận số click của lần chạy tạo mỗi ảnh và so phần người lái, hàng hóa với quy tắc ở mục 4.2 của guideline.

## 6. Calibration (Part 6)

Một ca có thể đưa ra thảo luận là xe ba bánh chồng lên người lái: người duyệt cần biết đường biên dừng ở đâu khi thân người che một phần xe. Để chốt ca này, cần đối chiếu ảnh gốc, mask cuối và câu chữ ở mục 4.2.

**Chờ kết quả Part 6:** ghi số quyết định trong `qc/calibration_decisions.csv`, mã quyết định quan trọng nhất, nguyên nhân gốc và quy tắc thực sự được thêm vào `guidelines/guideline_v1.1_calibrated.md`.

## 7. QC và quyết định (Part 6)

**Trạng thái báo cáo:** chưa đủ số liệu để chốt Release / Hold / Rework. Việc tự kiểm tra lại chính các ảnh mình đã duyệt có thể khiến em bỏ qua cùng một lỗi ở cả hai lượt.

Khi có `qc/qc_review.csv` và `qc/qc_report_auto.md`, ghi số ảnh được chọn từ nhóm `can_review` và nhóm ngẫu nhiên, số đối tượng lỗi trên tổng đối tượng kiểm, số ảnh lỗi trên tổng ảnh kiểm, cận trên 95% và tên ảnh minh họa cho từng lỗi chính. Dùng kết quả đó để ghi quyết định, ngày và việc cần làm tiếp theo. Kiểm tra tự động có thể phát hiện lỗi cấu trúc nhãn. Việc box có ôm đúng vật hoặc còn vật bị bỏ sót cần kiểm tra trên ảnh.

## 8. Ghi chú cá nhân

Em dùng ChatGPT để đọc lỗi PowerShell, sửa đường dẫn, hiểu cách nạp nhãn vào CVAT, chọn vị trí click thử cho SAM và biên soạn báo cáo. Khi kiểm tra gợi ý tọa độ, em có thể nhìn chấm dương, chấm âm và vùng tô trực tiếp trên ảnh kết quả.

Em dùng Windows PowerShell để chạy script, CVAT để duyệt và Colab để chạy SAM. Hai vướng mắc chính là dùng dấu nối dòng của Bash trong PowerShell và truyền đường dẫn file khác vị trí thực tế. Ví dụ, khi chạy thống kê, script dừng vì chưa tìm thấy `submission/labels/batch_reviewed.json`. Lần sau em sẽ kiểm tra file đầu vào trước khi chạy và lưu ảnh kết quả cùng số click ngay sau mỗi lượt thử SAM.

**Cần em xác nhận:** có xem pre-label hoặc hỏi AI về nhãn trước khi hoàn tất gold hay không, có dùng `data/cache/` không và loại GPU nếu đã ghi nhận.
