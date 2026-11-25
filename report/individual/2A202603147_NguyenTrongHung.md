# Báo cáo Day 13 — Nguyễn Trọng Hùng, 2A202603147

## 1. Kết luận trước
- **Mã số (MSSV):** `2A202603147`
- **Task CVAT:** `D13-2A202603147`
- **10 frame được giao:** `s01f00`, `s01f54`, `s02f12`, `s02f52`, `s03f01`, `s03f46`, `s04f26`, `s04f54`, `s05f17`, `s05f43`
- **Môi trường:** Windows, Chrome, CVAT 3D v2.83.0
- **Tổng quan kết quả:** Tổng cộng **53 cuboid** trên 10 frame (frame đầu `s01f00` có 8 cuboid; hai frame `s03f46` và `s05f43` ít nhất với 4 cuboid mỗi frame vì ngoài 4 phương tiện rõ trong vùng gần thì các cụm điểm còn lại ở xa quá thưa dưới 15–20% thể tích nên bỏ qua theo quy tắc `LOW_VISIBILITY` để không đoán mò; các frame còn lại có 5–6 cuboid). Kết quả `selfcheck` có **3/53 cuboid** bị gắn cờ cảnh báo. Phần tự tin nhất là việc xác định class, hướng mũi tên `yaw` và căn đáy `z_bottom` chạm mặt đường cho các phương tiện trong phạm vi 25–30 m; phần còn yếu nhất là ước lượng kích thước bao trọn đối với người đi bộ hoặc vật thể ở xa trên 20 m khi cụm điểm LiDAR bị thưa hoặc khuất một phần.

**Cách dùng CVAT:**
- Đổi góc nhìn: xoay khung 3D (chuột trái) để nhìn từ trên xuống (BEV), kéo chuột phải để dời, con lăn chuột để thu phóng.
- Vẽ và chỉnh một cuboid: chọn công cụ Cuboid → chọn class → bấm **Shape**, đưa con trỏ tới cụm điểm rồi nháy đúp để đặt hộp; chỉnh vị trí/dài/rộng/yaw ở khung **Top**, chỉnh độ cao và đáy chạm mặt đường ở khung **Side** và **Front**.
- Bật mũi tên hướng: tích chọn **Appearance → Cuboid orientation** để căn trục `+X` dọc thân xe hướng về đầu xe, và dùng `python scripts/lab.py preview` để đối chiếu cuboid trên 7 ảnh camera.

## 2. Đọc frame (Bước 2)
Hai frame đọc kỹ: `s01f00` và `s01f54`. Ảnh: `t1-s01f00-3d.png`, `t1-s01f00-top.png`, `t1-s01f54-3d.png`, `t1-s01f54-top.png` (ảnh nhìn từ trên xuống, có đánh dấu phía trước, +x, +y).

