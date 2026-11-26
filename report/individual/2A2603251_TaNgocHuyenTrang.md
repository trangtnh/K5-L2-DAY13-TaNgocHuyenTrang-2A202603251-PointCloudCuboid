# Báo cáo Day 13 — Tạ Ngọc Huyền Trang, 2A2603251

## 1. Kết luận trước
- **Mã số học viên**: 2A2603251
- **Task CVAT**: D13-2A2603251
- **10 frame của tôi**: s01f40, s01f59, s02f22, s02f54, s03f21, s03f40, s04f23, s04f47, s05f16, s05f45
- **Môi trường thực hành**: macOS (Apple Silicon), trình duyệt Google Chrome, phiên bản CVAT v2.74.1 (tương thích cvat-sdk v2.83.0).
- **MSSV bạn ngồi cạnh / đổi kiểm**: 2A202603067

Tổng cộng bài làm có **81 cuboid** trên 10 frame (trung bình 8.1 cuboid/frame). Frame đầu tiên `s01f40` có 10 cuboid (đáp ứng quy định tối thiểu ≥ 8), tất cả 9 frame còn lại đều có từ 5–21 cuboid (đáp ứng quy định tối thiểu ≥ 5 mỗi frame). Phân bố theo frame: s01f40=10, s01f59=6, s02f22=6, s02f54=5, s03f21=8, s03f40=5, s04f23=21, s04f47=6, s05f16=7, s05f45=7. Phiếu tự kiểm `selfcheck` ghi nhận 9/81 cuboid có cảnh báo (11.1%), chủ yếu là cảnh báo kích thước ở xe bị che khuất một phần hoặc lơ lửng do mặt đường cục bộ cao hơn trung bình. 

**Cách dùng CVAT (3 dòng thao tác cốt lõi):**
1. Đổi góc nhìn: Kéo chuột trái để xoay phối cảnh 3D, kéo chuột phải để di chuyển khung hình (Pan), lăn con lăn chuột để phóng to/thu nhỏ, xoay nhìn thẳng từ đỉnh xuống (Top-down view) để kiểm tra tổng thể và phân bổ vật thể quanh xe.
2. Vẽ và căn chỉnh cuboid: Chọn công cụ vẽ cuboid trên thanh công cụ trái, chọn class nhãn, bấm chọn chế độ **Shape** (tuyệt đối không dùng Track), di chuột đến cụm điểm và nháy đúp để đặt hộp; dùng khung Top để tinh chỉnh tâm (x, y), kích thước dài/rộng và góc xoay yaw; dùng khung Side và Front kéo cạnh trên/dưới để đáy chạm khít mặt đường và đặt chiều cao hộp.
3. Bật mũi tên hướng và đối chiếu camera: Bật tính năng **Appearance → Cuboid orientation** để kiểm tra vạch đỏ ngắn (+X của cuboid) luôn chỉ về đầu xe hoặc hướng di chuyển; chạy lệnh `python scripts/lab.py preview` để chiếu 3D cuboid lên 7 ảnh camera nhằm kiểm tra độ khớp trực quan.

## 2. Đọc frame (Bước 2)
Hai frame đọc kỹ. Ảnh: `t1-s01f40-3d.png`, `t1-s01f40-top.png`, `t1-s01f59-3d.png`, `t1-s01f59-top.png`.

