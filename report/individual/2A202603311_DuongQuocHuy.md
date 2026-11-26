# Báo cáo Day 13 — Dương Quốc Huy, 2A202603311

## 1. Kết luận trước
Mã số `2A202603311`, task `D13-2A202603311`, 10 frame: s01f04, s01f52, s02f05, s02f33, s03f10, s03f32, s04f08, s04f36, s05f12, s05f40. Máy Windows 11, trình duyệt Chrome, CVAT server v2.74.1 (bản trên máy từ trước, cũ hơn bản 2.83.0 của lớp). Ngồi cạnh: 2A202603248.

Em vẽ 55 cuboid (48 Car, 3 Truck, 3 Bus, 1 Cyclist). Ít nhất là s03f32 và s05f40, mỗi frame 2 cuboid: hai frame này đường thoáng hơn, nhưng lý do chính là em chưa kịp vẽ hết, vì trên camera vẫn thấy thêm xe chưa có hộp. `selfcheck` còn cờ 21/55 cuboid; phần lớn là `width` của Car vượt 2,3 m, một cuboid đáy chìm 0,41 m (s04f08 idx 7). Em tin nhất ở class và hướng của các xe gần, nơi điểm dày và đối chiếu được với camera; yếu nhất ở kích thước và vị trí của vật xa hoặc bị che một phần, nơi điểm chỉ bám được một mặt (đuôi hoặc hông xe).

Cách dùng CVAT:
- Đổi góc nhìn: xoay, dời và phóng khung 3D bằng chuột; xem ba ô Top (từ trên xuống), Side và Front.
- Vẽ cuboid dạng Shape, đặt vào cụm điểm, chọn hộp rồi chỉnh kích thước và góc xoay ở Top/Side/Front, đối chiếu ảnh cam01, cam03, cam05.
- Bật Cuboid orientation để thấy mũi tên hướng; xem cuboid trên 7 ảnh camera bằng `python scripts/lab.py preview` (CVAT trên máy không chiếu cuboid lên ảnh).

## 2. Đọc frame (Bước 2)
Hai frame đọc kỹ: s01f04 và s01f52. Ảnh: `t1-s01f04-3d`, `t1-s01f04-top`, `t1-s01f52-3d`, `t1-s01f52-top`.

| Frame | Phía trước xe ở cạnh nào của ảnh nhìn từ trên xuống; +x, +y chỉ về đâu | Mặt đường (phẳng/dốc; thấp hơn LiDAR ~ m) | Vật xa nhất đọc được (~ m) | Loại vật thấy | Vật phía sau / hai bên | Cụm chưa đọc được |
|---|---|---|---|---|---|---|
| s01f04 | Phía trước (−y) ở cạnh dưới; +y về cạnh trên (phía sau xe); +x về bên trái xe | Phẳng, không dốc; đáy các Car quanh −1,7 m (−1,64 đến −1,99 m) | Cuboid xa nhất 35 m (idx 8, Car phía sau); còn thấy điểm của xe xa hơn, khoảng 50–60 m | Car, Truck; cột điện, biển báo, rào chắn, tòa nhà | Phía sau: Car idx 8 (35 m), idx 2 (16 m), idx 3 (23 m); bên trái: Truck idx 5 (12,6 m), Car idx 7 (9,9 m); thêm tòa nhà, dải phân cách trên cam05 | Cây, tán cây ven đường bên trái, lẫn tường, hàng rào dọc vỉa hè |
| s01f52 | Phía trước (−y) ở cạnh dưới; +y về cạnh trên; +x về bên trái xe | Phẳng, không dốc; đáy các Car quanh −1,7 m (−1,70 đến −1,79 m) | Cuboid xa nhất 22 m (idx 4, Car phía sau); trên cam01/cam03 còn thấy xe xa hơn ở cuối đường | Car | Phía trước: Car idx 5, 6, 7 (9–17 m); phía sau: idx 1 (7 m), idx 4 (22 m); bên trái: idx 2, 3 (6–9 m); cam05 thấy xe phía sau và tòa nhà xa | Cây xanh ven đường, tường, rào chắn, vỉa hè hai bên |