| Frame | Phía trước xe ở cạnh nào của ảnh nhìn từ trên xuống; +x, +y chỉ về đâu | Mặt đường (phẳng/dốc; thấp hơn LiDAR ~ m) | Vật xa nhất đọc được (~ m) | Loại vật thấy | Vật phía sau / hai bên | Cụm chưa đọc được |
|---|---|---|---|---|---|---|
| `s01f00` | Phía trước xe là hướng `-y` (trùng hướng nhìn của `cam01`); trục `+x` chỉ sang bên trái xe, trục `+y` chỉ ra phía sau xe | Mặt đường phẳng, thấp hơn cảm biến LiDAR khoảng `1.7 m` (`z_bottom` quanh `-1.65 m` đến `-1.83 m`) | Khoảng `30.34 m` (xe buýt `idx=8`) đến `~40 m` | `Car`, `Bus`, `Cyclist`, `Pedestrian` | Phía sau có 3 xe con (`idx=2` ở `9.74 m`, `idx=4` ở `12.55 m`, `idx=6` ở `16.97 m`); bên trái gần có `Car idx=1` (`3.34 m`); bên phải có `Cyclist idx=3` (`10.81 m`), `Pedestrian idx=5` (`22.19 m`), `Car idx=7` (`23.12 m`) và `Bus idx=8` (`30.34 m`) | Cụm điểm thưa ở vùng rìa xa trên `40 m` chỉ có vài điểm rời rạc, không rõ biên dạng |
| `s01f54` | Phía trước xe là hướng `-y` (trùng hướng nhìn của `cam01`); trục `+x` chỉ sang bên trái xe, trục `+y` chỉ ra phía sau xe | Mặt đường phẳng, thấp hơn cảm biến LiDAR khoảng `1.7 m` (`z_bottom` quanh `-1.69 m` đến `-1.79 m`) | Khoảng `35–40 m` (trong phạm vi gán nhãn rõ nhất là `12.96 m`) | `Car` (nhiều xe con ở cả phía trước và phía sau) | Phía trước (`-y`) có 3 xe con (`idx=3` ở `9.44 m`, `idx=5` ở `10.52 m`, `idx=4` ở `12.96 m` lệch trái); phía sau (`+y`) có 2 xe con (`idx=1` ở `6.69 m` thẳng sau `az_deg = 178.8°`, `idx=2` ở `11.15 m` sau lệch trái) | Cụm điểm ở xa ngoài `35 m` bị khuất sau hàng xe phía trước, dưới 15% thể tích |

- **3 điều chỉ camera cho biết:** (1) màu sắc thân xe và trang phục người đi đường để phân biệt nhanh các xe cùng kích thước, (2) chi tiết đầu xe/đuôi xe (đèn pha trước, đèn hậu sau) khi cụm điểm LiDAR ở xa bị thưa giúp chốt đúng hướng `yaw`, và (3) ngữ cảnh bề mặt/vật thể nhỏ (như phân biệt người đi bộ `Pedestrian` với người đi xe hai bánh `Cyclist` hoặc biển báo/cây cối bên đường).
- **3 điều chỉ LiDAR cho biết:** (1) tọa độ 3D thực tế `(x, y, z)` và kích thước vật lý `(length, width, height)` đo trực tiếp bằng đơn vị mét (`m`), (2) khoảng cách ngang chính xác `dist_m = √(x² + y²)` trong toàn bộ góc 360° quanh xe (kể cả vùng góc mù giữa 7 camera), và (3) cao độ mặt đường thực tế `z_bottom` quanh từng vật thể.

Tám frame lướt:

| Frame | Phía trước ở cạnh nào của ảnh nhìn từ trên xuống | Điều lạ (nếu có) |
|---|---|---|
| `s02f12` | Hướng `-y` (trùng hướng `cam01`) | Có cả xe cùng chiều (`yaw_deg ≈ 273.6°–279.8°`) và xe ngược chiều (`yaw_deg ≈ 87.7°–88.0°`); `Car idx=4` ở gần (`3.97 m`) bị che khuất một phần (`occluded = 1`) |
| `s02f52` | Hướng `-y` (trùng hướng `cam01`) | Có 3 xe ngược chiều (`yaw_deg ≈ 88.3°–91.8°`) và 2 xe cùng chiều (`yaw_deg = 271.0°`) phân bố từ `5.76 m` đến `23.10 m` |
| `s03f01` | Hướng `-y` (trùng hướng `cam01`) | Có 5 xe cùng chiều (`yaw_deg ≈ 269.9°–271.7°`); xe `idx=5` ở `20.98 m` có thân cao (`height = 2.350 m`) |
| `s03f46` | Hướng `-y` (trùng hướng `cam01`) | Chỉ có 4 xe con rõ trong phạm vi `4.52 m` đến `28.52 m`; các cụm điểm xa hơn quá thưa không đủ cơ sở gán |
| `s04f26` | Hướng `-y` (trùng hướng `cam01`) | Có 3 xe cùng chiều bên phải (`x < 0`, `yaw_deg = 270.7°`) và 3 xe ngược chiều bên trái (`x > 0`, `yaw_deg ≈ 92.3°–95.6°`); mặt đường ở xa `20–25 m` hơi thấp dần |
| `s04f54` | Hướng `-y` (trùng hướng `cam01`) | Có xe `Car idx=2` ở khoảng cách xa nhất trong bài (`dist_m = 40.51 m`, chỉ còn `70` điểm LiDAR) |
| `s05f17` | Hướng `-y` (trùng hướng `cam01`) | Có xe tải lớn `Truck idx=4` (`length = 10.550 m, width = 3.200 m, height = 3.750 m`, `yaw_deg = 90.0°`) ở `dist_m = 13.00 m` với mật độ điểm rất dày (`6426` điểm) |
| `s05f43` | Hướng `-y` (trùng hướng `cam01`) | Có 3 `Car` và 1 `Truck` (`idx=2`, `length = 5.130 m`) rõ trong khoảng `5.31 m` đến `17.31 m`; vùng xa ít điểm |

