# Quan sát vạch ô đỗ

Ảnh `parking-lot-core.jpg` rộng 960, cao 720 pixel (điểm ảnh). Task (bài làm) trong CVAT (phần mềm vẽ nhãn lên ảnh) có tên `Day11 · parking_line · public-sample`. Tọa độ dưới đây tính trên ảnh gốc.

- **Hai vạch `parking_line` đã vẽ:**
  - Vạch 1 đi qua (407,655) → (464,686) → (525,719). Đây là vạch trắng chéo gần camera, ở giữa phía dưới ảnh.
  - Vạch 2 đi qua (697,624) → (826,653) → (959,684). Đây là vạch trắng chéo gần camera, ở bên phải.

  Tôi đọc hai vạch này là đường chia các ô đỗ kề nhau. Mỗi đường chỉ bám phần sơn nhìn thấy và dừng ở mép ảnh. Ảnh `parking-lot-contrast.png` là một bãi khác, dùng để so vai trò của vạch chia ô và lối xe chạy, không dùng để đo tọa độ của ảnh chính.
- **Một biên không vẽ và lý do:** biên nhạt gần ngang ở xa, khoảng (20,464)–(185,465), nằm ở mép bãi, bên trái xe đỏ. Phần biên xa còn tiếp tục về phía hàng cây. Tôi đọc đây là curb (mép bó vỉa) bao quanh bãi, không phải vạch chia từng ô. Vệt nhạt quanh y≈533 phía trái cũng không được vẽ vì chưa thấy rõ nó tạo ranh giới ô đỗ.
- **Polygon `free_space` dừng ở đâu:** `free_space` là phần mặt bãi còn trống và nhìn thấy được. Polygon (vùng khép kín nối bằng nhiều đoạn thẳng) nằm trong khoảng (0,475)–(960,670). Đây là bản tôi tự sửa tay trong CVAT (phần mềm vẽ nhãn); bản đầu chỉ phủ lối xe chạy ở giữa, khoảng (60,540)–(945,662).
  - Mép dưới nằm phía trên đầu hai vạch gần camera, (407,655) và (697,624), nên không phủ dãy ô đỗ gần camera. Ở mép trái, mép dưới xuống tới (0,670).
  - Mép trên đi gần như ngang ở y≈475–480, ngay dưới biên xa của bãi. Vì vậy vùng này phủ cả lối xe chạy lẫn các ô đỗ trống ở tầm giữa ảnh.
  - Hai mép bên kéo ra tới biên trái và phải của ảnh.

  Mép trên đi sát mép dưới xe đỏ, khoảng (194,458)–(220,479): tại x≈194–220, mép polygon ở y≈479, chạm đáy xe nhưng không cắt vào thân xe. Vùng này không phủ bó vỉa, cây hay cột đèn ở xa. Đây chỉ là nhận xét trên ảnh tĩnh, không chứng minh xe có thể chạy qua an toàn.
- **Ca chưa chắc cần hỏi người soát:** vạch mờ quanh y≈500–560 khó xác định điểm đầu, điểm cuối và vai trò. Tôi để trống để người soát xác nhận đó là vạch chia ô hay vạch chỉ đường. Cũng cần thống nhất lối chạy bắt đầu ở đâu để mọi người chừa khoảng cách trước đầu vạch ô giống nhau.
