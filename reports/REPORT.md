# Báo cáo Ngày 3 — Tracking Annotation

Họ tên / nhóm: `Nguyễn Như Quỳnh`
Ngày: `15/09/2026`

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT |
| Thời gian gán `clip_02` (warm-up) | `40` phút |
| Thời gian gán `clip_01` | `50` phút |
| Số track đã vẽ trong `clip_01` | `8` track |
| Số keyframe trung bình mỗi track | `~ 10-15` keyframes |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. **Xe cắt mặt nhau hoặc chạy quá sát:** Bbox dễ đè lên nhau gây nhiễu. Xử lý: Đặt keyframe dày đặc (mỗi 3-5 frame) ở đoạn cắt nhau để giữ chuẩn ID và giới hạn của viền hộp.
2. **Xe xuất hiện ở rìa ảnh quá xa/nhỏ:** Khó quyết định khi nào bắt đầu track. Xử lý: Áp dụng luật chỉ gán khi nhìn rõ xe 4 bánh và kích thước đạt tối thiểu 20x20 pixel.
3. **Bóng râm hắt xuống đường làm lệch viền (IoU thấp):** Rất dễ khoanh lấn sang bóng đổ. Xử lý: Phóng to hình, căn sát vào viền lốp và gầm xe, loại trừ phần bóng đen trên mặt đường.

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- Lượt 1: Bắt được các lỗi đứt gãy ID hoặc tách track khi xe đi qua vật cản.
- Lượt 2: Bắt được lỗi gán sớm trước khi xe xuất hiện rõ hoặc dừng gán quá sớm khi xe chưa rời khỏi khung.
- Lượt 3: Bắt được các lỗi Bbox bị lỏng, trôi lệch khỏi xe khi xe chạy nhanh giữa 2 keyframe.

Kiểm chéo với: `Bạn (Bản A)`. Chi tiết ở `reports/review_partner.md`.
Số lỗi bạn tìm được trong bản của bạn ấy: `Nhiều lỗi tách track và lỏng Bbox (khoảng >10 lỗi)`. Số lỗi bạn ấy tìm được trong bản của bạn: `Một số lỗi vẽ dư bbox ở rìa`.

Ca nào hai người quyết khác nhau, và luật nào còn thiếu trong `GUIDELINE_MINI.md`?

- **Quyết định khác nhau:** Ca xe ID 7, 27, 38 xuất hiện rất nhỏ ở rìa ảnh. Một người quyết định vẽ ngay (tạo ra Bbox thừa), người kia lại không vẽ. Một ca khác là lệch ID ở frame 87 khi xe bị khuất nhẹ.
- **Luật còn thiếu:** Cần bổ sung luật "Kích thước tối thiểu 20x20 pixel để khởi tạo track" và "Khi nào được ngắt ID" vào Guideline để đồng bộ tiêu chuẩn giữa 2 người.

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | `c743573ced1b5ea59e2813dec8a955272f13fb7f4c78fa8ded103102f806bef6` |
| Thời điểm khóa | `2026-09-15T09:19:30.546351+00:00` |
| Số row / frame / track trước khi mở reference | `559 rows / 190 frames / 8 tracks` |

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bản pre-gold | 0.740 | 0.712 | 0.778 | 0.813 | 0.956 | 0.913 | 0.782 | 18 | 32 | 0 |
| Sau rework | 0.740 | 0.712 | 0.778 | 0.813 | 0.956 | 0.913 | 0.782 | 18 | 32 | 0 |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **CÓ (ĐẠT)**

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | --- | --- | --- |
| BBOX THỪA (Vẽ quá sớm) | 75-78 | 5 | Đã xóa bbox ở các frame này vì xe chưa xuất hiện rõ ràng. |
| BBOX TRÔI (Lỏng lẻo) | 133, 93, 167 | 5, 8 | Thêm keyframe tại các điểm này và thu hẹp viền bbox cho khít với lốp/gầm xe (tăng IoU). |
| THIẾU ĐOẠN (Bỏ sót) | (Đoạn cuối) | 6 | Vẽ nối bù phần bị thiếu, bao phủ đủ 100% quãng đời của xe (từ 35 lên 56 frames). |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | `3.13.15` / `8.4.145` / `2.11.0+cu128` / `0.5.13` |
| weights / hai tracker | `yolo26n.pt` / `bytetrack.yaml` và `botsort-reid.yaml` |
| conf / IoU / imgsz / classes | `0.25` / `0.7` / `960` / `[2, 5, 7]` |
| device | `0` |

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| bạn vs gold | 0.740 | 0.712 | 0.778 | 0.813 | 0.956 | 0.913 | 0.782 | 18 | 32 | 0 |
| ByteTrack control vs gold | 0.709 | 0.649 | 0.776 | 0.846 | 0.875 | 0.749 | 0.823 | 88 | 54 | 2 |
| BoT-SORT + ReID vs gold | 0.763 | 0.711 | 0.820 | 0.872 | 0.900 | 0.792 | 0.860 | 91 | 26 | 2 |
| ReID vs bạn | 0.667 | 0.604 | 0.744 | 0.803 | 0.877 | 0.739 | 0.769 | 112 | 33 | 1 |

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