## 3. Ba cuboid có hướng dẫn (Bước 3)
Ảnh: `t2-a-car.png`, `t2-b-bus-truck.png`, `t2-c-ped-cyclist.png`.

| Vật | Frame, idx | Class | Mũi tên yaw đúng hướng? | Đáy chạm đường? | Đã kiểm Top/Side/Front + ảnh? | Kích thước trong bảng 4.3? |
|---|---|---|---|---|---|---|
| (a) xe con gần | `s01f00, idx=1` | `Car` | Đúng hướng (`yaw_deg = 268.9°`) | Đáy chạm đường (`z_bottom = -1.834 m`, `ground_gap_m = 0.01 m`) | Đã kiểm Top/Side/Front và `preview` | Đúng chuẩn (`l = 5.000 m, w = 1.670 m, h = 1.780 m`) |
| (b) xe buýt hoặc xe tải | `s01f00, idx=8` | `Bus` | Đúng hướng (`yaw_deg = 0.0°`) | Đáy chạm đường (`z_bottom = -1.676 m`, `ground_gap_m = 0.13 m`) | Đã kiểm Top/Side/Front và `preview` | Đúng chuẩn (`l = 7.000 m, w = 3.000 m, h = 3.670 m`) |
| (c) người đi bộ hoặc xe hai bánh | `s01f00, idx=3` | `Cyclist` | Đúng hướng (`yaw_deg = 274.8°`) | Đáy chạm đường (`z_bottom = -1.670 m`, `ground_gap_m = 0.03 m`) | Đã kiểm Top/Side/Front và `preview` | Đúng chuẩn (`l = 2.000 m, w = 1.000 m, h = 1.670 m`) |

- **Vòng hiệu chuẩn và vật bỏ qua:** Qua vòng hiệu chuẩn, cả lớp thống nhất quy tắc căn mũi tên đỏ (`+X` của cuboid) dọc thân xe hướng thẳng về phía đầu xe (xe đi cùng chiều phía trước hướng `-y` có `yaw_deg ≈ 270°`, xe ngược chiều hướng `+y` có `yaw_deg ≈ 90°`), không xoay hộp ở khung Side/Front, ước lượng phần thể tích bị khuất dựa theo kích thước chuẩn bảng 4.3 và hạ đáy hộp chạm sát mặt đường (`z_bottom ≈ -1.7 m`). Các cụm điểm quá thưa ở xa trên `30–40 m` (như trong frame `s03f46` và `s05f43`, chỉ thấy dưới 15–20% thể tích, không rõ hình khối trên LiDAR và bị khuất trên camera) được bỏ qua theo quy tắc `LOW_VISIBILITY` để không đoán mò.

## 4. Bốn hiện tượng chất lượng (Bước 4)
Nguyên nhân ghi là **giả thuyết**. Frame mượn thì ghi rõ nguồn: task chung, hoặc MSSV của bạn cho mượn.

