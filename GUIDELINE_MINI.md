# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Nhóm / tên: `Nguyễn Như Quỳnh`
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

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của nhóm | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | Giữ nguyên ID nếu bị che **dưới 25 frame** | Hệ thống vẫn nội suy được quỹ đạo, giúp giữ tính liên tục của dữ liệu. |
| Xe bị che lâu hơn ngưỡng trên | Ngắt track cũ, cấp **ID mới** | Sai số dự đoán quá lớn, dễ nhầm ID xe khác; thà đếm dư còn hơn làm sai luồng. |
| Xe rời khung hình rồi quay lại | Mặc định: **track mới** | Khó khẳng định 100% là xe cũ hay xe giống hệt (chu kỳ sống đã hết). |
| Hai xe cắt nhau / chồng lên nhau | Duy trì ID theo quỹ đạo cũ. Nếu chồng chéo > 25 frame thì cấp ID mới | Bbox bị đè gây nhiễu, cần dùng lịch sử (hướng đi, vận tốc) phân giải để tránh bị tráo ID. |

## 3. Luật bbox

| Tình huống | Luật của nhóm |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | bbox chạm đúng rìa, không đoán phần ngoài ảnh |
| Xe bị xe khác che một phần | bbox ôm phần **nhìn thấy được** |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | bắt đầu track từ frame đầu tiên xác định được là xe bốn bánh; ngưỡng nhóm chọn: Bbox phải đạt kích thước tối thiểu 20x20 pixel|
| Xe đang đỗ, không di chuyển | Vẫn gán bbox và giữ nguyên ID liên tục cho đến khi xe rời đi. |
| Keyframe đặt dày ở đâu | Keyframe phải được đặt dày (khoảng cách giữa các frame ngắn lại, ví dụ 3-5 frame/key) tại các thời điểm xe chuyển hướng đột ngột, tăng/giảm tốc độ gấp, hoặc bắt đầu / kết thúc quá trình bị che khuất. |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung.

### Ca 1 (Dựa trên lỗi Ghost Track thực tế)
- Clip / frame / ID: `clip_01` / Frame 75-78 / Pred ID 5
- Tình huống: Bắt đầu vẽ ID 5 quá sớm (trước khi track ground-truth xuất hiện). Lúc này xe có thể còn quá nhỏ, mờ, hoặc đang bị che khuất.
- Quyết định: Xóa (bỏ gán) các frame 75-78 của ID 5.
- Lý do: Theo luật mục 3, chỉ bắt đầu track khi xác định rõ là xe 4 bánh và bbox đạt kích thước tối thiểu 20x20 pixel.

### Ca 2 (Dựa trên lỗi Partially Covered)
- Clip / frame / ID: `clip_01` / Track ID 6 (tổng độ dài GT là 56 frames)
- Tình huống: Nhãn vẽ bị thiếu, chỉ bao phủ được 62% (35/56 frames) quỹ đạo thật của xe (có thể do xe bị khuất một phần nên người gán bỏ vẽ sớm hoặc nối thiếu ID).
- Quyết định: Vẽ bù phần bị thiếu, giữ ID 6 và nội suy xuyên qua đoạn bị khuất.
- Lý do: Theo luật mục 2, nếu xe bị che một phần, vẫn phải tiếp tục nội suy quỹ đạo và duy trì track liên tục.

### Ca 3 (Dựa trên lỗi Loose Box)
- Clip / frame / ID: `clip_01` / Frame 11 / Pred ID 1 (GT ID 2)
- Tình huống: Bbox vẽ quá lỏng lẻo (IoU với viền thực tế chỉ đạt ~0.525). Nguyên nhân có thể do vẽ lấn sang vùng bóng râm dưới mặt đường hoặc do xe đang bị xe khác che một phần.
- Quyết định: Bóp chặt viền bbox lại, cắt bỏ phần thừa.
- Lý do: Theo luật mục 3, bbox phải ôm vừa khít phần "nhìn thấy được" của xe, không đoán rộng ra ngoài hoặc vẽ bao lấy bóng xe.

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Luật nào trong file này hoá ra còn thiếu hoặc còn mơ hồ? Viết lại cho rõ:

- **Luật về kích thước khởi tạo và kết thúc track (gây lỗi Ghost Track / Vẽ thiếu):** Ban đầu việc xác định thời điểm "bắt đầu/kết thúc" quá mơ hồ. **Viết lại cho rõ:** "Chỉ bắt đầu gán bbox khi xe lộ diện đủ rõ ràng và kích thước bbox đạt tối thiểu 20x20 pixel. Tương tự, tiếp tục duy trì track (kể cả khi nội suy qua vật cản) cho đến khi xe rời hẳn khung hình hoặc nhỏ hơn 20x20 pixel mới được ngắt, không tự ý dừng gán sớm dựa trên cảm tính."
- **Luật về ranh giới Bbox và bóng đổ (gây lỗi Loose Box làm giảm IoU):** Quy định "ôm phần nhìn thấy được" trước đây gây hiểu nhầm khi xe có bóng râm hắt xuống mặt đường. **Viết lại cho rõ:** "Bounding box phải bám khít vào viền vật lý của phương tiện (khung kim loại, cản trước/sau, mép ngoài lốp xe). Tuyệt đối không được kéo giãn bbox bao trùm cả phần bóng đổ (shadow) của xe, vì sẽ làm sai lệch diện tích hộp và giảm nghiêm trọng độ chính xác vị trí (LocA / IoU)."
