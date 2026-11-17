# Thông tin học viên

- Họ và tên: Lê Văn Khởi
- MSHV: 03290
- Lớp (L2 / L3 / khác): L23
- Tài khoản GitHub: khoivancw
- Repo bài nộp (URL): https://github.com/khoivancw/K5-L23-Day08-03290-LeVanKhoi-AI-Assisted-Labeling
- Commit chốt bài (hash): HEAD
- Máy: Windows 11, GPU CUDA
- CVAT (Docker): v2.74.1

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 367/452 = 81.2% / 26/452 = 5.8% / 58/452 = 12.8% (mẫu số: 452 box máy); added: 59/452 = 13.1% (mẫu số: 452 đối tượng sau duyệt)
- So với gold: precision / recall của **máy** 49.2% / 54.5% (mẫu số: 55 đối tượng gold)
- Bắt bẫy: 6/12 = 50.0%
- Thời gian: gán tay 3.75 phút/ảnh; duyệt có pre-label 2.08 phút/ảnh
- Lỗi QC: 16.7% theo ảnh (1/6 ảnh), 1.0% theo đối tượng (1/102 đối tượng)
- **Quyết định:** REWORK — lý do một dòng: Tỷ lệ ảnh có lỗi đạt 16.7% > 2% và bắt bẫy 50.0% < 80% theo đúng cây quyết định kiểm toán chất lượng.
