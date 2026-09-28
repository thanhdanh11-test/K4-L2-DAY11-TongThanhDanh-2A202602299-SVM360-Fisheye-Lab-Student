# Rework delta

| zone | matched before | matched after | missing before | missing after | spurious before | spurious after |
|---|---:|---:|---:|---:|---:|---:|
| center | 4 | 4 | 0 | 0 | 2 | 2 |
| mid | 10 | 13 | 3 | 0 | 4 | 1 |
| edge | 0 | 0 | 0 | 0 | 0 | 0 |

## Findings action=rework
- adasind_261480.jpg L3 SPURIOUS: chưa sửa
- adasind_261480.jpg L6 SPURIOUS: đã sửa
- adasind_261480.jpg R3+M3 MISSING: đã sửa
- adasind_265065.jpg L10 SPURIOUS: đã sửa
- adasind_265065.jpg R7 MISSING: đã sửa
- adasind_265065.jpg R8+M5 MISSING: đã sửa

## Nhận xét của học viên

Các mã L dưới đây dùng thứ tự của bản khóa r1_craft, trừ khi ghi rõ là chỉ số mới. Box là khung chữ nhật bao vật; reference là bản nhãn mẫu của giảng viên. Rework nghĩa là sửa lại nhãn.

- **Ba quyết định rework** có mức P1 (lỗi vật cần sửa), được ghi ở r3_diag và decision log (nhật ký quyết định) D2–D4. Bản sửa của task (bài làm) `Day11 · ADASIND · B4-mid · raw_fisheye` đã xuất lại và khóa với mã **8DB5-6D8E**:
  1. 261480 L6 đổi `Truck` (xe tải) → `Car` (ô tô con), giữ khung (127,743)-(268,882). R04 xếp van chở người vào Car. Hai dòng L6 và R3+M3 ở trên báo đã sửa.
  2. 265065 L10, (730,763)-(770,847), được tách thành hai `Pedestrian` (người đi bộ): (731,763)-(753,841), có `occluded=true` (bị vật khác che), và (744,765)-(770,847). Đây là sửa theo R02. Ba dòng L10, R7, R8+M5 báo đã sửa. Cùng với sửa van, số thiếu ở mid (vùng giữa ảnh) giảm từ 3 xuống 0.
  3. Xóa 261480 L3 `Car` (29,764)-(74,843), vì phần ô tô nhìn thấy được ước tính dưới 40 pixel (điểm ảnh), theo R01/R02. Bảng vẫn báo “L3 chưa sửa” vì công cụ tìm theo số thứ tự. Sau khi xóa, L3 mới là ThreeWheeler (xe ba bánh), (72,768)-(101,827), đã khớp mẫu. Tệp mới có 8 box ở 261480 và không còn khung Car nói trên. Số thừa ở mid giảm từ 4 xuống 1 sau ba sửa đổi.
- **Spurious còn lại ở mid (1)** nghĩa là một khung không ghép được với mẫu: 261480 L2 Bike (xe hai bánh). Tôi giữ nhãn vì nghi mẫu thiếu vật, mã `E0_reference_defect`. Ticket 3 chuyển QA (người kiểm tra) xác minh; không xóa nhãn chỉ để giống mẫu.
- **Spurious ở center (2)** là 265065 L7 và L9 ở vùng tâm ảnh. Giữ `E5_unresolved` (chưa phân xử được) và `keep_with_reason` (giữ nhãn, có lý do). Chờ người soát thứ hai đo phần nhìn thấy.
- **Không sửa** ba box 265065 L1–L3 thuộc bất đồng `IGNORE_SCOPE` (vùng được tính). Bản mẫu bỏ qua cụm bằng `crowd_or_group` (cụm không tách được từng vật), còn tôi vẽ riêng. Ca này chờ duyệt R06a ở Ticket 2; quyết định hiện tại vẫn là khoảng trống luật.
- Bảng trước/sau chỉ đo độ khớp với bản mẫu trên ba ảnh. Nó không chứng minh nhãn đúng tuyệt đối.
