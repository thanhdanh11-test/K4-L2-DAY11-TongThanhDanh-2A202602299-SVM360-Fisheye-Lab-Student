# Exit ticket

Đọc `docs/10-svm360-reading-vi.md` trước khi trả lời câu 1–2. Các câu về zone, `why`, rework, parking và sampling
đã nằm trong file tương ứng nên không hỏi lại ở đây.

1. Một vật ở vùng seam giữa hai camera thật xuất hiện với hai box khác nhau: đó là lỗi `DUPLICATE` hay cần một quy
   tắc riêng? Vì sao?

   **Không phải `DUPLICATE`, mà cần một quy tắc riêng.** Box là khung chữ nhật bao vật; seam là vùng hai camera nhìn chồng nhau. `DUPLICATE` là hai box cho cùng một vật **trên cùng một ảnh**. Ở seam, mỗi camera là một ảnh riêng và vật thật sự xuất hiện trên cả hai ảnh, nên với detector (bộ nhận diện) của từng camera, cả hai box đều hợp lệ: có thể `edge` (vùng rìa)/truncated (bị biên ảnh cắt) ở camera này và `mid` (vùng giữa) ở camera kia. Xóa một box thì detector camera đó sẽ bị tính FN (vật bị tính thiếu) oan. Chúng chỉ trở thành "một vật" khi hệ thống có:
   - timestamp (thời điểm chụp) đồng bộ giữa hai camera,
   - calibration (thông số hiệu chỉnh camera), kèm thông tin mặt đường/độ sâu và điểm tương ứng trên vật để đối chiếu vị trí trong BEV (ảnh nhìn từ trên xuống)/hệ tọa độ xe,
   - quy tắc đầu ra nói rõ cần "mỗi camera một box" hay "một vật hợp nhất".

   Khi chưa có đủ ba điều này thì giữ cả hai box, ghi chú là ca seam, và không gán cùng ID.

2. Một vật đi qua nhiều frame trên cùng camera: khi nào giữ cùng track ID, khi nào thêm keyframe hoặc trạng thái
   Outside? Nêu bằng chứng sẽ cần trước khi nối track qua hai camera.

   - **Giữ cùng track ID** (mã theo dõi vật qua nhiều ảnh) khi vẫn là cùng một vật thật còn quan sát được liên tục, kể cả khi đổi zone (mid → edge) hay bị che tạm thời ngắn mà vẫn xác định được là nó.
   - **Thêm keyframe** (ảnh mốc được chỉnh nhãn trực tiếp) khi hình học thay đổi lớn, khung máy tự tính giữa hai ảnh mốc không còn bám phần nhìn thấy: vật đi vào vùng méo mạnh ở rìa, bị cắt bởi vòng kính (truncated), đổi hướng, hoặc bị che/hết che.
   - **Đặt Outside** (trạng thái vật ở ngoài) khi vật ra khỏi trường nhìn, theo quy định của bài. Nếu vật đi vào `lens_border` (vành đen)/`ego_body` (phần xe gắn camera) hoặc bị che hoàn toàn, cần quy định riêng của task; tài liệu hiện tại chưa nói cứ bị che hoàn toàn là Outside. Nếu vật quay lại và xác định chắc là cùng vật thì giữ cùng ID theo quy định; nếu không chắc thì dùng ID mới.
   - **Nối track qua hai camera** chỉ khi có đủ bằng chứng:
     1. Timestamp đồng bộ, và thời điểm vật rời camera A khớp với lúc vào camera B.
     2. Intrinsic/extrinsic (thông số ống kính, vị trí và hướng camera) đã hiệu chỉnh. Cần thêm thông tin mặt đường/độ sâu và điểm tương ứng để kiểm vị trí qua seam.
     3. Class (loại vật) nhất quán; thuộc tính phải phù hợp từng góc nhìn. Ví dụ occluded (bị che) có thể khác giữa hai camera.
     4. Quy tắc đầu ra yêu cầu theo dõi cùng một vật trên toàn hệ thống.

     Thiếu bất kỳ điều nào thì giữ track riêng cho từng camera.

3. Nhìn lại cả buổi: một chỗ bạn tin nhãn mình đúng nhưng reference hoặc người soát nghĩ khác (dẫn frame/`object_ref`),
   bạn đã xử lý thế nào, và nếu làm lại slice này bạn sẽ đổi gì trong cách làm?

   Ở `adasind_261480.jpg` **L2**, mình vẽ Bike cho người áo xanh ngồi trên xe máy phía sau L1 ở mép trái, nhưng reference (bản nhãn mẫu của giảng viên) không có khung riêng, nên compare (báo cáo so sánh) báo SPURIOUS (khung không khớp mẫu). Mình không xóa nhãn để khớp số. Mình phóng to ảnh gốc, thấy rõ người ngồi trên xe máy, khung L2 cao 101 pixel (điểm ảnh), và model (AI nhận diện tự động) M11 cũng có Bike (xe hai bánh) ở cùng chỗ, với IoU≈0.47 (mức trùng khít giữa hai khung, từ 0 đến 1). Vì vậy mình ghi `E0_reference_defect` (nghi bản mẫu thiếu vật), `action=escalate` (chuyển cấp xử lý) cho QA (người kiểm tra) (Ticket 3, decision log D7) và giữ nguyên nhãn trong rework (vòng sửa nhãn).

   Nếu làm lại, mình sẽ ghi rõ chiều cao **phần nhìn thấy** của từng vật bị che ngay lúc vẽ. Lý do là hai lỗi hình học của mình đến từ khung bao cả phần bị che hoặc gộp vật: ô tô 261480 L3 và hai người 265065 L10. Ngoài ra còn lỗi gọi van 261480 L6 là Truck thay vì Car theo R04. Với cảnh đông xe, mình sẽ soát từng cụm và kiểm loại xe trước khi khóa.
