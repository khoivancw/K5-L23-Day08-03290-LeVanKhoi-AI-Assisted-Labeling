# Label contract — Phố châu Á — BỔ SUNG v1.1 — 03290 - Lê Văn Khởi

> **Chỉ viết phần thay đổi so với v1.0** (không copy cả file). Chốt các câu hỏi mở ở mục 6 của v1.0 (hoặc nói rõ câu nào chưa chốt, vì sao)
> và thêm quy tắc mới rút ra từ lỗi của máy / lỗi của bạn. Mỗi ca khó **có ảnh ví dụ** (`img_0xx`) và khớp một dòng trong `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 1 | Xe đạp điện / xe máy điện nhỏ: bicycle hay motorbike? | Thống nhất gán vào lớp `motorbike` vì có động cơ điện, vận tốc di chuyển nhanh và lưu thông chung làn xe cơ giới. | img_018, img_024 | CAL-01 |
| 3 | Hàng nhô ra: xe máy / ba gác chở hoa quả, rổ cồng kềnh nhô xa: box xe có gồm hàng không? | Box xe ôm cả xe và toàn bộ khối hàng hóa xếp trên giá đèo hàng gắn cố định để hệ thống xe tự hành phát hiện trọn vẹn chướng ngại vật. | img_005, img_021 | CAL-02 |

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| img_013, img_016 | Xe tuk-tuk 3 bánh địa phương có mui che | YOLO26 gọi nhầm thành `car` hoặc `truck` do COCO closed-set không có nhãn này | Dùng YOLOE với text prompt hoặc sửa tay về `three_wheeler` | Mục 2: Mọi xe 3 bánh có mui / thùng chở khách/hàng ở Đông Nam Á bắt buộc gán `three_wheeler` |
| img_028, img_029 | Xe/người bị AI bỏ sót nhưng reviewer không phát hiện ra do automation bias | AI bỏ sót box (bẫy missing), reviewer chỉ tập trung nhìn box máy nên bị lọt | Thêm quy tắc bắt buộc ẩn toàn bộ nhãn máy để quét ảnh gốc trước | Mục 4.1: Checklist duyệt nhãn 4 bước bắt buộc phải ẩn nhãn máy (phím Space) để quét vùng trống |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên viết sẵn | — |
| v1.1 | Quy định xe đạp điện vào `motorbike`; hàng trên giá đèo thuộc box xe; chuẩn hóa `three_wheeler` cho tuk-tuk; bổ sung checklist review chống automation bias | CAL-01, CAL-02, CAL-03, CAL-04 |