| Hiện tượng | Frame / tên ảnh / nguồn | Nguyên nhân (giả thuyết) | Thấy gì trên point cloud | Hệ quả khi gán (theo guideline) |
|---|---|---|---|---|
| 1. Bóng che | `s02f12` (hoặc `s01f00`) / `t4-1-bong-che.png` / task `D13-2A202603147` | **Giả thuyết:** Vật thể đứng gần LiDAR chắn đường truyền thẳng của chùm tia laser, tạo vùng khuất hình quạt phía sau vật đó | Phía sau xe ở gần xuất hiện dải trống không có điểm mặt đường và làm vật phía sau bị khuyết mất một phần cụm điểm thân xe | Bật cờ `occluded = 1` (như `s02f12, idx=4`), vẽ cuboid bao trọn cả phần thể tích bị che suy luận được theo kích thước chuẩn bảng 4.3 |
| 2. Thưa ở xa | `s04f54, idx=2` (`dist_m = 40.51 m`) / `t4-2-thua-o-xa.png` / task `D13-2A202603147` | **Giả thuyết:** Ở khoảng cách xa (`> 30–40 m`), góc phân kỳ giữa 40 tia LiDAR dọc và bước góc quét ngang mở rộng làm khoảng cách giữa hai tia liền kề lớn hơn | Xe con ở `40.51 m` (`s04f54, idx=2`) chỉ còn `70` điểm (so với `2822–5018` điểm ở dưới `5 m`), các cụm xa hơn chỉ còn vài vệt điểm mảnh | Nếu còn nhận ra khoảng `15–20%` thể tích và đối chiếu được trên camera thì vẽ cuboid đúng kích thước chuẩn; nếu dưới `15–20%` thể tích thì bỏ qua (`LOW_VISIBILITY`) |
| 3. Phản xạ thấp / ít điểm bất thường | `s01f54` (hoặc `s04f26`) / `t4-3-phan-xa-thap.png` / task `D13-2A202603147` | **Giả thuyết:** Bề mặt sơn xe tối màu/hấp thụ tia hồng ngoại, bề mặt kính nghiêng phản xạ gương lệch hướng, hoặc góc tới của tia laser quá lớn | Một số vùng trên thân xe/kính xe ở cự ly trung bình (`10–20 m`) bị rỗng điểm bất thường dù không bị vật khác che chắn phía trước | Không thu nhỏ cuboid theo phần điểm còn lại; đối chiếu ảnh camera để xác nhận loại xe và giữ kích thước hộp bao đủ thân xe theo bảng 4.3 |
| 4. Điểm lạ | `s01f00` / `t4-4-diem-la.png` / task `D13-2A202603147` | **Giả thuyết:** Nhiễu phản xạ từ bụi/khói, tia phản xạ đa đường (multipath) qua mặt kính/kim loại hoặc nhiễu gần cảm biến | Xuất hiện cụm điểm nhỏ lơ lửng hoặc rời rạc không gắn với mặt đường và không có vật thể tương ứng khi kiểm tra trên 7 ảnh camera | Tuyệt đối không gán nhãn (không vẽ cuboid) cho các cụm điểm lạ này |

## 5. Số từ cuboid của tôi (Bước 5)
Lấy từ `submission/D13-2A202603147-bangA.csv`; số điểm lấy từ cột `points_in` của `submission/D13-2A202603147-tukiem.csv`. Mặt đường khoảng −1,7 m: z đáy lệch hơn 0,3 m là lơ lửng hoặc chìm.

