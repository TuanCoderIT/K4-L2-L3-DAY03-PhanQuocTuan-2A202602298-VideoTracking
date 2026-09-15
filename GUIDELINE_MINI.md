# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Nhóm / tên: Phan Quốc Tuấn (Nhóm 018)
Clip: `clip_01`, `clip_02`

---

## 1. Phạm vi: gán cái gì, không gán cái gì

Một lớp duy nhất: **`vehicle`** — xe bốn bánh (xe con, van, xe buýt, xe tải).

| Gán | Không gán |
| --- | --- |
| xe con, SUV, taxi, xe bán tải | người đi bộ |
| van, minivan | xe đạp |
| xe buýt, minibus | **xe máy / mô tô** |
| xe tải, xe đầu kéo | xe trong ảnh quảng cáo, trong gương, dưới bóng nước |

Bổ sung của nhóm (nếu có): Gán cả xe ba bánh chuyên chở hàng hóa cỡ lớn có kết cấu khung cabin giống xe tải; không gán xe đẩy hàng thủ công.

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của nhóm | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | giữ nguyên ID nếu bị che **dưới 25 frame** (mặc định của lab: 25 frame = 2 giây @ 12.5 fps) | Bảo toàn tính liên tục của đối tượng (Association) theo chuẩn đánh giá MOT |
| Xe bị che lâu hơn ngưỡng trên | Tách thành track/ID mới khi xe xuất hiện lại | Tránh gán nhầm identity khi đối tượng bị mất dấu quá lâu và quỹ đạo bị biến đổi |
| Xe rời khung hình rồi quay lại | mặc định: **track mới** | Không thể xác định chắc chắn bằng thị giác nếu không có ReID chuyên sâu |
| Hai xe cắt nhau / chồng lên nhau | Xe phía trước giữ nguyên bbox và ID; xe phía sau chỉ bbox phần hở, duy trì ID cũ | Đảm bảo nguyên tắc bbox ôm phần nhìn thấy và giữ vững identity |

## 3. Luật bbox

| Tình huống | Luật của nhóm |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | bbox chạm đúng rìa, không đoán phần ngoài ảnh |
| Xe bị xe khác che một phần | bbox ôm phần **nhìn thấy được** |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | bắt đầu track từ frame đầu tiên xác định được là xe bốn bánh; ngưỡng nhóm chọn: `kích thước chiều ngang >= 12px` |
| Xe đang đỗ, không di chuyển | Duy trì một bbox cố định bằng cách khóa keyframe từ frame đầu đến frame cuối di chuyển |
| Keyframe đặt dày ở đâu | Đặt dày (mỗi 2-3 frame) tại các vùng xe quay đầu, tăng/giảm tốc nhanh hoặc chuyển hướng góc nhìn |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung.

### Ca 1
- Clip / frame / ID: `clip_01` / Frame 85–115 / ID 5
- Tình huống: Xe bị lùm cây ở dải phân cách che mất hơn 60% thân xe trong 15 frame liên tiếp.
- Quyết định: Tiếp tục giữ nguyên ID 5 và vẽ bbox thu nhỏ chỉ ôm lấy phần đầu/đuôi xe nhô ra khỏi lùm cây.
- Lý do: Thời gian che khuất < 25 frame và vẫn quan sát được chuyển động liên tục của một phần thân xe.

### Ca 2
- Clip / frame / ID: `clip_01` / Frame 108 / ID 6
- Tình huống: Xe xuất hiện ở xa ở rìa trên khung hình, kích thước chỉ khoảng 10x12 pixel, rất mờ.
- Quyết định: Bắt đầu đánh dấu ID 6 ngay từ Frame 108 thay vì đợi đến Frame 117 khi xe lại gần.
- Lý do: Khi tua tiến/lùi đã xác định chắc chắn khối pixel đó là phương tiện chuyển động tiến vào dải quan sát.

### Ca 3
- Clip / frame / ID: `clip_01` / Frame 125–134 / ID 7
- Tình huống: Xe di chuyển chéo góc làm kích thước bounding box biến đổi phi tuyến tính giữa các keyframe.
- Quyết định: Thêm 3 keyframe bổ sung tại các frame 127, 130, 133 thay vì chỉ dựa vào interpolation tự động từ frame 125 đến 134.
- Lý do: Tránh lỗi loose box (bbox bị rộng ra ngoài thân xe do sai lệch nội suy tự động).

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Luật nào trong file này hoá ra còn thiếu hoặc còn mơ hồ? Viết lại cho rõ:

- **Bổ sung quy định Start Entry:** Bắt buộc tua thử lượt 0 (tua lùi từ giữa clip về đầu) để phát hiện điểm xuất hiện sớm nhất của các xe ở xa ranh giới ảnh, tránh bỏ sót frame đầu (tối ưu FN).
- **Rõ ràng hóa luật Loose Box:** Khi xe chuyển hướng hoặc thay đổi góc nhìn so với camera, khoảng cách giữa các keyframe không được vượt quá 4 frame để tránh sai lệch IoU (< 0.60).