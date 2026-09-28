# Guideline patch

- **Rule mới đề xuất:** **R06a — Khi nào dùng `crowd_or_group`.** Đây là đề xuất bổ sung luật, chưa được duyệt. Polygon (vùng khép kín) `ignore_region` là vùng bỏ qua khi so sánh. Chỉ đặt `reason=crowd_or_group` (lý do: cụm không tách được từng vật) khi thỏa **cả hai** điều kiện:
  1. Có ít nhất 3 vật cùng hoặc khác class (loại nhãn) chồng nhau.
  2. Không xác định được mép trái/phải để vẽ box (khung chữ nhật bao vật) riêng; hoặc phần nhìn thấy của mỗi vật cao dưới 40 pixel (điểm ảnh), theo R01.

  Nếu mỗi vật có mép riêng và cao ít nhất 40 pixel thì vẽ từng vật, không bỏ qua cả cụm.
  - Ví dụ đúng: `adasind_265065.jpg`, cụm (35..146, 751..817) có hai xe tuk-tuk vàng và một xe máy. Ba khung cao lần lượt 56, 58 và 51 pixel. Tôi thấy có thể tách mép từng xe, nên đề xuất giữ L1/L2 ThreeWheeler (xe ba bánh) và L3 Bike (xe hai bánh).
  - Ví dụ nên bỏ qua: hàng xe máy đỗ sát nhau ở xa, chỉ thấy một dải yên và tay lái, không đếm được từng xe. Đây là ví dụ giả định, không phải phát hiện mới trong ba ảnh.

  Bằng chứng: `screenshots/01_265065_crowd_ignore_vs_boxes.png`.
- **Áp dụng cho:** `ignore_region` có `reason=crowd_or_group` theo R06. Có liên quan R01 (ngưỡng H=40) và R09 (không để từ 50% diện tích box trở lên trong vùng bỏ qua). Áp dụng cho sáu loại vật của bài học ở mọi zone (vùng tâm/giữa/rìa ảnh).
- **Vì sao luật hiện tại (`docs/02-rules-vi.md`) không đủ:** R06 nói “cụm vật không tách được từng cái” nhưng chưa có tiêu chí đo. Trong 265065, reference (bản nhãn mẫu của giảng viên) bỏ qua cụm, còn tôi vẽ ba xe. `compare.md` ghi ba dòng `IGNORE_SCOPE` (bất đồng phạm vi được tính); r3_diag gộp thành một finding (ghi nhận lỗi) L1, mức P0 (ưu tiên xử lý phạm vi trước) theo R10. Nếu hiểu luật khác nhau, cùng một vật có lúc được tính đúng/thiếu, có lúc không được tính. Vì thế recall (tỷ lệ vật mẫu tìm được) thay đổi theo cách chọn phạm vi. Đề xuất chỉ áp dụng sáu loại vật của lab, không mở rộng sang vật tĩnh trong bộ nhãn ADASIND/slide.
- **`rules_version` mới:** đề xuất v1.0.0 → **v1.1.0**.
- **Hiệu lực từ:** đề xuất áp dụng ở round `rework` (vòng sửa nhãn) của đợt sau và các slice (nhóm ảnh) mới, sau khi được duyệt. Nhãn đã khóa theo v1.0.0 được giữ lại; khi so sánh phải ghi phiên bản luật. QA (người kiểm tra) cần soát lại bản mẫu 265065 trước khi dùng làm gold (nhãn chuẩn đã kiểm chứng).
