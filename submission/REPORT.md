# Báo cáo Day 08 — 03290 - Lê Văn Khởi

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | YOLO26-s | car=0.45, còn lại 0.3 | r02, r03: Model bắt rất tốt các lớp phổ biến COCO; car quét ngưỡng tối ưu đạt 100% recall ở conf 0.45; các lớp khác giữ 0.3 để tránh bỏ sót |
| three_wheeler | YOLOE-26s, prompt: 'tuk-tuk', 'cyclo', 'cargo tricycle' | 0.25 (mặc định 0.3) | r05: YOLO26 ra 0 box do closed-set COCO; YOLOE dùng text prompt bắt được 9 xe three_wheeler |

- Lệnh `merge_prelabels.py` đã dùng (dán nguyên):
  `.\.venv\Scripts\python.exe scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "car=0.45" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json`
- **Phiên bản SAM bạn chọn** (SAM 1 / 2.1 / 3) và lý do: Chọn **SAM 2.1 Base** (`sam2.1_b.pt`) vì chất lượng mask bám rất nét, cho 1 polygon liền mạch bao quanh xe (317.260 px), cân bằng tốc độ suy luận GPU (0.2s) và bộ nhớ tốt hơn hẳn so với MobileSAM bị phân mảnh thành 2 polygon (245.517 px).
- Lỗi nào **không sửa được bằng đổi ngưỡng** (ví dụ tuk-tuk bị gọi `car`)? Sửa thế nào? Lỗi gán sai lớp do danh mục đóng (*Closed-set classification error*): YOLO26 không có nhãn `three_wheeler` nên dù tăng hay giảm ngưỡng thì tuk-tuk vẫn bị gán nhầm là `car` hoặc `truck`. Cách sửa: bắt buộc dùng mô hình từ vựng mở (*Open-vocabulary detector* như YOLOE) có text prompt hoặc con người can thiệp sửa nhãn thủ công.
- Box **điểm cao mà sai** hoặc **điểm thấp mà đúng** (ảnh nào):
  - Điểm cao mà sai: tại `img_016.jpg`, xe tuk-tuk có điểm conf 0.72 nhưng bị gán nhầm thành `car`.
  - Điểm thấp mà đúng: tại `img_011.jpg`, xe máy ở xa bị che khuất một phần có điểm conf 0.28 nhưng detector vẫn khoanh đúng vị trí.
- Giới hạn: chọn ngưỡng trên chính gold (4 ảnh) nên số đo máy trên gold là lạc quan: Do chọn ngưỡng và chấm điểm trên cùng 4 ảnh gold (in-sample bias), dữ liệu mẫu quá nhỏ (4 ảnh) nên số đo Precision/Recall trên gold mang tính ước lượng thô và lạc quan hơn so với triển khai thực tế.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 367 | 81.2% | 367/452 box máy |
| edited | 26 | 5.8% | 26/452 box máy |
| deleted | 58 | 12.8% | 58/452 box máy |
| added | 59 | 13.1% | 59/452 đối tượng sau duyệt |

- Lớp nào máy tệ nhất? Vì sao (dẫn ảnh)? Lớp `three_wheeler` và `motorbike` máy tệ nhất:
  - `three_wheeler`: bị xóa 19 box và sửa 8 box do YOLOE tạo ra nhiều box dư thừa hoặc box lỏng lẻo quanh mui xe tại `img_013.jpg`, `img_016.jpg`.
  - `motorbike`: bị vẽ thêm (*added*) 22 box do máy bỏ sót người và xe máy ở xa bị che khuất tại các ảnh đông xe như `img_018.jpg`, `img_024.jpg`.
- So với gold: precision / recall của **máy**: Precision = **49.2%**, Recall = **54.5%** (mẫu số: 55 đối tượng gold).

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: **6/12 = 50.0%**; loại nào lọt: lọt 3 bẫy bỏ sót `missing` (`img_028.jpg`, `img_029.jpg`, `img_034.jpg`) và 3 bẫy box lỏng `loose` (`img_003.jpg`, `img_024.jpg`, `img_035.jpg`).
- Vì sao (bạn đã bỏ bước nào của checklist)?
  - Đã bỏ qua bước đầu tiên của checklist: **Tạm ẩn nhãn máy (phím Space)** để quét toàn cảnh ảnh gốc trước. Do mắc thiên kiến tự động hóa (*Automation bias & Inattentional blindness*), mắt chỉ tập trung nhìn vào những vị trí đã có box máy sẵn nên hoàn toàn không chú ý đến các khoảng trống bị AI bỏ sót vật (`missing`).
  - Mắc tâm lý thỏa mãn sơ bộ (*Satisficing heuristic*): thấy box đã ôm quanh thân xe thì chấp nhận luôn, không phóng to lên để kiểm tra mép viền nên để lọt bẫy `loose`.
