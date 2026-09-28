# Tự soát

Nhóm ảnh **B4-mid** gồm adasind_249480, 261480 và 265065. Bài làm trong CVAT (phần mềm vẽ nhãn lên ảnh) có tên `Day11 · ADASIND · B4-mid · raw_fisheye`.

Theo nhật ký của người học, bản nháp `exports/r1-draft.xml` được xuất lại ở cấp task (toàn bài) vì lần xuất ở cấp job (phần việc) thiếu tên `raw_fisheye`. Sau đó bước tự kiểm `selfqc` không còn cảnh báo. Bản cuối `r1-final.zip` được khóa thành `r1_craft`, mã **4C3B-1E1C**. Người học ghi nhận chưa mở reference (bản nhãn mẫu của giảng viên), model (AI nhận diện tự động) hay ảnh hướng dẫn B4-mid trước khi khóa. Tệp khóa xác nhận bản nhãn; riêng thứ tự mở tài liệu là lời ghi lại của người học.

## Checklist thủ công
- [x] Phạm vi H=40 và vật cần vẽ: Đã soát 23 box (khung chữ nhật bao vật); mỗi khung cao ít nhất 40 pixel (điểm ảnh). Ba ô tô xa bên trái 249480 trong bản mẫu cao 18, 20 và 23 pixel, đều dưới ngưỡng R01. Các vật quá nhỏ ở 261480/265065 cũng không vẽ. Biển hiệu, cột và nhà không thuộc sáu loại nhãn. Chiều cao khung không tự chứng minh phần vật nhìn thấy đủ 40 pixel; ca L3 ở 261480 được sửa ở bước sau.
- [x] lens_border và ego_body: Mỗi ảnh có hai polygon (vùng khép kín) `lens_border` đánh dấu vành đen. Đây là nhãn có sẵn đã được soát và chỉnh mép theo R08. Cả ba ảnh đều thấy phần xe/chân/dép ở mép dưới trái nên có `ego_body` theo R07: 249480 có 2 vùng, 261480 có 3, 265065 có 1. Hai ngoại lệ 006840/271039 không thuộc nhóm ảnh này.
- [x] Class sáu nhãn: Sáu class (loại vật) là Car (ô tô con), Bus (xe buýt), Truck (xe tải), ThreeWheeler (xe ba bánh), Bike (xe hai bánh), Pedestrian (người đi bộ). Bản khóa có Truck ở 249480 L1, 261480 L6 và 265065 L6; L6 ở 261480 về sau được xác định là van chở người, phải đổi sang Car theo R04. MPV bạc thuộc Car. Xe tuk-tuk vàng/xanh thuộc ThreeWheeler.
- [x] Rider và Bike: Theo R03, rider (người ngồi trên xe hai bánh) và xe được vẽ chung thành một Bike. Xe chở hai người ở 249480 cũng chỉ có một khung. Người đứng cạnh xe ở 265065 được vẽ riêng thành Pedestrian; người ngồi trong ô tô không có khung riêng.
- [x] Geometry trên ảnh fisheye gốc: Vẽ trên ảnh fisheye (ống kính mắt cá, ảnh cong ở rìa) gốc theo R02, không nắn thẳng ảnh. Có bốn cặp box và polygon mang cùng `group_id` (mã ghép cặp). Fill ratio là diện tích polygon chia diện tích box; ở đây khoảng 0.61–0.78, chỉ để minh họa, không phải ngưỡng đạt. Pedestrian 265065 có tỷ lệ thấp nhất 0.606 do phần trống quanh dáng người và giữa hai chân.
- [x] truncated và occluded: `truncated` nghĩa là bị biên ảnh hoặc vòng kính cắt. Hai Bike ở 261480 được đặt true (có): (0,754)-(62,926) và (644,590)-(1080,1357). `occluded` nghĩa là bị vật khác che; ví dụ Car bạc bị xe máy che và xe tuk-tuk cạnh xe van. Hai thuộc tính được xét riêng theo R05. Vùng làm mờ để bảo vệ riêng tư không được tính là vật che.
- [x] Vật thiếu hoặc box trùng: Đã phóng to vùng đông xe trên cả ba ảnh. Bốn polygon K12 là hình đối chiếu với box, không phải bốn vật thêm. Lúc tự soát còn nghi cụm Car/Bike ở mép trái 261480, x<75: ba khung chồng nhau, khó tách vật. Đã ghi điểm này để QA (người kiểm tra nhãn) xem lại.
- [x] ignore_region có reason: Mỗi `ignore_region` (vùng bỏ qua) có một reason (lý do): `lens_border` hoặc `ego_body`. Không box nào nằm từ 50% diện tích trở lên trong các vùng bỏ qua của bản người học theo R09. Lúc vẽ, tôi thấy từng vật tách được nên chưa dùng `crowd_or_group` (cụm không tách được từng vật). Bản mẫu về sau cho thấy bất đồng ở 265065 L1–L3.
- [x] Tên task raw_fisheye và export CVAT 1.1: Tên task (bài làm) có `raw_fisheye`. Tệp xuất theo định dạng "CVAT for images 1.1", không kèm ảnh. Ba ảnh có đúng tên và kích thước 1080×1920.

## Fill ratio (K12)
- adasind_249480.jpg box 1 mid: 0.776
- adasind_261480.jpg box 1 mid: 0.740
- adasind_261480.jpg box 8 mid: 0.752
- adasind_265065.jpg box 12 mid: 0.606
