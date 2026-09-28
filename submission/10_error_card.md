# Error analysis card

## Zone × block

| zone | block | what | count |
|---|---|---|---:|
| center | B4 | MISSING | 1 |
| center | B4 | SPURIOUS | 8 |
| center | C0 | BOX_GEOMETRY | 1 |
| center | C0 | SPURIOUS | 2 |
| edge | B4 | SPURIOUS | 1 |
| mid | B4 | ATTRIBUTE | 1 |
| mid | B4 | BOX_GEOMETRY | 2 |
| mid | B4 | IGNORE_SCOPE | 4 |
| mid | B4 | MISSING | 7 |
| mid | B4 | SPURIOUS | 13 |
| mid | B4 | WRONG_CLASS | 2 |

## Top defects
- SPURIOUS: 24 (ví dụ frame adasind_019560.jpg)
- MISSING: 8 (ví dụ frame adasind_265065.jpg)
- IGNORE_SCOPE: 4 (ví dụ frame adasind_265065.jpg)

## Phân tích của bạn

Các bảng trên đếm các dòng trong `findings.csv` qua nhiều vòng làm bài. Một vật có thể được ghi ở nhiều vòng, nên số dòng lỗi không phải số vật khác nhau. L là nhãn người học, R là reference (bản nhãn mẫu của giảng viên), M là model (AI nhận diện tự động). Box là khung chữ nhật bao vật.

- Nguyên nhân khả dĩ (`why`) và vì sao bạn nghĩ vậy:
  - `SPURIOUS` (khung không ghép được với mẫu) có 24 dòng. Trong đó **12/24** là khung của AI thuộc `M_only` (chỉ M có), không phải 16. Có 7 ở mid (vùng giữa), 4 ở center (vùng tâm) và 1 ở edge (vùng rìa). Theo ảnh, 3 khung ở 249480, 8 ở 261480 và 1 ở 265065.
  - Tôi giữ phân loại `E4_model_domain` (AI chưa phù hợp dữ liệu hoặc quy ước bài) cho các ca đã ghi. Dấu hiệu gồm: tách người ngồi trên xe thành Pedestrian ở 249480 M2/M4 và 261480 M2/M5; gọi xe ba bánh là Car/Truck ở 261480 M6/M8/M10/M12; khung trùng ở 261480 M13 và 265065 M8. Ca M10/M12 cũng là một cặp trùng. Lặp lại trong mẫu nhỏ là lý do cần kiểm tra thêm, chưa đủ chứng minh lỗi hệ thống trên toàn bộ dữ liệu.
  - 12 dòng SPURIOUS còn lại thuộc L: 2 ở vòng calib (bài làm thử), 4 ở r1_craft và 6 ở r3_diag. Trong r3_diag, `E1_annotator_error` (lỗi người vẽ) gồm 261480 L3 bao phần bị che, L6 gọi van là Truck, và 265065 L10 gộp hai người. 261480 L2 là `E0_reference_defect` (nghi bản mẫu thiếu xe máy). 265065 L7/L9 là `E5_unresolved` (chưa đủ bằng chứng phân xử).
  - `MISSING` (thiếu khung ghép được) có 8 dòng qua các vòng: 1 ở r2_qa, 1 ở r1_craft và 6 ở r3_diag. Sáu dòng sau gồm 3 xe ba bánh M không ghép đúng loại, 2 người R7/R8 do L10 gộp, và van R3+M3 do L6 sai loại. Ca r2_qa L5 là nghi vấn cũ; L đã có L4 Pedestrian riêng.
  - `IGNORE_SCOPE` (bất đồng vùng được tính) có 4 dòng, nhưng cùng nói về cụm 265065 L1–L3: 3 dòng r1_craft và 1 dòng tổng hợp r3_diag. Giữ `E2_guideline_gap` (luật chưa đủ rõ), P0 (ưu tiên xử lý phạm vi trước), theo R06/R10.
- Cách sửa và ai nhận việc (`owner`):
  - `annotator` (người vẽ): đã rework (sửa lại nhãn) ba ca P1 (lỗi vật cần sửa): L6 đổi sang Car, xóa L3, tách L10. Bảng `rework/delta.md` cho thấy mid ghép đúng tăng 10→13, thiếu giảm 3→0, thừa giảm 4→1.
  - `ai_team` (nhóm AI): Ticket 1 đề nghị kiểm loại ThreeWheeler và quy ước rider (người ngồi trên xe hai bánh) của YOLO26m trên 48 ảnh. Trong lúc chờ, không dùng nhãn AI của các ca này làm nhãn điền sẵn.
  - `guideline` (nhóm phụ trách luật): Ticket 2 đề nghị duyệt R06a, phiên bản v1.1.0, về `crowd_or_group` (cụm không tách được từng vật).
  - `qa` (người kiểm tra): Ticket 3 đề nghị xác minh và bổ sung Bike 261480 L2 vào bản mẫu; người soát thứ hai đo lại 265065 L7/L9.
- Bằng chứng (ảnh trong `screenshots/`, dòng findings, rule):
  - `screenshots/02_261480_threewheeler_model_gap.png`: xe ba bánh L/R đối chiếu M8/M10/M12; xe L5 còn tương ứng M6. L2 đối chiếu M11; liên quan R04 và R01.
  - `screenshots/03_265065_L10_merged_pedestrians.png`: L10 gộp so với R7+R8, liên quan R02.
  - `screenshots/01_265065_crowd_ignore_vs_boxes.png`: cụm xe bị bỏ qua, liên quan R06.
  - Các dòng r3_diag trong `findings.csv` giữ nguyên why/severity/owner/action (nguyên nhân, ưu tiên, người xử lý, hành động). `r3_diag/local_quality_conflicts.csv` ghi L6/R3 ở 261480 khác loại với IoU=0.917 (mức trùng khít giữa hai khung, từ 0 đến 1); R7/R8 ở 265065 không ghép được.
  - Giới hạn: kết quả B4-mid chỉ dựa trên 3 ảnh của một camera và bản mẫu dùng để dạy.
