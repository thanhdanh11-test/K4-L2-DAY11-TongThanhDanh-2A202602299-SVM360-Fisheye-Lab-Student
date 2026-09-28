# Zone table (slice của bạn)

Lệnh `python3 lab11.py model` tự ghi bảng số (cùng cách đếm với `r1_craft/compare.md` và `model_compare.md`); chạy lại lệnh sẽ cập nhật bảng và giữ nguyên phần nhận xét. Bạn chỉ viết mục Nhận xét.

| Zone | n_ref | L missing | L spurious | M missing (`LR_noM` + `R_only`) | M thừa (`LM_noR` + `M_only`) | Lỗi L chính (`what`) |
|---|---:|---:|---:|---:|---:|---|
| center | 4 | 0 | 2 | 1 | 4 | SPURIOUS (2) |
| mid | 13 | 3 | 4 | 3 | 7 | SPURIOUS (2) |
| edge | 0 | 0 | 0 | 0 | 1 | — |

## Nhận xét

Trong bảng, L là nhãn người học, M là model (AI nhận diện tự động), R là reference (bản nhãn mẫu của giảng viên). Box là khung chữ nhật bao vật. `n_ref` là số khung mẫu; missing là thiếu khung ghép được; spurious là khung không ghép được với mẫu. Zone là vùng ảnh: center ở tâm, mid ở giữa, edge ở rìa.

- Zone nào người (L) và model (M) gãy nhiều nhất, dẫn số ở bảng trên: **mid** có nhiều chênh lệch nhất nếu đếm số khung.
  - L ở mid thiếu 3 và thừa 4 so với 13 khung mẫu.
  - M ở mid thiếu 3: hai ThreeWheeler (xe ba bánh) ở 261480, L4+R5 và L5+R6, thuộc `LR_noM` (L và R có, M không ghép được); một Pedestrian (người đi bộ) ở 265065 R7 thuộc `R_only` (chỉ R có). M thừa 7 khung ở mid.
  - Center: L thừa 2 khung, 265065 L7 và L9, còn chờ phân xử. M thiếu 1 xe ba bánh, 261480 L7+R4. M thừa 4: 249480 M2/M4 tách người ngồi trên xe; 261480 M10/M12 gọi xe ba bánh thành Car/Truck (ô tô con/xe tải).
  - Edge: không có khung mẫu, chỉ có một khung M thừa, 249480 M6, ở người ngồi trên thùng xe tải. Một ca này không đủ để đánh giá chất lượng ở rìa ảnh.
- Giả thuyết vì sao (méo fisheye, box lỏng, thiếu `ego_body`, ...) và giới hạn của slice ba frame:
  - Các ca đã xác định ở L liên quan tới đọc luật và vẽ phần nhìn thấy: 261480 L6 gọi van là Truck thay vì Car (R04); 265065 L10 gộp hai người (R02); 261480 L3 bao cả phần ô tô bị che dù phần nhìn thấy ước tính dưới 40 pixel (R01). Chưa có phép đo tách riêng ảnh hưởng của fisheye (ống kính mắt cá, ảnh cong ở rìa), nên không loại trừ hoàn toàn ảnh hưởng này.
  - 265065 L1–L3 là ba xe mà L vẽ riêng nhưng R bỏ qua bằng `crowd_or_group` (cụm không tách được từng vật). Đây là bất đồng về cách áp dụng R06.
  - M không ghép đúng loại cho 3/3 xe ba bánh, tách người ngồi trên xe hai bánh thành Pedestrian ở 4 ca, và có khung trùng. Đây là dấu hiệu cần kiểm tra thêm, chưa chứng minh đầu ra của AI không có loại ThreeWheeler.
  - Bảng không báo thêm lỗi `ego_body` (phần xe gắn camera) hoặc `lens_border` (vành đen). Điều này không chứng minh các vùng bỏ qua đều đúng; bảng chủ yếu so box.
  - Chỉ có 3 ảnh từ một camera. Mẫu có 4 vật ở center và 0 ở edge. Không suy rộng rằng mid luôn khó nhất cho SVM (hệ thống quan sát quanh xe bằng nhiều camera).
  - Zone tính theo bán kính tới tâm vòng kính, không phải khoảng cách vật tới xe. Bản mẫu dùng để dạy cũng có thể thiếu vật; 261480 L2 đang được chuyển người soát kiểm tra theo `E0_reference_defect` (nghi lỗi bản mẫu).
