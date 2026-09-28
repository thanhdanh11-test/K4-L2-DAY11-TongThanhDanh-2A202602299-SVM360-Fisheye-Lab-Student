# Escalation ticket

L là nhãn người học, R là reference (bản nhãn mẫu của giảng viên), M là model (AI nhận diện tự động). Box là khung chữ nhật bao vật; số sau L/R/M là thứ tự vật trong từng ảnh theo báo cáo. Ticket là phiếu chuyển vấn đề cho nhóm có trách nhiệm.

## Ticket 1

- **Frame:** `adasind_261480.jpg`: L4 (72,768)-(101,827), L5 (97,760)-(142,844), L7 (252,744)-(338,862). L và R đều gọi ba vật là ThreeWheeler (xe ba bánh). YOLO26m gọi M8/M6/M10 là Car (ô tô con), đồng thời có M12 Truck (xe tải) trùng với M10. Các dòng r3_diag `L4+R5`, `L7+R4`, `L5+R6` mang `LR_noM` (L/R có, M không ghép đúng loại) và `E4_model_domain` (AI chưa phù hợp dữ liệu hoặc quy ước bài).
- **Ảnh chụp:** `submission/screenshots/02_261480_threewheeler_model_gap.png`.
- **Expected impact:** AI nhận đúng loại **0/3** xe ba bánh được tính trong nhóm ảnh này. Nếu dùng các dự đoán này làm pre-label (nhãn điền sẵn), người vẽ phải đổi Car/Truck bằng tay. M10+M12 còn làm tăng số khung thừa. Ba ca cùng một ảnh cho thấy vấn đề đáng kiểm tra, chưa đủ kết luận mọi xe tuk-tuk hoặc toàn bộ dữ liệu đều bị sai như vậy.
- **Owner:** `ai_team` (nhóm phụ trách AI).
- **Recommendation:**
  1. Kiểm danh sách loại đầu ra của YOLO26m: có ThreeWheeler/auto-rickshaw (xe ba bánh) hay đang đổi tên sang Car/Truck. Không suy ra danh sách loại chỉ từ ba dự đoán sai.
  2. Đo trên 48 ảnh ADASIND, đếm xe ba bánh bị gọi thành Car/Truck để kiểm tra giả thuyết E4 trên nhiều ảnh.
  3. Trong lúc chờ, không dùng loại dự đoán của AI để điền sẵn cho xe ba bánh. Nhắc lại R04: không gọi ThreeWheeler là Bus/Truck (xe buýt/xe tải).

## Ticket 2

- **Frame:** `adasind_265065.jpg`, cụm (35..146, 751..817). L1/L2 ThreeWheeler và L3 Bike (xe hai bánh) nằm chủ yếu trong polygon (vùng khép kín) `crowd_or_group` của R. Vùng R là (0,760)-(145,815), không bao kín toàn bộ ba khung. Finding (ghi nhận) r3_diag L1 giữ `IGNORE_SCOPE` (bất đồng vùng được tính), `E2_guideline_gap` (luật chưa đủ rõ), P0 (ưu tiên xử lý phạm vi trước).
- **Ảnh chụp:** `submission/screenshots/01_265065_crowd_ignore_vs_boxes.png`.
- **Expected impact:** ba khung bị loại khỏi phép đo theo quy tắc don't-care (không tính đúng hay sai). Vì thế precision (tỷ lệ khung vẽ khớp mẫu) và recall (tỷ lệ vật mẫu tìm được) phụ thuộc cách áp dụng R06. Cần thống nhất cách xử lý cụm đông xe trước khi so người vẽ với nhau.
- **Owner:** `guideline` (nhóm phụ trách luật).
- **Recommendation:** xem xét R06a trong `20_guideline_patch.md`, phiên bản v1.1.0. Sau khi duyệt, QA (người kiểm tra) soát lại `crowd_or_group` của bản mẫu 265065 theo luật mới.

## Ticket 3

- **Frame:** `adasind_261480.jpg`, L2 Bike (34,773)-(69,874), cao 101 pixel (điểm ảnh), phía sau L1 ở mép trái. Tôi đọc ảnh là người áo xanh ngồi trên xe máy. R không có khung riêng; M11 Bike có tọa độ chính xác (41.57,807.62)-(69.46,867.85), thường ghi gọn là (41,807)-(69,867). IoU≈0.47 (mức trùng khít giữa hai khung, từ 0 đến 1). Dòng r3_diag L2 giữ `E0_reference_defect` (nghi bản mẫu thiếu vật).
- **Ảnh chụp:** `submission/screenshots/02_261480_threewheeler_model_gap.png`, vùng L2/M11 ở mép trái.
- **Expected impact:** nếu QA xác nhận xe máy đủ điều kiện R01, mẫu đang thiếu một Bike. Khi đó một nhãn hợp lệ bị tính FP (khung thừa so với mẫu). Dùng mẫu chưa xác minh làm gold (nhãn chuẩn đã kiểm chứng) có thể khiến người học xóa nhầm vật thật.
- **Owner:** `qa` (người kiểm tra nhãn).
- **Recommendation:** xác minh vật và phần nhìn thấy trên ảnh gốc. Nếu đúng, bổ sung Bike vào bản mẫu và ghi phiên bản mới. Giữ L2 trong rework (vòng sửa nhãn); không sửa chỉ để khớp mẫu.
