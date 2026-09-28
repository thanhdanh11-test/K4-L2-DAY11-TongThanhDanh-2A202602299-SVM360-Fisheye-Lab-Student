# Kế hoạch review từ lỗi quan sát được

Từ `findings.csv` và `zone_table.md`, chọn **hai lát cắt của bài ADASIND một camera** cần review (kiểm tra) trước. L là nhãn người học; R là reference (bản nhãn mẫu của giảng viên); M là model (AI nhận diện tự động). Box là khung chữ nhật bao vật. Zone center/mid/edge là vùng tâm/giữa/rìa ảnh. Frame là một ảnh; ThreeWheeler/Car/Truck là xe ba bánh/ô tô con/xe tải. `crowd_or_group` là cụm không tách được từng vật. Bảng này nói về ảnh thật đã làm; kế hoạch bốn camera giả lập nằm trong `45_sampling_plan.csv`.

| Lát cắt / frame | Số ca và loại lỗi | Vì sao review trước | Bằng chứng cần giữ |
|---|---|---|---|
| **Cảnh đô thị đông xe, zone mid**: `adasind_261480.jpg`, `adasind_265065.jpg` và các frame B*-dense/B*-mid tương tự | 261480: 1 WRONG_CLASS (sai loại) (van → Truck, R04), 1 box vượt ngưỡng giả (L3, R01), 1 ca nghi bản mẫu thiếu (L2, E0_reference_defect). 265065: 1 gộp 2 người (L10, R02), 3 box trong `crowd_or_group` (IGNORE_SCOPE (bất đồng phạm vi), P0), 2 ca chưa phân xử (L7, L9, E5_unresolved) | Mid có 13/17 vật mẫu và nhiều lỗi P0/P1 (ưu tiên xử lý phạm vi hoặc sửa vật). Không phải tất cả P1 đều ở mid: xe ba bánh 261480 L7+R4 ở center. R04 đã quy định rõ van; gọi van là Truck là lỗi áp dụng luật. R06 còn cần làm rõ khi nào bỏ qua cụm xe | Tệp nhãn đã khóa (`r1_craft`, mã 4C3B-1E1C) cùng `rework/annotations-v2.xml`, `local_quality_conflicts.csv`, ảnh bằng chứng 01/03, phiên bản luật (v1.0.0 → v1.1.0 nếu patch R06a được duyệt) |
| **Xe ba bánh và rider, mọi zone**: ThreeWheeler ở 261480 (L4, L5, L7) và rider ở 249480 (L2), 261480 (L1, L9) | Model: 3/3 ThreeWheeler bị đoán Car/Truck, 4 lần tách rider (người ngồi trên xe hai bánh) thành Pedestrian (người đi bộ), 3 khung trùng (E4_model_domain: AI chưa phù hợp dữ liệu/quy ước). Người gán: C0 L6 đọc rider/người dắt xe khác reference (R03) | AI lặp cùng kiểu sai trong mẫu nhỏ. Cần kiểm tra trước khi dùng pre-label (nhãn điền sẵn), chưa thể nói lỗi sẽ lan sang mọi ảnh. C0 L6 là bất đồng cách đọc R03 từ bài calib (làm thử) | `model_compare.md/html`, ảnh bằng chứng 02, danh sách M_only (chỉ AI có)/LR_noM (người học và mẫu có, AI không ghép được) trong `findings.csv`, Ticket 1 chuyển nhóm AI (ai_team) |

Giới hạn: B4-mid chỉ có 3 ảnh của **một** camera, cộng thêm 1 ảnh làm thử C0. B4-mid có n_ref=17 khung mẫu và không có vật mẫu ở edge. Precision=0.70 (tỷ lệ khung vẽ khớp mẫu) và recall≈0.82 (tỷ lệ vật mẫu tìm được), với IoU≥0.5 (mức trùng khít giữa hai khung, từ 0 đến 1), lấy từ `local_quality.md`. Đây là độ khớp với bản mẫu dùng để dạy. Ca 261480 L2 còn cần QA (người kiểm tra) xác minh mẫu có thiếu vật hay không. Hai nhóm trên là **giả thuyết cần soi thêm**, không phải tỷ lệ lỗi của cả bộ dữ liệu.

## Chuyển sang kế hoạch bốn camera giả lập

Cách soát độ phủ của 200 frame ở `45_sampling_plan.csv` gồm các bước sau:

1. **Chia tầng trước khi lấy mẫu.** Tầng là nhóm ảnh cùng camera và độ khó. Mỗi `camera_id` × `slice_type` (normal/hard: thường/khó) là một tầng. Ca hard (khó) được gắn tag (dấu phân nhóm) trước khi chọn: đông xe/crowd, rider và xe ba bánh, vật ở rìa có tỷ lệ khoảng cách tới tâm/bán kính vòng kính r/R ≥0.6, ban đêm/mưa/ngược sáng, vùng seam (hai camera nhìn chồng nhau), bãi đỗ.
2. **Chống đếm trùng.** Nhóm frame theo `scene_id` (chuỗi liên tục cùng xe, cùng đoạn đường, cách nhau dưới 5 giây). Mỗi `scene_id` được lấy **tối đa 2 frame** trong một tầng, và phải đến từ ít nhất N/2 cảnh khác nhau, với N là số ảnh của tầng đó.
3. **Bảng kiểm độ phủ.** Sau khi chọn, lập bảng theo camera × tag × zone. Ô nào có 0 ca thì bổ sung, ô nào quá dày thì bớt. Nhật ký chọn mẫu (seed (số khởi tạo lấy mẫu ngẫu nhiên), bộ lọc) được lưu lại để lặp lại được.

Kế hoạch này chỉ giúp **tìm ca cần soi**. Mẫu bị lệch có chủ đích về phía ca khó (hard chiếm 115/200), nên không đo được tỷ lệ lỗi thật của 50.000 frame. Muốn ước lượng tỷ lệ thì cần lấy ngẫu nhiên trong từng tầng và tính trọng số theo xác suất được chọn.