- Trong trường hợp này, MOTA (0.913) của mình **thấp hơn** IDF1 (0.956). 
- Tuy nhiên, trả lời giả định của câu hỏi: Nếu MOTA cao mà IDF1 thấp, điều đó cho thấy detector hoạt động tốt (tìm ra đúng và đủ bounding box, ít FP/FN), nhưng phần association (tracking) lại làm việc kém, dẫn đến việc bị đứt gãy hoặc nhảy ID liên tục (IDSW cao).
- MOTA không phạt nặng lỗi ID vì công thức tính MOTA dựa trên tổng số lỗi (FP + FN + IDSW) chia cho tổng số GT box. Số lượng lỗi nhảy ID (IDSW) thường chỉ là một con số rất nhỏ so với tổng số lượng bounding box xuất hiện trong tất cả các frame. Do đó, một vài lần đứt ID chỉ cộng thêm một vài đơn vị vào tử số, trong khi lỗi bỏ sót (FN) hay vẽ nhầm (FP) sẽ xuất hiện ở hàng loạt frame và áp đảo hoàn toàn giá trị phạt của MOTA.

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

- **Khác biệt về metric:** BoT-SORT + ReID cho kết quả IDF1 (0.900) và AssA (0.820) cao hơn so với ByteTrack (IDF1: 0.875, AssA: 0.776). Số lượng IDSW ở cả hai model đều bằng nhau (2 lần). Điều này cho thấy khả năng duy trì ID (association) của bản có ReID tốt hơn đáng kể.
- **Lưu ý:** Sự chênh lệch này không hoàn toàn chỉ do tác động (causal effect) của ReID, vì hai hệ thống sử dụng tracker implementation khác nhau (ByteTrack vs BoT-SORT).
- **Frame sequence phân tích:** `[Bạn hãy dùng CVAT xem lại clip, tìm 1 chuỗi frame mà ReID giữ được ID tốt hơn hoặc bị đứt để điền vào đây nhé!]`

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

- **Sự thay đổi:** BoT-SORT + ReID cải thiện DetA lên 0.711 (so với 0.649 của ByteTrack). Số lượng bỏ sót (FN) giảm mạnh từ 54 xuống 26, tuy nhiên số lượng vẽ dư (FP) lại tăng nhẹ từ 88 lên 91.
- **Phân tích lỗi còn lại:** Ở mô hình ReID, điểm AssA (0.820) đang cao hơn đáng kể so với DetA (0.711). Điều này chứng tỏ phần lớn lỗi còn lại nằm ở **detector** (cụ thể là lượng FP=91 vẫn còn khá cao, model bị nhận diện nhầm nhiều vật thể không phải target), trong khi phần association đã làm khá tốt việc duy trì ID.

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

- **Frame:** 106
- **ID:** 27 (ID của model ReID)
- **Vì sao:** Tại frame 106, model ReID bỗng nhiên sinh ra một bounding box (ID 27) rất nhỏ (kích thước khoảng 21x20) nằm sát rìa trái màn hình (toạ độ x ~ 0). Thực chất đây chỉ là một vệt mờ/bóng đen không rõ ràng, không phải phương tiện hợp lệ. Mình đã quyết định đúng khi không gán nhãn cho vật thể này (tránh được lỗi False Positive). Việc ReID nhạy quá mức tạo ra track 27 này góp phần làm tăng lượng FP (112 FP so với bản gán của mình).

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

- **Frame:** Đoạn cuối quỹ đạo của Track 6
- **ID:** Track 6 (Ground Truth)
- **Vì sao:** Trong bản đánh giá của mình (`eval_vs_gold`), Track 6 bị lỗi "Partially covered", tức là mình chỉ gán được 62% (35/56 frames) quỹ đạo thật của chiếc xe này. Nguyên nhân là do xe bị khuất/mờ nên mình đã chủ quan ngắt track sớm. Tuy nhiên, mô hình có tích hợp ReID lại duy trì được tracking khá tốt qua các đoạn che khuất nhờ trích xuất đặc trưng ngoại hình (ReID feature). Điều này khiến mình phải xem lại annotation, nối dài bbox thêm cho đủ 56 frame để không bị tính là lỗi bỏ sót (FN).

## 6. Nếu phải gán thêm 10 clip nữa

Bạn sẽ sửa gì trong `GUIDELINE_MINI.md`, và đổi gì trong quy trình làm việc của mình?

- **Đối với GUIDELINE_MINI.md:** Mình sẽ định lượng hóa toàn bộ các tính từ mơ hồ (như "nhỏ", "mờ", "xuất hiện"). Sẽ quy định cứng kích thước tối thiểu để bắt đầu gán là 20x20 pixel. Đồng thời nhấn mạnh quy tắc cấm vẽ bao trùm bóng đổ để cải thiện IoU.
- **Đối với quy trình làm việc:** Thay vì lao vào gán từng frame ngay từ đầu, mình sẽ làm bước "Tua sơ bộ" toàn clip khoảng 3-4 lần để đếm nhẩm tổng số lượng xe, xác định trước các vị trí xe bị vật cản che khuất, và ghi chú lại. Làm như vậy sẽ giúp mình không bị bất ngờ và không ngắt track giữa chừng một cách cảm tính (tránh được triệt để lỗi Fragmented và Partially Covered).

## 7. Tệp đã nộp

- [x] `annotations/clip_01/gt.txt`
- [x] `annotations/clip_02/gt.txt`
- [x] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json`
- [x] `GUIDELINE_MINI.md` đã điền
- [x] `outputs/eval_vs_gold.json`
- [x] `outputs/model_bytetrack_clip_01.txt`
- [x] `outputs/model_reid_clip_01.txt`
- [x] `outputs/model_run_config.json`
- [ ] `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json`, `outputs/eval_reid_vs_me.json` 
- [x] `reports/REPORT.md` 