- Accepted cao mà bắt bẫy thấp (hoặc ngược lại) nói gì về cách bạn duyệt? Tỷ lệ accepted cao (81.2%) nhưng bắt bẫy thấp (50.0%) chứng minh reviewer có xu hướng **tiếp nhận thụ động các gợi ý của AI**, lười nghi ngờ và lười vẽ bổ sung các vùng trống ngoài các box máy có sẵn.

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): gán tay **3.75 phút/ảnh** (15 phút / 4 ảnh gold) vs duyệt **2.08 phút/ảnh** (50 phút / 24 ảnh batch). Tiết kiệm ~44.5% thời gian gán nhãn cho mỗi ảnh. Thiên lệch: đã quen quy tắc và cảnh phố châu Á từ tập gold nên tốc độ duyệt nhanh hơn.
- Kết luận: **CÓ ĐÁNG DÙNG**, dựa trên tỷ lệ chấp nhận 81.2% và thời gian giảm một nửa, nhưng AI chỉ được đóng vai trò người đề xuất bản nháp (*Drafting assistant*), con người bắt buộc phải rà soát độc lập theo quy trình nghiêm ngặt.
- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. *Quy tắc 15 giây mù*: Bắt buộc bấm Space ẩn toàn bộ nhãn máy trong 15 giây đầu để quét bằng mắt thường và vẽ trước các đối tượng nhìn thấy (kiểm tra bằng tỷ lệ `added` > 0).
  2. *Quy tắc phóng to 200%*: Phóng to tối thiểu 200% đối với các box có kích thước trung bình và nhỏ để kiểm tra độ khít viền cạnh (kiểm tra bằng tỷ lệ `edited` > 5%).
  3. *Quy tắc kiểm soát thời gian tập trung*: Nghỉ giải lao 2 phút sau mỗi 8 ảnh duyệt liên tục để tránh hiện tượng suy giảm chú ý thị giác (kiểm tra qua nhật ký thời gian theo đợt trong `time_log.csv`).

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| img_013.jpg | SAM 2.1 Base (`sam2.1_b.pt`) | 3 (2 pos, 1 neg) | Không gồm người lái (đã trừ bằng 1 click âm) | Mask ôm sát thân và mui xe (317.260 px), 1 polygon duy nhất | submission/masks/tuktuk_sam2_img013.jpg |
| img_013.jpg | MobileSAM (`mobile_sam.pt`) | 3 (2 pos, 1 neg) | Không gồm người lái | Mask bị phân mảnh thành 2 polygon (245.517 px), mất chi tiết đuôi xe | submission/masks/tuktuk_mobilesam_img013.jpg |
| img_004.jpg | SAM 2.1 Base (`sam2.1_b.pt`) | 2 pos | Chỉ gồm xe máy, không gồm người ngồi trên | Mask ôm trọn bánh xe và khung xe (92.070 px), 1 polygon | submission/masks/motorbike_sam2_img004.jpg |

- Đối chiếu guideline mục 4.2: Quy tắc tách người lái thành box `person` riêng và xe thành box riêng được SAM hỗ trợ rất tốt thông qua điểm âm (*Negative prompt point*); tuy nhiên guideline chưa hướng dẫn rõ việc loại trừ bóng đổ dưới gầm xe khiến SAM có xu hướng lem mask xuống mặt đường.
- *(Tuỳ chọn)* Interactor SAM trong CVAT: Script offline giúp kiểm soát chính xác vị trí tọa độ pixel và đổi phiên bản trọng số mô hình linh hoạt hơn, trong khi CVAT Interactor tương tác trực quan nhanh hơn cho từng ca đơn lẻ.

## 6. Calibration (Part 6)