| Frame | Phía trước xe ở cạnh nào của ảnh nhìn từ trên xuống; +x, +y chỉ về đâu | Mặt đường (phẳng/dốc; thấp hơn LiDAR ~ m) | Vật xa nhất đọc được (~ m) | Loại vật thấy | Vật phía sau / hai bên | Cụm chưa đọc được |
|---|---|---|---|---|---|---|
| s01f40 | Phía trước xe ở cạnh trên của ảnh nhìn từ trên xuống (-y); +x chỉ sang trái, +y chỉ xuống dưới (phía sau xe) | Mặt đường tương đối phẳng; thấp hơn cảm biến LiDAR khoảng 1.7 m (z ≈ -1.7 m) | ~31.6 m (xe con #8 ở xa phía trước) | Car, Cyclist | Phía sau xe có ô tô con ở y ≈ 13.3 m (cam09); bên trái có xe đạp ở x ≈ -10.8 m | Cụm điểm thưa ở x ≈ 11.5 m, y ≈ -32.1 m (chưa rõ là rào chắn hay đuôi xe) |
| s01f59 | Phía trước xe ở cạnh trên của ảnh nhìn từ trên xuống (-y); +x chỉ sang trái, +y chỉ xuống dưới (phía sau xe) | Mặt đường phẳng; thấp hơn cảm biến LiDAR khoảng 1.7 m (z ≈ -1.7 m) | ~16.2 m (xe con #6 phía trước) | Car | Phía sau có xe con ở y ≈ 6.6 m; bên phải có xe ở x ≈ 5.9 m, y ≈ -10.9 m | Cụm điểm phía sau bên trái x ≈ -10.5 m, y ≈ 15.2 m bị khuất một phần do bóng che |

**3 điều CHỈ CAMERA cho biết:**
1. Màu sơn thực tế của phương tiện (xe trắng, xe đen, taxi màu vàng) và màu sắc trang phục người đi bộ.
2. Nội dung chữ viết và ký hiệu trên biển báo giao thông, biển số xe, vạch kẻ làn đường.
3. Trạng thái tín hiệu đèn giao thông (đỏ, vàng, xanh) và trạng thái đèn xi-nhan, đèn phanh của xe phía trước.

**3 điều CHỈ LIDAR cho biết:**
1. Khoảng cách vật lý chính xác tuyệt đối theo đơn vị mét tính từ tâm xe đến từng bề mặt phản xạ (không phụ thuộc méo quang học của thấu kính camera).
2. Kích thước hình học 3 chiều thực tế (chiều dài, chiều rộng, chiều cao đo bằng mét) của vật thể.
3. Khả năng phát hiện vật thể đáng tin cậy trong bóng tối hoàn toàn hoặc trong tình huống camera bị lóa mắt do ngược sáng chói chang.

Tám frame lướt:

| Frame | Phía trước ở cạnh nào của ảnh nhìn từ trên xuống | Điều lạ (nếu có) |
|---|---|---|
| s02f22 | Phía trên của khung nhìn (-y) | Đoạn đường hơi cong nhẹ, có xe tải lớn di chuyển làm che khuất góc nhìn bên trái |
| s02f54 | Phía trên của khung nhìn (-y) | Nhiều xe ô tô con dừng đỗ sát mép vỉa hè ở cả hai bên làn đường |
| s03f21 | Phía trên của khung nhìn (-y) | Nhóm người đi bộ tập trung qua đường ở phía sau bên trái xe |
| s03f40 | Phía trên của khung nhìn (-y) | Dải tường quây công trình bên đường tạo phản xạ liên tục |
| s04f23 | Phía trên của khung nhìn (-y) | Mật độ xe con đông đúc, khoảng cách giữa các xe tương đối sát nhau |
| s04f47 | Phía trên của khung nhìn (-y) | Biển báo phản quang bên lề phải cho cường độ phản xạ (intensity) rất cao |
| s05f16 | Phía trên của khung nhìn (-y) | Xe buýt lớn di chuyển ở làn ngược chiều phía trước bên phải |
| s05f45 | Phía trên của khung nhìn (-y) | Giao lộ ngã tư có xe tải và nhiều người điều khiển xe máy đang chuyển làn |

## 3. Ba cuboid có hướng dẫn (Bước 3)
Ảnh minh chứng: `t2-a-car.png`, `t2-b-bus-truck.png`, `t2-c-ped-cyclist.png`.

| Vật | Frame, idx | Class | Mũi tên yaw đúng hướng? | Đáy chạm đường? | Đã kiểm Top/Side/Front + ảnh? | Kích thước trong bảng 4.3? |
|---|---|---|---|---|---|---|
| (a) xe con gần | s01f40, #1 | Car | Đúng (vạch đỏ chỉ về đầu xe, yaw = 270.5°) | Đúng (z đáy = −1.836 m, đáy chạm mặt đường) | Đã kiểm tra sát điểm trên cả 3 view và ảnh cam01 | Dài 4.478 m, Rộng 1.873 m, Cao 1.550 m (chuẩn sedan Bảng 4.3: 3.8–5.3 m) |
| (b) xe buýt hoặc xe tải | s05f45, #4 | Truck | Đúng (vạch đỏ chỉ về đầu xe, yaw = 90.0°) | Đúng (z đáy = −2.063 m, đáy sát mặt đường) | Đã kiểm tra sát điểm trên cả 3 view và ảnh camera | Dài 10.855 m, Rộng 2.782 m, Cao 3.158 m (chuẩn xe tải Bảng 4.3: 5.0–13.0 m) |
| (c) người đi bộ hoặc xe hai bánh | s03f21, #1 | Pedestrian | Đúng (vạch đỏ chỉ theo hướng mặt, yaw = 254.2°) | Đúng (z đáy = −1.546 m, đáy sát mặt đường) | Đã kiểm tra sát điểm trên cả 3 view và ảnh cam06 | Dài 0.683 m, Rộng 0.650 m, Cao 1.920 m (chuẩn người lớn Bảng 4.3: 0.4–0.9 m, cao 1.5–1.95 m) |

**Vòng hiệu chuẩn:** Trong buổi học hiệu chuẩn chung, ban đầu một số học viên dễ bị nhầm lẫn giữa Class Pedestrian và Cyclist (đặc biệt khi người dắt xe máy vs người đang ngồi điều khiển xe). Cả lớp đã thống nhất theo Guideline: người đang ngồi lái xe 2 bánh thì dán nhãn Cyclist (bao gồm cả xe và người trong 1 cuboid); còn người dắt bộ xe máy/xe đạp thì dán nhãn Pedestrian. Ngoài ra, lớp đã chốt vạch đỏ (+X) luôn phải hướng về đầu xe (yaw quanh 270° nếu cùng chiều, 90° nếu ngược chiều).
Các vật đã bỏ qua vì không đủ cơ sở: Tại frame `s01f40`, ở vị trí x ≈ 11.5 m, y ≈ -32.1 m có một cụm điểm mờ chỉ gồm khoảng 10 điểm, ảnh camera bị lóa sáng không nhận diện được xe hay vật cố định, tôi quyết định bỏ qua theo quy tắc LOW_VISIBILITY thay vì đoán mò.

## 4. Bốn hiện tượng chất lượng (Bước 4)
Nguyên nhân được ghi nhận dưới dạng **giả thuyết khoa học**. Ảnh: `t4-1-bong-che.png`, `t4-2-thua-o-xa.png`, `t4-3-phan-xa-thap.png`, `t4-4-diem-la.png`.

| Hiện tượng | Frame / tên ảnh / nguồn | Nguyên nhân (giả thuyết) | Thấy gì trên point cloud | Hệ quả khi gán (theo guideline) |
|---|---|---|---|---|
| 1. Bóng che | s01f40, `t4-1-bong-che.png` | Tia LiDAR phát thẳng bị bề mặt kim loại của xe buýt/xe tải kích thước lớn phía trước chặn lại hoàn toàn | Xuất hiện một khoảng khuyết điểm hình quạt rỗng kéo dài ngay phía sau phương tiện | Bật cờ thuộc tính `occluded` cho vật thể bị che khuất một phần; cuboid vẫn bao trọn phần thể tích suy luận hợp lý |
| 2. Thưa ở xa | s01f40, `t4-2-thua-o-xa.png` | Do chùm tia phát LiDAR có góc mở phân kỳ góc cố định, cự ly càng xa thì mật độ tia quét trên diện tích bề mặt càng giảm mạnh | Xe ở khoảng cách 48.4 m chỉ còn vài chục điểm rải rác, trong khi xe ở 3.2 m có hơn 1400 điểm | Nếu thấy được khoảng 15–20% thể tích và có căn cứ từ camera thì vẽ; nếu quá ít điểm không thể xác định biên dạng thì bỏ qua và ghi `LOW_VISIBILITY` |
| 3. Phản xạ thấp / ít điểm bất thường | s01f40, `t4-3-phan-xa-thap.png` | Bề mặt xe tối màu (sơn đen) hoặc kính xe có đặc tính hấp thụ và tán xạ phần lớn bước sóng tia laser LiDAR, tín hiệu phản xạ về cảm biến rất yếu | Xe ở cự ly gần (13 m) nhưng số điểm phản xạ nhận được ít hơn hẳn các vật thể bên cạnh; giá trị cường độ phản xạ (intensity) rất thấp | Ghi nhận dưới dạng giả thuyết; đối chiếu ảnh camera để xác định ranh giới thật của xe, tránh thu hẹp hộp quá mức theo cụm điểm |
| 4. Điểm lạ | s02f35, `t4-4-diem-la.png` (Task chung vòng hiệu chuẩn) | Hiện tượng phản xạ gương đa đường (multipath reflection) khi tia LiDAR chiếu vào bề mặt vũng nước ướt hoặc nắp hố ga kim loại rồi phản xạ dội lại | Xuất hiện cụm 102 điểm dị vật nằm sâu dưới mặt đường ở độ sâu z ≈ -13.5 m (thấp hơn mặt đường thực tế khoảng 11 m) | Tuyệt đối không dán nhãn cụm điểm này thành bất kỳ phương tiện hay người; ghi nhận hiện tượng bất thường do quang học cảm biến |

## 5. Số từ cuboid của tôi (Bước 5)
Số liệu trích xuất từ `submission/D13-2A2603251-bangA.csv` và `submission/D13-2A2603251-tukiem.csv`. Mặt đường danh định tại z ≈ -1.7 m (ngưỡng cho phép sai số $\pm 0.3$ m):

| Cuboid (frame, idx, class) | z | h | z đáy = z − h/2 | dist_m | r = √(x²+y²+z²) | yaw_deg | Nhận xét (lơ lửng / chìm / chạm đường) |
|---|---|---|---|---|---|---|---|
| s01f40, #1, Car | −1.061 m | 1.550 m | −1.836 m | 3.20 m | 3.33 m | 270.5° | Chạm mặt đường tốt (lệch 0.14 m so với −1.7 m) |
| s01f40, #2, Cyclist | −0.471 m | 1.484 m | −1.213 m | 11.0 m | 11.03 m | 265.9° | Lệch 0.49 m — xe đạp có độ nâng gầm bình thường |
| s01f40, #3, Car | −1.031 m | 1.571 m | −1.817 m | 14.6 m | 14.64 m | 273.2° | Chạm mặt đường tốt (lệch 0.12 m so với −1.7 m) |

**(a) Cuboid gần nhất và xa nhất:**
- **Cuboid gần nhất**: Frame `s05f16`, cuboid #1 (Car) có $x = -4.510\text{ m}, y = 1.812\text{ m}, z = -0.797\text{ m}$; khoảng cách mặt phẳng ngang $\text{dist\_m} = \sqrt{x^2 + y^2} = 4.86\text{ m}$, khoảng cách không gian 3D $r = \sqrt{x^2 + y^2 + z^2} = 4.93\text{ m}$.
- **Cuboid xa nhất**: Frame `s05f45`, cuboid #6 (Truck) có $x = -32.218\text{ m}, y = 16.028\text{ m}, z = 0.490\text{ m}$; khoảng cách ngang $\text{dist\_m} = 36.0\text{ m}$, khoảng cách 3D $r = 35.98\text{ m}$.

**(b) Kích thước so với Bảng 4.3:**
- **Người đi bộ (s03f21 #1 Pedestrian)**: $L = 0.683\text{ m}, W = 0.650\text{ m}, H = 1.920\text{ m}$. Kích thước này hoàn toàn nằm trong khoảng tham chiếu của người lớn (dài 0.4–0.9 m, rộng 0.4–0.9 m, cao 1.50–1.95 m); chiều cao 1.92 m gần biên trên nhưng vẫn hợp lệ.
- **Xe con (s01f40 #1 Car)**: $L = 4.478\text{ m}, W = 1.873\text{ m}, H = 1.550\text{ m}$. Nằm chuẩn xác trong khoảng sedan tham chiếu (dài 3.8–5.3 m, rộng 1.7–2.1 m, cao 1.4–2.1 m). Chiều dài $L > W$ đảm bảo hình chữ nhật dọc thân xe.

**(c) Góc xoay Yaw:**
- Xe cùng chiều phía trước (di chuyển theo hướng xe thu dữ liệu): $\text{yaw} \approx 270.0^\circ$ (mũi tên đỏ quay về phía trước $-y$).
- Xe ngược chiều (di chuyển hướng lại gần xe thu dữ liệu): $\text{yaw} \approx 90.0^\circ$ (mũi tên đỏ quay về $+y$).

**Mật độ theo khoảng cách (4 cuboid ở các cự ly dist_m khác nhau):**

| Cuboid (frame, idx, class) | dist_m | Số điểm (`points_in`) |
|---|---|---|
| s04f47, #2, Car | 7.8 m | 3022 điểm |
| s01f40, #3, Car | 14.6 m | 829 điểm |
| s04f23, #8, Car | 27.1 m | 133 điểm |
| s05f45, #6, Truck | 36.0 m | 492 điểm |

**Quy luật mật độ điểm:** Khi khoảng cách tăng từ 7.8 m lên 36.0 m (tăng ~4.6 lần), số điểm giảm từ 3022 xuống 492 điểm (giảm ~6.1 lần), gần đúng với tỷ lệ $1/d^2$. Mật độ điểm LiDAR suy giảm xấp xỉ tỷ lệ nghịch với bình phương khoảng cách do góc quét của các tia phát dãn rộng ra. Kết quả này tương đồng với đường cong lý thuyết trên slide bài giảng (xe con ở 0–10 m có trung vị ~1968 điểm, 10–20 m còn ~718 điểm, 20–30 m còn ~214 điểm).
**Tự kiểm sau sync:** Tôi đã kiểm tra và điều chỉnh 2 cuboid trên CVAT sau khi xem ảnh preview và selfcheck: (1) tại frame `s04f47` cuboid #2 (Car, dist=7.8 m) — căn chỉnh lại đáy sát mặt đường vì hộp bị lệch nhẹ lên trên; (2) tại frame `s03f40` cuboid #3 (Cyclist, dist=13.9 m) — điều chỉnh góc yaw để mũi tên đỏ chỉ đúng chiều di chuyển thay vì hướng ngược lại. Các cảnh báo kích thước còn lại (xe bị che khuất một phần ở s04f23) được giữ nguyên vì việc thu nhỏ hộp sẽ bỏ sót thể tích thực của xe.

**Đổi kiểm chéo:** Đã đối chiếu số liệu mục 5 với bạn học MSSV 2A202603067. Hai con số được chọn đối chiếu là khoảng cách $\text{dist\_m} = 3.20\text{ m}$ của cuboid s01f40#1 (Car) và $z_{\text{đáy}} = -1.836\text{ m}$; cả hai con số đối chiếu khớp 100% giữa Bảng A và màn hình CVAT.

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc
**(a) Từ phía mô hình học máy (Model 3D Detection):**
1. Nhãn 3D bắt buộc phải đo bằng mét trong không gian thực (thay vì pixel như ảnh 2D) vì xe tự hành cần biết chính xác khoảng cách vật lý thực tế để thuật toán điều khiển lên kế hoạch phanh khẩn cấp và căn khoảng cách an toàn tránh va chạm.
2. Vật thể thưa điểm ở xa cần quy tắc gán nhãn nghiêm ngặt riêng (quy tắc 15–20% thể tích) bởi vì nếu gán nhãn tùy tiện khi chỉ có 1–2 điểm nhiễu, mô hình sẽ học sai đặc trưng (overfitting vào nhiễu) và dẫn tới hiện tượng "phanh ma" (phantom braking) cực kỳ nguy hiểm khi xe vận hành thực tế.

**(b) Cách sử dụng công cụ hỗ trợ:**
1. Công cụ: ChatGPT và trợ lý Antigravity trong VS Code.
2. Câu hỏi chính: Hỏi về cách sửa lỗi môi trường ảo Python (.venv) trên macOS, lệnh chạy CVAT Docker, và giải thích quy ước hệ tọa độ LiDAR xe VinFast (x sang trái, y về sau, z hướng lên; phía trước xe là -y).
3. Sử dụng kết quả: Dùng hướng dẫn để cấu hình môi trường chạy script `lab.py sync` và `check_submission.py`, hiểu đúng hướng đầu xe để tự xoay vạch đỏ (+X) của cuboid trong CVAT đạt góc yaw chuẩn.

**(c) Điều còn chưa chắc:**
Ở frame `s04f23`, tại làn xe ngược chiều có một xe tải chở hàng cồng kềnh với phần bạt phủ phía sau phồng to, việc xác định ranh giới chuẩn giữa kích thước thân xe danh định và phần hàng hóa nhô ra còn phụ thuộc vào góc nhìn của camera, cần thêm ý kiến trao đổi từ Lab Coach.