Chỉ camera cho biết: màu xe và trạng thái đèn (xe trắng, xe tối màu; đèn pha, đèn hậu); biển báo và ký hiệu trên biển; màu cảnh quan như bầu trời, tán cây, vạch kẻ đường. Chỉ LiDAR cho biết: khoảng cách thật tới từng vật theo mét (ví dụ s01f04 idx 8 cách 35 m); kích thước thật dài × rộng × cao, không bị méo phối cảnh; hình học 3D như độ cao mặt đường và cụm điểm ở chỗ tối, khó thấy trên ảnh.

Tám frame lướt:

| Frame | Phía trước ở cạnh nào của ảnh nhìn từ trên xuống | Điều lạ (nếu có) |
|---|---|---|
| s02f05 | Cạnh dưới | Có Bus ở xa và một xe hai bánh có người (Cyclist) |
| s02f33 | Cạnh dưới | Ô tô trắng áp sát rìa trái khung nhìn phía trước (cam01) |
| s03f10 | Cạnh dưới | Xe bán tải hoặc van nhỏ đỏ-trắng phía trước; ô tô bạc bám sát phía sau (cam05) |
| s03f32 | Cạnh dưới | Ít vật nhất (1 Car, 1 Bus), đường thoáng |
| s04f08 | Cạnh dưới | Có Truck; nhiều dải rào chắn vàng hai bên làn, tường công trình lớn bên phải |
| s04f36 | Cạnh dưới | Truck chạy song song sát 2 Car phía trước; một xe máy ở rìa phải cam01 chưa gán nhãn |
| s05f12 | Cạnh dưới | Qua ngã tư có vạch kẻ lớn; cọc tiêu tròn trên dải phân cách; 4 Car dồn sát nhau |
| s05f40 | Cạnh dưới | Bus rất dài sát làn bên trái; một xe trắng xa bên phải trên cam01 chưa gán cuboid |

## 3. Ba cuboid có hướng dẫn (Bước 3)
Ảnh: `t2-a-car`, `t2-b-bus-truck`, `t2-c-ped-cyclist`.

| Vật | Frame, idx | Class | Mũi tên yaw đúng hướng? | Đáy chạm đường? | Đã kiểm Top/Side/Front + ảnh? | Kích thước trong bảng 4.3? |
|---|---|---|---|---|---|---|
| (a) xe con gần | s01f04, 2 | Car | Có: xe cùng chiều, mũi tên chỉ đầu xe, `yaw_deg` 271,0° | Có: z đáy −1,73 m, `ground_gap_m` 0,00 | Có: Top, Side, Front và cam01, cam03 | l 4,60 / w 2,22 / h 1,69 m: dài, cao hợp SUV; rộng hơn 2,10 m |
| (b) xe buýt hoặc xe tải | s01f04, 5 | Truck | Có: xe đi ngược chiều, mũi tên chỉ đầu xe, `yaw_deg` 88,9° | Có: z đáy −1,90 m, `ground_gap_m` 0,10 | Có: Top, Side, Front và cam01, cam03, cam05 | l 6,17 / w 2,73 / h 3,00 m: rộng hơn 2,60 m |
| (c) người đi bộ hoặc xe hai bánh | s02f05, 5 | Cyclist | Có: mũi tên chỉ hướng đi, `yaw_deg` 275,9° (đã sửa sau `selfcheck`) | Có: z đáy −1,55 m, `ground_gap_m` 0,21 | Đã sửa hướng, kích thước và đáy ở Top/Side | l 1,96 / w 0,88 / h 1,62 m: trong khoảng xe máy có người |

Vòng hiệu chuẩn: em không tham gia vòng hiệu chuẩn trên task chung, nên không có số để so với các bạn.

Vật bỏ qua hoặc chưa gán:
- s03f32, ô tô con ở xa chính giữa đường: thấy trên cam01, cam03 nhưng điểm LiDAR rất thưa, không đủ cơ sở xác định kích thước và hướng (LOW_VISIBILITY).
- s04f36, xe máy ở rìa phải cam01 và s05f40, xe trắng xa bên phải cam01: chưa gán cuboid.

