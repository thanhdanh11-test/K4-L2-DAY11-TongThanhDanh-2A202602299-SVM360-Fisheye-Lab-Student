# Đề xuất gold set theo camera — tình huống giả lập

**Đầu bài:** 50.000 frame từ bốn camera SVM, ngân sách chọn 200 frame để review/gold. Đây là tình huống trên slide,
**không phải** 50.000 frame có trong repo. Phân bổ đúng 200 ở `45_sampling_plan.csv` cho bốn camera, mỗi camera có
normal và hard slice. “Gold set” ở đây là **kế hoạch tạo** reference sau kiểm chứng, không phải teaching reference
ADASIND hoặc nhãn bạn vừa vẽ. Nếu cần, dùng `notebooks/day11-svm360-colab.ipynb` để thử tổng phân bổ; notebook
không làm thay phần lý do.

Các từ dùng trong kế hoạch: frame là một ảnh; SVM là hệ thống quan sát quanh xe; review là kiểm tra; gold là nhãn chuẩn sau kiểm chứng. Reference là bản nhãn mẫu; box là khung chữ nhật bao vật; polygon là vùng khép kín nối bằng nhiều đoạn thẳng; class là loại vật. H=40 là chiều cao tối thiểu 40 pixel (điểm ảnh). Truck/Car/ThreeWheeler lần lượt là xe tải/ô tô con/xe ba bánh. Camera front/rear/left/right lần lượt ở trước/sau/trái/phải. Normal/hard là cảnh thường/khó.

Phân bổ (tổng 200): front 25 normal + 35 hard; rear 20 + 25; left 20 + 30; right 20 + 25.

| camera_id | Hard case cần chọn | Vì sao dễ sai | Annotation space / calibration cần giữ | Cách review trước khi gọi là gold |
|---|---|---|---|---|
| front | Cảnh đô thị đông xe có rider, xe ba bánh, van chở người; cụm xe ở xa sát ngưỡng H=40; ngược sáng | Đây là các kiểu lỗi đã thấy ở ADASIND có hướng nhìn giống phía trước: van gán Truck (R04), rider (người ngồi trên xe hai bánh)/người dắt xe (R03), cụm crowd (cụm đông xe) hay vẽ từng xe (R06), model (AI nhận diện tự động) không ghép đúng loại xe ba bánh ThreeWheeler | Nhãn trên **ảnh fisheye gốc** (ống kính mắt cá, ảnh cong ở rìa; chưa nắn ảnh); lưu intrinsic (thông số ống kính), vòng kính (cx, cy, r) và `camera_id`, `rules_version` cho mỗi frame | 2 annotator (người vẽ) làm độc lập; QA (người kiểm tra) thứ ba phân xử mà không biết ai vẽ bản nào. Mọi bất đồng class/rider đi qua nhóm phụ trách luật, ghi vào decision log (nhật ký quyết định) trước khi khóa |
| rear | Lùi/đỗ xe: người và xe rất gần bị cắt bởi vòng kính, lẫn với thân xe ego; vạch đỗ phía sau | Vật gần camera bị méo mạnh và bị `truncated` (biên ảnh hoặc vòng kính cắt); ranh giới `ego_body` (phần xe gắn camera)/`lens_border` (vành đen) với vật rất dễ sai (R07–R09) | Polygon `ego_body` và `lens_border` theo **đúng rig của camera sau** (cách lắp camera), không dùng chung với front; lưu extrinsic (vị trí và hướng camera) để biết vùng thân xe cố định | Soát riêng vùng bỏ qua (P0, ưu tiên xử lý phạm vi) trước khi soát box: một người review chỉ kiểm `ego_body`/`lens_border`/ignore, sau đó mới tới class/box |
| left | Xe máy vượt sát hông, vật ở seam trước-trái và sau-trái, vạch đỗ cong ở rìa | Vật chạy song song bị kéo dài dọc cạnh ảnh; cùng một vật xuất hiện ở hai camera (seam, vùng hai camera nhìn chồng nhau) dễ bị coi là trùng | Giữ **timestamp đồng bộ** (thời điểm chụp được căn cùng nhau) và extrinsic giữa left–front và left–rear để đối chiếu vùng seam; đánh dấu `edge_zone` (vùng rìa ảnh) | Review cặp frame cùng timestamp của hai camera kề nhau. Box ở seam chỉ gọi là gold khi quy tắc giữa các camera (bên dưới) đã được áp dụng |
| right | Người đi bộ sát lề bị xe đỗ che; curb (bó vỉa) và `parking_line` (vạch chia ô đỗ); seam trước-phải | Dễ nhầm curb/lề với vạch chia ô (docs/11) và dễ bỏ sót người bị che (occluded: bị vật khác che) | Như left, cộng thêm phía lề đường của rig; lưu calibration (thông số hiệu chỉnh camera) để chiếu `parking_line` sang BEV (ảnh nhìn từ trên xuống) khi cần kiểm hình học | 2 người vẽ độc lập và một người phân xử không biết ai vẽ bản nào; ca curb/parking_line so với ảnh đối chiếu vai trò vạch; tỷ lệ đồng thuận tính **riêng cho camera right** |

