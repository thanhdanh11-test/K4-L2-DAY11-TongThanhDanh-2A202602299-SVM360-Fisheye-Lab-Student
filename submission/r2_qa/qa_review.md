# QA review · B4-mid

Mã khóa: 4C3B-1E1C

Đây là cold review (tự kiểm lại sau khi khóa, không sửa trực tiếp nhãn). Tôi đối chiếu `qa_overlay.html` (ảnh có khung nhãn) với ảnh gốc của `submission/r1_craft/annotations.xml`. Tôi phóng to vùng đông xe ở mép trái 261480 và 265065. Theo nhật ký, lúc này chưa mở reference (bản nhãn mẫu của giảng viên), model (AI nhận diện tự động) hoặc ảnh hướng dẫn B4-mid. Các mã L chỉ vật trong bản người học; số thứ tự tính từ 1 trong từng ảnh. Box là khung chữ nhật bao vật; R01–R11 là mã luật của bài học.

| frame | object_ref | rule_id | nhận xét |
|---|---|---|---|
| adasind_261480.jpg | L6 | R04 | Khung `Truck` (xe tải), (127,743)-(268,882), bao một van trắng chở người. Ảnh cho thấy kính trước và cửa hông kiểu minivan. Theo R04, van chở người thuộc `Car` (ô tô con). Đề xuất đổi loại nhãn và giữ tọa độ khung. |
| adasind_265065.jpg | L5 | R03 | Khung `Bike` (xe hai bánh), (158,765)-(190,835), bao xe tay ga bạc. Xe có vẻ đang đỗ; người áo sáng đứng sau chưa rõ tư thế. Nếu người đứng riêng thì R03 yêu cầu một `Pedestrian` (người đi bộ) riêng. Bản khóa đã có L4 Pedestrian (147,761)-(168,831) bên cạnh. Cần kiểm tra L4 có đúng người đó trước khi kết luận thiếu người; nếu chưa rõ thì giữ nhãn và ghi lý do. |
| adasind_261480.jpg | L3 | R02 | Khung `Car` (29,764)-(74,843) chồng nhiều lên `Bike` L2 (34,773)-(69,874) và L1 ở mép trái. Phần ô tô nhìn thấy ít. Cần xác nhận đây là vật riêng và khung chỉ bám phần thấy được theo R02. Nếu không tách được, cân nhắc R06: `crowd_or_group` (cụm không tách được từng vật) hoặc `unreadable` (quá mờ hoặc bị che gần hết). |
| adasind_261480.jpg | L9 | R05 | Xe máy gần camera, (644,590)-(1080,1357), có `truncated=true` (bị cắt) vì chạm mép phải ảnh. `occluded=false` (không bị vật khác che) cũng phù hợp với ảnh. Ca này đạt, không đề xuất sửa. |

Các điểm đã soát: 249480 L2 chở hai người nhưng vẫn là một Bike theo R03; mỗi ảnh có hai polygon (vùng khép kín) `lens_border` cho vành đen; cả ba ảnh có `ego_body` cho phần xe gắn camera theo R07. Các vật xa cao dưới 40 pixel (điểm ảnh) không vẽ theo R01.
