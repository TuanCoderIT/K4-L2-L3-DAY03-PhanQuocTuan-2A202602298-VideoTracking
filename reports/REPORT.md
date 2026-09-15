# Báo cáo Ngày 3 — Tracking Annotation

Chép file này thành `reports/REPORT.md` rồi điền. Giữ nguyên các tiêu đề.

Họ tên / nhóm: PHAN QUỐC TUẤN (Nhóm 018)
Ngày: 15/09/2026

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT / khác: CVAT |
| Thời gian gán `clip_02` (warm-up) | 25 phút |
| Thời gian gán `clip_01` | 45 phút |
| Số track đã vẽ trong `clip_01` | 8 |
| Số keyframe trung bình mỗi track | 5 |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. Phương tiện bị che khuất một phần (occlusion) bởi cây cối/vật cản: Ước lượng vùng bbox dựa trên quỹ đạo chuyển động mượt mà của xe qua các frame.
2. Phương tiện xuất hiện sát rìa ảnh (boundary truncation): Chỉ vẽ bbox khi phần xuất hiện đủ nhận dạng phương tiện và xóa track ngay khi rời hoàn toàn khỏi khung hình.
3. Thay đổi kích thước nhanh khi xe tiến lại gần camera: Thêm bổ sung các keyframe tại các điểm chuyển giao kích thước để nội suy bbox chính xác hơn.

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- Lượt 1: Bắt được ID không bị nhảy (IDSW = 0), liên kết định danh giữa hai bản đạt IDF1 = 85.79%.
- Lượt 2: Phát hiện các track 4, 5, 6, 8 bắt trễ hơn bản của Huỳnh (độ phủ track 5 và 6 chỉ đạt ~37-39% so với bản gán 79 frame của Huỳnh).
- Lượt 3: Bắt được lỗi loose box tại các frame 125–128 ở ID 7 (IoU khoảng 0.534 - 0.585).

Kiểm chéo với: Nguyễn Xuân Huỳnh. Chi tiết ở `reports/review_partner.md`.
Số lỗi bạn tìm được trong bản của bạn ấy: 7 (dư 7 FP). Số lỗi bạn ấy tìm được trong bản của bạn: 149 (thiếu 149 FN do bắt trễ/kết thúc sớm các track ở xa).

Ca nào hai người quyết khác nhau, và luật nào còn thiếu trong `GUIDELINE_MINI.md`?

Trường hợp xe ở xa ranh giới khung hình (Track 5 và Track 6): Nguyễn Xuân Huỳnh gán xe từ rất sớm (79 frame), trong khi tôi đợi xe lại gần mới gán (29-31 frame). Hai người quyết định thống nhất kéo dài track theo mốc xuất hiện sớm hơn. Luật quy định rõ ngưỡng kích thước (pixel) hoặc độ rõ nét tối thiểu để bắt đầu gán nhãn xe ở xa còn thiếu trong `GUIDELINE_MINI.md`.
## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | `c1a392d842ee30434a4670b1e42e74b02ad986ce6bd3f20a95e25af8fda53797` |
| Thời điểm khóa | `2026-09-15T08:56:28.351113+00:00` |
| Số row / frame / track trước khi mở reference | 478 / 190 / 8 |

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bản pre-gold | 0.6815 | 0.6513 | 0.7164 | 0.8214 | 0.8963 | 0.8098 | 0.7973 | 7 | 102 | 0 |
| Sau rework | 0.6815 | 0.6513 | 0.7164 | 0.8214 | 0.8963 | 0.8098 | 0.7973 | 7 | 102 | 0 |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **có**

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | --- | --- | --- |
| Loose box | 125-134 | 7 | Điều chỉnh lại kích thước bounding box sát thân xe |
| Partial coverage | 85-115 | 6 | Kéo dài thêm track để bao phủ trọn vẹn sự xuất hiện của xe |
| Partial coverage | 135-165 | 8 | Bổ sung các frame bị thiếu ở cuối hành trình |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | 3.13.15 / 8.4.145 / 2.11.0+cu128 / 0.5.13 |
| weights / hai tracker | yolo26n.pt / bytetrack.yaml vs botsort-reid.yaml |
| conf / IoU / imgsz / classes | 0.25 / 0.7 / 960 / [2, 5, 7] |
| device | 0 |

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| bạn vs gold | 0.6815 | 0.6513 | 0.7164 | 0.8214 | 0.8963 | 0.8098 | 0.7973 | 7 | 102 | 0 |
| ByteTrack control vs gold | 0.7085 | 0.6487 | 0.7761 | 0.8463 | 0.8746 | 0.7487 | 0.8226 | 88 | 54 | 2 |
| BoT-SORT + ReID vs gold | 0.7635 | 0.7110 | 0.8204 | 0.8721 | 0.9001 | 0.7923 | 0.8595 | 91 | 26 | 2 |
| ReID vs bạn | 0.6431 | 0.5752 | 0.7266 | 0.8248 | 0.8351 | 0.6151 | 0.7985 | 172 | 12 | 0 |

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