Quy tắc chung trước khi gọi là gold:
- Chọn mẫu theo tầng (nhóm camera và độ khó) và tag (dấu phân nhóm), tối đa 2 ảnh mỗi scene (cùng cảnh quay) trong một tầng, như trong `45_review_plan.md`.
- Người rà là người **độc lập** với người vẽ.
- Bất đồng được giải quyết bằng một trọng tài (adjudicator), dựa trên luật có phiên bản. Kết quả ghi `decision`, `rationale` (quyết định và lý do) vào decision log (nhật ký quyết định).
- Chỉ khóa khi mọi finding (ghi nhận lỗi) P0/P1 đã đóng hoặc có quyết định escalate (chuyển cấp xử lý), theo kế hoạch đề xuất. Phải ghi rõ ca còn chờ; chuyển cấp chưa có nghĩa là nhãn đã được xác nhận đúng.

- **Khi nào cần refresh gold set (đổi camera, calibration hoặc rule):**
  - Khi thay kiểu camera hoặc vị trí lắp camera hoặc thay đổi calibration (intrinsic/extrinsic, vòng kính dịch >2% bán kính; đây là ngưỡng đề xuất, không phải luật hiện hành). Khi đó kiểm tra lại **toàn bộ** frame của camera bị đổi.
  - Khi `rules_version` tăng phiên bản phụ/chính (ví dụ R06a v1.1.0 về `crowd_or_group`). Khi đó soát lại những frame chứa class/tag bị luật mới ảnh hưởng.
  - Khi phân bố dữ liệu vận hành lệch khỏi gold, ví dụ tỷ lệ ban đêm/mưa hoặc thành phố mới.
  - Đề xuất rà mỗi quý: soát một mẫu nhỏ để phát hiện trôi (drift).
- **Một ca seam/cross-camera cần policy và evidence trước khi ghép hai box:** Ví dụ giả lập: một xe máy vượt ở góc trước-trái xuất hiện đồng thời ở camera front (vùng edge, tức rìa ảnh, bị cắt một phần) và camera left (vùng mid, tức giữa ảnh). Trước khi ghép hai box thành một vật, hoặc coi một box là `DUPLICATE` (khung trùng), cần có:
  1. Timestamp đồng bộ của hai frame (chênh lệch < khoảng cách 1 frame).
  2. Calibration intrinsic/extrinsic, cùng thông tin mặt đường hoặc độ sâu và điểm tương ứng trên vật, để đối chiếu vị trí trong BEV/hệ tọa độ xe. Chỉ có hai khung chữ nhật và thông số camera chưa đủ để suy ra vị trí 3D của cả vật.
  3. Quy tắc đầu ra: hệ cần "mỗi camera một box" (cho bộ nhận diện từng camera) hay "một vật duy nhất trên BEV" (cho bước hợp nhất dữ liệu).

  Khi chưa có đủ ba điều này thì **giữ cả hai box** (mỗi camera hợp lệ), không xóa, không gán cùng track ID (mã theo dõi vật).
- **Vì sao peer agreement hoặc quality report trên ảnh một camera chưa chứng minh gold set đúng cho cả bốn camera:**
  - Báo cáo ở lab đo trên 3 frame của một camera ADASIND, n_ref = 17, zone edge (vùng rìa) không có vật mẫu nào, và so với một bản mẫu dùng để dạy còn ca nghi thiếu vật (E0_reference_defect ở 261480 L2).
  - Hai người đồng thuận vẫn có thể cùng sai theo một cách hiểu luật (ví dụ R06 crowd).
  - Mỗi camera có méo, vùng ego, loại vật và seam khác nhau, nên lỗi ở front không dự đoán được lỗi ở rear/left/right.
  - Vì vậy gold phải được kiểm và đo đồng thuận **riêng từng camera**, có calibration và phiên bản luật đi kèm.
