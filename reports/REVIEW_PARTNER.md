# Peer review — Day 3

Chép file này thành `reports/review_partner.md`. Reviewer chỉ ghi finding; tác giả
tự sửa bài của mình và điền closure.

| Trường | Giá trị |
| --- | --- |
| Author | Phan Quốc Tuấn |
| Reviewer | Nguyễn Xuân Huỳnh |
| Pair ID | Pair-01 |
| CVAT version | 2.15.0 |
| Thời điểm review | 2026-09-15 14:30:00 |

## Danh sách finding

Mỗi finding bắt buộc có `frame + ID + lỗi + sửa thế nào`. Nếu chưa thống nhất,
dùng `needs-review`; không ép tác giả sửa theo cảm tính.

| # | CVAT frame | MOT frame | ID | Loại lỗi | Quan sát + rule áp dụng | Cách sửa đề xuất | Closure: fixed / not-a-defect / needs-review |
| ---: | ---: | ---: | ---: | --- | --- | --- | --- |
| 1 | 125-128 | 125-128 | 7 | Loose box | Bbox track 7 của tác giả bị lệch IoU (< 0.6) so với track 6 của Huỳnh | Điều chỉnh lại kích thước bbox ôm sát viền xe hơn | fixed |
| 2 | 85-115 | 85-115 | 6 | Partial coverage / Short track | Track 6 của tác giả chỉ bao phủ 29/79 frame (37%) so với bản gán của Huỳnh | Kéo dài track 6 để phủ trọn vẹn sự xuất hiện của xe ở góc xa | fixed |
| 3 | 90-120 | 90-120 | 5 | Partial coverage / Short track | Track 5 của tác giả chỉ bao phủ 31/79 frame (39%) so với bản gán của Huỳnh | Bổ sung keyframe đầu và cuối cho track 5 | fixed |

## Reviewer checklist

| Hạng mục | PASS / FINDING / N/A | Frame–ID–evidence |
| --- | --- | --- |
| Có tối thiểu 6 track hợp lệ; chỉ gồm xe bốn bánh | PASS | Đủ 8 track phương tiện bốn bánh |
| Một xe giữ một ID; không reuse ID cho xe khác | PASS | Không phát hiện IDSW (IDSW = 0) |
| Occlusion ngắn giữ ID; crossing không đổi ID | PASS | Các track duy trì ID tốt qua khoảng che |
| Entry/exit đúng; không box treo sau khi xe rời khung | FINDING | Các track 4, 5, 6, 8 của tác giả bắt trễ và nhả sớm hơn bản của Huỳnh |
| Bbox ôm phần nhìn thấy, không đoán phần bị che/ngoài khung | FINDING | Loose box ở frame 125-128 (ID 7) |
| Frame giữa hai keyframe không bị interpolation drift | PASS | Khung hình nội suy mượt mà |
| Export đúng MOT 1.1; frame bắt đầu từ 1; cột 2 là track ID | PASS | Định dạng `gt.txt` khớp chuẩn MOT 1.1 |
| Mọi finding có cách sửa và closure do tác giả điền | PASS | Đã hoàn thành điền closure cho cả 3 findings |

## Self-QC attestation của reviewer

| Lượt | PASS / ĐÃ SỬA / NEEDS-REVIEW | Frame–ID–evidence |
| --- | --- | --- |
| 1 — identity/timeline | PASS | Tính nhất quán ID giữa hai bản đạt IDF1 = 85.79% |
| 2 — endpoint/scope | ĐÃ SỬA | Đã mở rộng độ phủ các track 4, 5, 6, 8 theo bản gán tham chiếu của Huỳnh |
| 3 — geometry/interpolation | ĐÃ SỬA | Đã chỉnh lại bbox loose tại các frame 125-128 |

## Exit ticket

1. Finding quan trọng nhất và rule dùng để kết luận: Finding 2 & 3 về độ phủ track (Partially covered tracks) - Quy tắc bắt đối tượng ngay từ frame đầu tiên xác định được xe bốn bánh.
2. Một finding tác giả đóng là `not-a-defect`, kèm lý do (nếu có): Không có (tác giả thống nhất với các vị trí chênh lệch độ phủ do Huỳnh chỉ ra).
3. Một rule cần Lab Coach làm rõ (nếu có): Quy định cụ thể về kích thước pixel tối thiểu ở khoảng cách xa để hai người gán nhãn thống nhất mốc Entry/Exit.