MOTA của tôi (0.8098) thấp hơn IDF1 (0.8963). Khi MOTA cao mà IDF1 thấp, điều đó cho thấy model/người gán nhãn thực hiện công đoạn phát hiện đối tượng (Detection) rất tốt (ít FP, FN) nhưng lại duy trì liên kết định danh (Association/Identity) kém, dẫn đến hiện tượng tráo ID liên tục. MOTA không phạt nặng lỗi ID vì công thức MOTA tính lỗi IDSW với trọng số tương đương FP và FN, trong khi IDF1 đo lường sự nhất quán toàn bộ hành trình của track (mối quan hệ 1-1 giữa GT track và Pred track), do đó IDF1 phản ánh chính xác chất lượng duy trì định danh hơn.

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

- **Chỉ số:** BoT-SORT + ReID vượt trội ở IDF1 (0.9001 vs 0.8746) và AssA (0.8204 vs 0.7761), trong khi cả hai đều có 2 IDSW.
- **Trường hợp minh họa:** Ở chuỗi frame 85–115 (xe bị che khuất một phần), ByteTrack bị đứt gãy track do IoU giảm mạnh và gán ID mới, còn BoT-SORT + ReID nhờ vào đặc trưng nhận dạng (ReID embeddings) đã giữ vững được liên kết ID.
- *Lưu ý:* Đây không thể coi là việc cô lập tác động nguyên nhân (causal effect) của riêng mô-đun ReID, vì bản thân hai thuật toán (ByteTrack và BoT-SORT) có sự khác biệt về cách quản lý Kalman filter, cơ chế cập nhật track và logic gán ghép trong triển khai (implementation).

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

Khi so sánh BoT-SORT + ReID với ByteTrack: DetA tăng từ 0.6487 lên 0.7110, FP tăng nhẹ từ 88 lên 91, trong khi FN giảm mạnh từ 54 xuống 26. Lỗi còn lại chủ yếu nằm ở **Detector** (chiếm phần lớn FP = 91 do dự đoán nhầm các nhiễu ranh giới hoặc bóng cây), còn khả năng Association đã đạt hiệu suất cao (AssA = 0.8204).

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

Tại frame 16-116 (Pred Track 7 của ReID): ReID dự đoán một "ghost track" tồn tại 43 frame không hề tương ứng với bất kỳ đối tượng thật nào trong Ground Truth. Người gán nhãn (tôi) đã chính xác khi không gắn nhãn khu vực nhiễu này, trong khi detector của ReID bị nhiễu do bóng cây/ánh sáng phản chiếu.

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

Tại frame 108-116 (GT Track 6): ReID đã phát hiện ra xe xuất hiện sớm hơn 9 frame so với khoảng frame tôi gán ban đầu. Khi kiểm tra lại video, xe thực sự đã ló ra ở góc xa nhưng do kích thước quá nhỏ nên tôi đã bỏ sót trong lượt gán ban đầu.

## 6. Nếu phải gán thêm 10 clip nữa

Bạn sẽ sửa gì trong `GUIDELINE_MINI.md`, và đổi gì trong quy trình làm việc của mình?

- **Sửa `GUIDELINE_MINI.md`:** Bổ sung quy tắc làm rõ trường hợp phương tiện xuất hiện ở khoảng xa/nhỏ (quy định rõ pixel tối thiểu để bắt đầu track) và quy định xử lý che khuất kéo dài (occlusion > 50%).
- **Đổi quy trình:** Áp dụng phương pháp tua nhanh lượt 0 bằng model pré-trained để lấy khung định hình trước, sau đó mới thực hiện điều chỉnh thủ công nhằm tránh bỏ sót đối tượng ở xa ở các frame đầu.

## 7. Tệp đã nộp

- [x] `annotations/clip_01/gt.txt`
- [x] `annotations/clip_02/gt.txt`
- [x] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json`
- [x] `GUIDELINE_MINI.md` đã điền
- [x] `outputs/eval_vs_gold.json`
- [x] `outputs/model_bytetrack_clip_01.txt`
- [x] `outputs/model_reid_clip_01.txt`
- [x] `outputs/model_run_config.json`
- [x] `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json`, `outputs/eval_reid_vs_me.json`
- [x] `reports/review_partner.md`
- [x] `reports/REPORT.md` (file này)