- Số quyết định: **4 quyết định** trong `submission/qc/calibration_decisions.csv`; quan trọng nhất:
  - `CAL-01`: Quy ước xe đạp điện / xe máy điện thống nhất gán vào lớp `motorbike` vì có động cơ và lưu thông chung làn xe cơ giới.
  - `CAL-02`: Box xe máy chở hàng cồng kềnh ôm trọn cả xe và khối hàng hóa gắn cố định trên giá để phục vụ cảnh báo va chạm xe tự hành.
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`: Đã cập nhật mục 1 và mục 2 của bản hợp đồng nhãn v1.1 kèm hình ảnh minh chứng.
- Nguyên nhân gốc: Do bản guideline v1.0 cố ý để ngỏ các tình huống thực tế địa phương (*Guideline gaps*) và do mô hình nhận diện đóng COCO thiếu nhãn phương tiện đặc thù châu Á (*Model taxonomy gap*).

## 7. QC và quyết định (Part 6)

- Mẫu: **6 ảnh** (1 ảnh `can_review` + 5 ngẫu nhiên, seed 42); người kiểm: Lê Văn Khởi (**hạn chế**: tự kiểm tra lại bài của chính mình, tiềm ẩn nguy cơ thiên kiến xác nhận).
- Lỗi theo **đối tượng**: **1/102 = 1.0%** (mẫu số: 102 đối tượng trong mẫu) · theo **ảnh**: **1/6 = 16.7%** (mẫu số: 6 ảnh trong mẫu, ngưỡng Release ≤ 2%).
- Top lỗi: `sai_vi_tri_kich_thuoc` (1 lỗi tại `img_003.jpg` do box bị rộng và lệch viền từ bẫy loose).
- **Giới hạn mẫu nhỏ** (cận trên 95% trong `qc_report_auto.md`): Cận trên khoảng tin cậy 95% (Wilson score) của tỷ lệ ảnh lỗi thật lên tới **56%**. Do cỡ mẫu chỉ có 6 ảnh nên ngưỡng chất lượng 2% về mặt toán học không thể được chứng minh chắc chắn; đây là giới hạn thực tế của việc lấy mẫu kiểm toán nhỏ.
- Script kiểm được / không kiểm được: Script kiểm tra được định dạng, kích thước tối thiểu, trùng lặp nhãn và danh mục taxonomy; nhưng script không kiểm tra được box có ôm khít viền thực tế hay không và không phát hiện được nếu con người cùng bỏ sót vật với AI.

**Quyết định: REWORK**, ngày 2026-11-17. Lý do (đi theo Hold → Rework → Release):
- Guideline đã được làm rõ qua bản v1.1 (không có guideline-gap).
- Tuy nhiên, tỷ lệ ảnh có lỗi trong mẫu đạt **16.7% > 2%** (ngưỡng tối đa cho phép) và tỷ lệ bắt bẫy chỉ đạt **50.0% < 80%** (dưới ngưỡng tối thiểu 80%). Căn cứ theo cây quyết định chuẩn, lô dữ liệu chưa đủ độ tin cậy để Release.
- Việc cần làm tiếp: Tiến hành Rework lại toàn bộ 24 ảnh của tập batch, tuân thủ nghiêm ngặt checklist 4 bước (đặc biệt là bước ẩn nhãn máy để rà soát các vật bị AI bỏ sót) trước khi đóng gói phát hành chính thức.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài**: Sử dụng Antigravity AI Assistant làm kỹ sư Tech Lead đồng hành hỗ trợ giải thích kiến thức, gợi ý quy trình thực hiện, phân tích nguyên nhân lỗi và chạy các lệnh kiểm tra tự động; toàn bộ thao tác gán nhãn thủ công trên CVAT và quan sát hình ảnh đều do học viên trực tiếp thực hiện.
- Gold có bị nhiễm không? **HOÀN TOÀN KHÔNG BỊ NHIỄM**. 4 ảnh gold được tự tay gán nhãn 100% thủ công trên CVAT trước khi chạy bất kỳ mô hình AI nào.
- Máy / GPU / CVAT dùng; vướng mắc cài đặt: Windows 11, GPU NVIDIA CUDA, CVAT chạy trên Docker Desktop v2.74. Quá trình ban đầu cần chú ý gọi đúng python trong `.venv`.
- Dùng cache `data/cache/` thay vì tự chạy? **KHÔNG**, tự chạy toàn bộ các mô hình trên GPU nội bộ của máy.
- Lỗi / bug script đã gặp: Gặp lỗi thiếu module khi chưa kích hoạt venv ngoài terminal PowerShell; đã khắc phục bằng cách trỏ trực tiếp đường dẫn thực thi `.venv\Scripts\python.exe`.
- Một điều bạn làm khác lần sau: Sẽ luôn ẩn nhãn máy trước khi xem ảnh để tránh bị bẫy bỏ sót do tâm lý phụ thuộc vào AI.
