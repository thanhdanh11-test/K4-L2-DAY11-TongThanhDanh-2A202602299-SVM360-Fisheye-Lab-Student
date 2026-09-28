# Rework delta

| zone | matched before | matched after | missing before | missing after | spurious before | spurious after |
|---|---:|---:|---:|---:|---:|---:|
| center | 4 | 4 | 0 | 0 | 2 | 1 |
| mid | 10 | 11 | 3 | 2 | 4 | 1 |
| edge | 0 | 0 | 0 | 0 | 0 | 0 |

## Findings action=rework
- adasind_261480.jpg L3 SPURIOUS: chưa sửa
- adasind_261480.jpg L6 SPURIOUS: đã sửa
- adasind_261480.jpg R3+M3 MISSING: đã sửa
- adasind_265065.jpg L10 SPURIOUS: đã sửa
- adasind_265065.jpg R7 MISSING: chưa sửa
- adasind_265065.jpg R8+M5 MISSING: đã sửa

## Nhận xét của học viên

Các mã L dưới đây dùng thứ tự của bản khóa r1_craft, trừ khi ghi rõ là chỉ số mới. Box là khung chữ nhật bao vật. Reference là bản nhãn mẫu của giảng viên. Rework nghĩa là sửa lại nhãn.

Bảng trên so bản khóa r1_craft (4C3B-1E1C) với bản rework **cuối cùng**. Bản rework cuối là bản tôi tự sửa tay trong CVAT (phần mềm vẽ nhãn), sau đó khóa lại bằng `--relock` với mã **48A2-5E6E**. Nó thay cho bản rework trước (8DB5-6D8E). Lý do khóa lại được ghi ở decision log D9.

- **Các sửa đổi giữ từ bản rework trước** (P1, decision log D2–D4):
  1. 261480 L6 đổi `Truck` (xe tải) → `Car` (ô tô con), giữ khung (127,743)-(268,882), theo R04. Hai dòng L6 và R3+M3 báo đã sửa.
  2. Xóa 261480 L3 `Car` (29,764)-(74,843), vì phần nhìn thấy dưới 40 pixel (R01/R02). Bảng vẫn báo "L3 chưa sửa" vì công cụ tìm theo số thứ tự. Sau khi xóa, L3 mới là ThreeWheeler (xe ba bánh) (72,768)-(101,827), đã khớp mẫu.
  3. 265065 L10 (730,763)-(770,847), khung gộp hai người: còn lại một `Pedestrian` (người đi bộ) (744,765)-(770,847), ghép được với R8. Khung người thứ hai (731,763)-(753,841) có ở bản rework trước, nhưng đã bị bỏ trong lần sửa tay. Vì vậy dòng R7 báo "chưa sửa" và mid vẫn thiếu R7.
- **Thay đổi thêm trong lần sửa tay:**
  - 265065: bỏ khung `Pedestrian` người áo đỏ sẫm (147,761)-(168,831), nên R6 của mẫu trở thành thiếu.
  - Thu nhỏ theo chiều dọc 5 khung: hai ThreeWheeler (35,770)-(77,808), (77,772)-(113,809); Bike (114,775)-(131,811); Bike (158,787)-(186,828); Bike (420,788)-(454,823). Bốn khung trong số này cao dưới 40 pixel nên theo R01 không còn được tính khi so sánh. Vì vậy cụm L1–L3 không còn báo `IGNORE_SCOPE`, và L9 (Bike cạnh cầu thang) không còn báo spurious.
  - Thu nhỏ Pedestrian L7 xuống (276,759)-(302,811).
  - 249480: thêm hai `Car` ở xa, (0,760)-(70,792) cao 31 px và (73,755)-(98,781) cao 26 px. Cả hai dưới ngưỡng 40 pixel nên không được tính.
  - Bỏ 3 polygon K12 (hình viền dùng để đo fill ratio) và một mảnh `ego_body` nhỏ (381..469, 1773..1800) ở 261480. K12 chỉ dùng cho vòng r1_craft (xem `r1_craft/selfqc.md`).
- **Kết quả ở mid (vùng giữa ảnh):** ghép đúng tăng 10 → 11, thiếu giảm 3 → 2, thừa giảm 4 → 1.
  - Hai vật còn thiếu là người R6 (145,766)-(163,826) và R7 (731,778)-(753,841) ở 265065.
  - Khung thừa còn lại ở mid là 261480 L2 Bike. Tôi giữ nhãn này vì nghi mẫu thiếu vật (`E0_reference_defect`, Ticket 3).
- **Center (vùng tâm):** thừa giảm 2 → 1. Khung còn lại là Pedestrian (276,759)-(302,811) ở 265065 (`E5_unresolved`, chưa phân xử được). L9 không còn được tính vì khung mới chỉ cao 35 pixel.
- **Lưu ý khi đọc số:** giảm "thừa" ở đây một phần đến từ việc khung bị thu dưới 40 pixel, tức bị loại khỏi phép đo, chứ không phải vì nhãn khớp mẫu hơn. Bảng trước/sau chỉ đo độ khớp với bản mẫu trên ba ảnh, không chứng minh nhãn đúng tuyệt đối.