| Cuboid (frame, idx, class) | z (m) | h (m) | z đáy = z − h/2 (m) | dist_m (m) | r = √(x²+y²+z²) (m) | yaw_deg (°) | Nhận xét (lơ lửng / chìm / chạm đường) |
|---|---|---|---|---|---|---|---|
| `s01f00, idx=1, Car` | `-0.944` | `1.780` | `-1.834` | `3.34` | `3.47` | `268.9°` | Chạm đường (`ground_gap_m = 0.01 m`, lệch `0.134 m < 0.3 m` so với `-1.7 m`) |
| `s01f00, idx=3, Cyclist` | `-0.835` | `1.670` | `-1.670` | `10.81` | `10.84` | `274.8°` | Chạm đường (`ground_gap_m = 0.03 m`, sát mức chuẩn `-1.7 m`) |
| `s04f54, idx=2, Car` | `-1.288` | `1.550` | `-2.063` | `40.51` | `40.53` | `92.3°` | Chạm mặt đường tại chỗ ở xa (`ground_gap_m = 0.19 m`, mặt đường ở `~40.5 m` thấp hơn gần gốc) |

- **(a) Cuboid gần nhất và xa nhất:**
  - **Cuboid gần nhất:** `s01f00, idx=1 (Car)` có tọa độ tâm `(x = 3.315 m, y = -0.384 m, z = -0.944 m)` → khoảng cách ngang `dist_m = √(3.315² + (-0.384)²) = 3.34 m`, khoảng cách 3D `r = √(3.315² + (-0.384)² + (-0.944)²) = 3.47 m` (`3.468 m`).
  - **Cuboid xa nhất:** `s04f54, idx=2 (Car)` có tọa độ tâm `(x = 7.716 m, y = -39.768 m, z = -1.288 m)` → khoảng cách ngang `dist_m = √(7.716² + (-39.768)²) = 40.51 m`, khoảng cách 3D `r = √(7.716² + (-39.768)² + (-1.288)²) = 40.53 m` (`40.530 m`).
- **(b) So sánh kích thước `l, w, h` với bảng 4.3:**
  - **Xe hai bánh (`s01f00, idx=3, Cyclist`):** `length = 2.000 m, width = 1.000 m, height = 1.670 m` (đạt chuẩn bảng 4.3). Với người đi bộ `s01f00, idx=5 (Pedestrian)` có `length = 0.550 m, width = 1.780 m, height = 1.890 m`: chiều `width = 1.780 m` vượt ngoài khoảng `0.2–1.3 m` do cụm điểm ở khoảng cách `22.19 m` bị thưa và hộp bị kéo dư theo chiều ngang.
  - **Xe con (`s01f00, idx=1, Car`):** `length = 5.000 m, width = 1.670 m, height = 1.780 m` (cả 3 chiều đều nằm trong khoảng chuẩn bảng 4.3: dài `3.8–5.3 m`, rộng `1.6–2.2 m`, cao `1.4–2.0 m`). Riêng cuboid `s03f01, idx=5 (Car)` có `length = 4.670 m, width = 2.210 m, height = 2.350 m` bị cờ `height=2.35 ngoài khoảng 1.2–2.3 của Car` do hộp bao hơi cao phần nóc của xe gầm cao ở khoảng cách `20.98 m`.
- **(c) Góc `yaw_deg` của xe cùng chiều và ngược chiều:**
  - **Xe cùng chiều phía trước (`-y`):** `s01f54, idx=3 (Car)` tại `(x = 0.033 m, y = -9.435 m)` có `yaw_deg = 272.9°`, và `s03f01, idx=1 (Car)` tại `(x = -0.338 m, y = -8.584 m)` có `yaw_deg = 271.0°` — rất gần `270°`.
  - **Xe ngược chiều (`+y`):** `s02f12, idx=2 (Car)` tại `(x = 3.932 m, y = -9.688 m)` có `yaw_deg = 88.0°`, và `s02f52, idx=2 (Car)` tại `(x = 4.751 m, y = -8.260 m)` có `yaw_deg = 91.8°` — rất gần `90°`.

Mật độ theo khoảng cách (4 cuboid ở các `dist_m` khác nhau từ `submission/D13-2A202603147-tukiem.csv`):