## 4. Bốn hiện tượng chất lượng (Bước 4)
Nguyên nhân ghi là **giả thuyết**.

| Hiện tượng | Frame / tên ảnh / nguồn | Nguyên nhân (giả thuyết) | Thấy gì trên point cloud | Hệ quả khi gán (theo guideline) |
|---|---|---|---|---|
| 1. Bóng che | s02f00 / `t4-1-bong-che` / task chung D13-HC485087 | Thân xe buýt chắn tia LiDAR, nên vùng phía sau nó không có tia nào tới | Xe buýt đỏ sát bên trái, khoảng 4 m; phía sau xe buýt gần như không có điểm, hai bên vùng khuất vẫn dày điểm | Vùng trống là chưa quan sát được, không coi là đường trống; vật sau xe buýt không đủ điểm thì không đoán |
| 2. Thưa ở xa | s03f32 / `t4-2-thua-o-xa` / task của em | Càng xa, các tia LiDAR càng tỏa rộng nên ít tia trúng vật | Bus idx 2 ở 28,2 m có 385 điểm, chỉ vài đường quét rời ở mặt sau và hông; Car idx 1 ở 7,9 m có 2287 điểm | Thấy khoảng 15–20% thể tích thì vẽ, dựa camera và bảng 4.3 cho kích thước; ít hơn thì bỏ qua và ghi lại |
| 3. Phản xạ thấp / ít điểm bất thường | s04f02 / `t4-3-phan-xa-thap` / task chung D13-HC485087 | Sơn đen hấp thụ phần lớn tia laser; xe sát bên hông nên tia tới rất xiên, phản xạ lệch ra ngoài | SUV đen ở làn phải, sát bên xe thu dữ liệu: khối điểm thưa, intensity yếu hơn hẳn các xe sáng màu phía trước | Vẫn vẽ theo cụm điểm có được cộng ảnh camera; không nới hộp theo intensity |
| 4. Điểm lạ | s02f35 / `t4-4-diem-la` / task chung D13-HC485087 | Phản xạ đa đường: tia dội qua bề mặt bóng (dải phân cách kim loại, kính, mặt đường ướt) rồi mới về cảm biến, nên quãng đường đo dài hơn thật | Cụm khoảng 46 điểm phía sau bên trái, cách khoảng 29 m, ở z ≈ −13,5 m, thấp hơn mặt đường quanh đó (z ≈ −2,5 m) khoảng 11 m | Không gán thành vật |

## 5. Số từ cuboid của em (Bước 5)

| Cuboid (frame, idx, class) | z | h | z đáy = z − h/2 | dist_m | r = √(x²+y²+z²) | yaw_deg | Nhận xét (lơ lửng / chìm / chạm đường) |
|---|---|---|---|---|---|---|---|
| s01f04, 2, Car | −0,883 m | 1,687 m | −1,727 m | 16,10 m | 16,12 m | 271,0° | chạm đường (lệch −1,7 m chỉ 0,03 m) |
| s03f10, 6, Car | −0,634 m | 1,594 m | −1,431 m | 11,60 m | 11,62 m | 271,4° | cao hơn −1,7 m 0,27 m, sát ngưỡng lơ lửng; `ground_gap_m` 0,34 |
| s04f08, 7, Car | −1,439 m | 1,868 m | −2,374 m | 12,35 m | 12,43 m | 93,6° | chìm: thấp hơn −1,7 m 0,67 m; `selfcheck` báo đáy thấp hơn mặt đường 0,41 m |

(a) Gần nhất: s03f10 idx 3 (Car), `dist_m` 4,45 m, r 4,54 m. Xa nhất: s02f05 idx 1 (Bus), `dist_m` 41,13 m, r 41,14 m.

(b) Xe hai bánh s02f05 idx 5 (Cyclist): l 1,96 / w 0,88 / h 1,62 m, cả ba chiều nằm trong khoảng xe máy có người (1,8–2,3 / 0,70–0,95 / 1,40–1,80 m). Nếu đó là xe đạp thì rộng hơn khoảng 0,50–0,70 m và dài hơn 1,9 m. Xe con s01f04 idx 7: l 4,77 / w 2,42 / h 1,74 m; dài và cao nằm trong khoảng SUV (4,4–5,3 / 1,65–2,10 m), rộng vượt 2,10 m. Giả thuyết: hộp bao rộng hơn cụm điểm, hoặc tính cả gương.