| Cuboid (frame, idx, class) | dist_m (m) | Số điểm (`points_in`) |
|---|---|---|
| `s01f00, idx=1, Car` (bin `0–10 m`) | `3.34` (`3.3`) | `2822` |
| `s01f00, idx=6, Car` (bin `10–20 m`) | `16.97` (`17.0`) | `815` |
| `s04f54, idx=6, Car` (bin `20–30 m`) | `27.41` (`27.4`) | `213` |
| `s04f54, idx=2, Car` (bin `30–50 m`) | `40.51` (`40.5`) | `70` |

- **Quy luật mật độ và tự kiểm (`selfcheck`):**
  - **Quy luật:** Khi khoảng cách `dist_m` tăng gấp khoảng `2–2.5` lần thì số điểm LiDAR trong hộp (`points_in`) giảm xuống chỉ còn khoảng `1/3` đến `1/4` (từ `2822` điểm ở `3.34 m` giảm xuống `815` điểm ở `16.97 m`, `213` điểm ở `27.41 m` và còn `70` điểm ở `40.51 m`). Kết quả này bám rất sát đường cong trung vị của xe con trên slide bài giảng (`~1968` điểm ở `0–10 m`, `~718` ở `10–20 m`, `~214` ở `20–30 m`, `~72` ở `30–40 m`).
  - **Tự kiểm và chỉnh sửa sau `preview` / `selfcheck`:** Đã kiểm tra và chỉnh lại độ cao đáy `z_bottom` cùng góc xoay `yaw` của các hộp xe con ở `s01f00` (`idx=1, idx=2`) và `s02f12` (`idx=4`, bật `occluded=1`) để khớp với mặt đường và hướng xe trên camera. Ba cảnh báo giữ nguyên gồm: `s01f00, idx=5 (Pedestrian)` (`width=1.78 m`), `s03f01, idx=5 (Car)` (`height=2.35 m`), và `s04f26, idx=4 (Car)` (`82 điểm sát ngay ngoài mép` do có cụm điểm của mặt đường/vật lân cận sát mép hộp ở khoảng cách `20.93 m`, trong khi `ground_gap_m = 0.01 m` đã sát mặt đường).
- **Đổi kiểm:** Đã đối chiếu chéo 2 con số trong Mục 5 (`s01f00, idx=1` với `dist_m = 3.34 m, points_in = 2822` và `s04f54, idx=2` với `dist_m = 40.51 m, points_in = 70`) với Bảng A và Phiếu tự kiểm, kết quả khớp 100%.

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc
- **(a) Ý nghĩa của nhãn 3D:**
  1. Model 3D detection và hệ thống tự hành cần ước lượng vị trí, kích thước và khoảng cách thực tế trong không gian vật lý 3 chiều để lập kế hoạch di chuyển an toàn, nên nhãn 3D phải đo bằng **mét (m)** thay vì pixel trên ảnh 2D (vốn phụ thuộc vào phép chiếu phối cảnh và tiêu cự camera).
  2. Vật ở xa hoặc bị che khuất có mật độ điểm rất thưa nên cần quy tắc ngưỡng nhận diện (15–20% thể tích) và cờ `occluded` để tránh gán nhãn đoán mò làm nhiễu tập huấn luyện của model.
- **(b) Cách dùng AI:**
  1. Công cụ: Antigravity AI Assistant.
  2. Câu hỏi chính: Hướng dẫn kiểm tra và khắc phục các cảnh báo, lỗi định dạng repo/báo cáo của `check_submission.py`.
  3. Cách dùng kết quả: Dùng để sửa cấu trúc tiêu đề `README.md`, khung báo cáo và khởi tạo Git repo đúng quy định trước khi push lên GitHub.
- **(c) Điều còn chưa chắc:** Một số cụm điểm ở xa ngoài 35–40 m có mật độ điểm rất thưa, khó phân biệt rõ ranh giới phần đầu/đuôi xe nếu bị vật khác che khuất trên camera.