(c) Xe cùng chiều phía trước: s01f52 idx 6 (y −10,23 m) có `yaw_deg` 269,8°, gần 270°. Xe ngược chiều: s01f04 idx 7 có `yaw_deg` 90,3°, gần 90°.

Mật độ theo khoảng cách (4 cuboid ở các `dist_m` khác nhau):

| Cuboid (frame, idx, class) | dist_m | Số điểm (`points_in`) |
|---|---|---|
| s03f10, 3, Car | 4,45 m | 2420 |
| s01f04, 4, Car | 11,61 m | 838 |
| s01f04, 3, Car | 22,97 m | 279 |
| s01f04, 8, Car | 35,17 m | 108 |

Xa gấp khoảng 2 lần thì số điểm còn khoảng 1/3 (11,6 → 23,0 m: 838 → 279; 23,0 → 35,2 m: 279 → 108). Cùng xu hướng với đường cong ở slide (718 ở 10–20 m, 214 ở 20–30 m, 72 ở 30–40 m); số của em cao hơn một chút ở mỗi khoảng.

Các cuboid đã sửa sau `selfcheck`:
- s02f05 idx 5 (Cyclist): `selfcheck` báo `length` < `width` (yaw không dọc thân xe). Em xoay mũi tên dọc thân xe về hướng đi, đổi kích thước từ 0,86 × 1,45 m thành 1,96 × 0,88 m và hạ đáy; khoảng hở với mặt đường giảm từ 0,34 m xuống 0,21 m, hết cờ.
- s01f04 idx 1 và idx 8 (Car): giảm `width` từ 2,38 m và 2,42 m xuống 2,30 m.

Cảnh báo giữ nguyên: các cờ `width` > 2,3 m của Car còn lại và s04f08 idx 7 (chìm 0,41 m) em chưa kịp xem lại, nên chưa khẳng định được hộp đúng hay sai.

Đổi kiểm với 2A202603248: đối chiếu Bus ở s03f32 (idx 2 trong bảng A của em). Rộng: em 3,90 m, bạn 4,0 m; cao: em 4,10 m, bạn 4,1 m. Hai số khớp, chỉ lệch khoảng 0,1 m ở chiều rộng, nên em không sửa. Cả hai đều rộng hơn khoảng 2,50–2,60 m của bảng 4.3 (`selfcheck` cũng cờ chiều rộng hộp này), đây là điểm cần xem lại. Ngoài ra hai bên có lệch ở các frame s03f32, s04f08, s04f36.

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc
(a) Model học trên nhãn này phải ước lượng kích thước và khoảng cách thật để xe tự hành tính quỹ đạo và khoảng cách phanh; đo bằng mét thì một xe dài 4,5 m vẫn là 4,5 m dù cách 5 m hay 50 m, còn pixel thì to nhỏ theo phối cảnh. Vật thưa điểm chỉ lộ một mặt nên mỗi người dễ đoán kích thước và hướng một kiểu; cần quy tắc riêng (ngưỡng thể tích thấy được, bảng kích thước tham chiếu, đối chiếu camera, bỏ qua khi không đủ cơ sở) để nhãn nhất quán và model không học từ nhãn đoán bừa.

(b) Công cụ: Claude Code (trợ lý AI trong VS Code). Câu hỏi chính: cài môi trường, chạy lệnh `scripts/lab.py`, git; theo cho phép của mentor, nhờ AI đọc bảng A và phiếu tự kiểm để điền số, và viết lại thành câu các quan sát em ghi trong ghi chú. Cách dùng kết quả: số liệu do AI chép từ bảng A, em đối chiếu lại với `submission/`; nội dung quan sát trên point cloud và ảnh là của em. AI không xem dữ liệu VF, ảnh camera hay ảnh chụp màn hình.

(c) Không có điều chưa chắc nào khác ngoài các vật đã ghi bỏ qua hoặc chưa gán ở mục 3.
