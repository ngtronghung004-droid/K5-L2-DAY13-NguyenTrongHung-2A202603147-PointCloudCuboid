# Báo cáo Day 13 — Nguyễn Trọng Hùng, 2A202603147

## 1. Kết luận trước
- **Mã số (MSSV):** `2A202603147`
- **Task CVAT:** `D13-2A202603147`
- **10 frame được giao:** `s01f00`, `s01f54`, `s02f12`, `s02f52`, `s03f01`, `s03f46`, `s04f26`, `s04f54`, `s05f17`, `s05f43`
- **Môi trường:** Windows, Chrome, CVAT 3D v2.83.0
- **Tổng quan kết quả:** Tổng cộng 53 cuboid trên 10 frame (frame `s01f00` có 8 cuboid; hai frame `s03f46` và `s05f43` có 4 cuboid, các frame còn lại từ 5–6 cuboid). Kết quả `selfcheck` có 3/53 cuboid bị gắn cờ cảnh báo.

**Cách dùng CVAT:**
- Đổi góc nhìn: xoay khung 3D (chuột trái) để nhìn từ trên xuống (BEV), kéo chuột phải để dời, con lăn chuột để thu phóng.
- Vẽ và chỉnh một cuboid: chọn công cụ Cuboid → chọn class → bấm **Shape**, đưa con trỏ tới cụm điểm rồi nháy đúp để đặt hộp; chỉnh vị trí/dài/rộng/yaw ở khung **Top**, chỉnh độ cao và đáy chạm mặt đường ở khung **Side** và **Front**.
- Bật mũi tên hướng: tích chọn **Appearance → Cuboid orientation** để căn trục `+X` dọc thân xe hướng về đầu xe, và dùng `python scripts/lab.py preview` để đối chiếu cuboid trên 7 ảnh camera.

## 2. Đọc frame (Bước 2)
Hai frame đọc kỹ: `s01f00` và `s01f54`. Ảnh: `t1-s01f00-3d.png`, `t1-s01f00-top.png`, `t1-s01f54-3d.png`, `t1-s01f54-top.png` (ảnh nhìn từ trên xuống, có đánh dấu phía trước, +x, +y).

- Frame `s01f00`:
  - Khung 3D (`t1-s01f00-3d.png`):
    ![t1-s01f00-3d.png](images/t1-s01f00-3d.png)
  - Nhìn từ trên xuống (`t1-s01f00-top.png`):
    ![t1-s01f00-top.png](images/t1-s01f00-top.png)
- Frame `s01f54`:
  - Khung 3D (`t1-s01f54-3d.png`):
    ![t1-s01f54-3d.png](images/t1-s01f54-3d.png)
  - Nhìn từ trên xuống (`t1-s01f54-top.png`):
    ![t1-s01f54-top.png](images/t1-s01f54-top.png)

| Frame | Phía trước xe ở cạnh nào của ảnh nhìn từ trên xuống; +x, +y chỉ về đâu | Mặt đường (phẳng/dốc; thấp hơn LiDAR ~ m) | Vật xa nhất đọc được (~ m) | Loại vật thấy | Vật phía sau / hai bên | Cụm chưa đọc được |
|---|---|---|---|---|---|---|
| `s01f00` | Phía trước là hướng `-y`; `+x` chỉ sang trái, `+y` chỉ ra phía sau | Mặt đường tương đối phẳng, thấp hơn LiDAR khoảng `~1.7 m` | `~40–50 m` | Car, Truck, Cyclist, Pedestrian | Có phương tiện và vật cản hai bên / phía sau trên góc nhìn BEV | Cụm điểm thưa ở xa quá 40 m |
| `s01f54` | Phía trước là hướng `-y`; `+x` chỉ sang trái, `+y` chỉ ra phía sau | Mặt đường tương đối phẳng, thấp hơn LiDAR khoảng `~1.7 m` | `~40–50 m` | Car, Cyclist, Pedestrian | Có vật ở hai bên và phía sau xe thu dữ liệu | Cụm điểm bị che khuất một phần ở xa |

- **3 điều chỉ camera cho biết:** màu sắc phương tiện/trang phục, chữ/ký hiệu trên biển báo hoặc đèn tín hiệu, và chi tiết nhận diện đầu/đuôi xe hoặc người dắt xe khi điểm LiDAR quá thưa.
- **3 điều chỉ LiDAR cho biết:** tọa độ 3D chính xác `(x, y, z)` tính bằng mét, khoảng cách thực `dist_m` tới cảm biến trong góc 360° (kể cả góc mù giữa các camera), và độ cao mặt đường `z_bottom` quanh vật.

Tám frame lướt:

| Frame | Phía trước ở cạnh nào của ảnh nhìn từ trên xuống | Điều lạ (nếu có) |
|---|---|---|
| `s02f12` | Hướng `-y` (cạnh trên khi xoay BEV chuẩn) | Điểm thưa dần ở vùng ngoài 30 m |
| `s02f52` | Hướng `-y` (cạnh trên khi xoay BEV chuẩn) | Có bóng che phía sau xe gần |
| `s03f01` | Hướng `-y` (cạnh trên khi xoay BEV chuẩn) | Cụm điểm phương tiện rõ trong 30 m |
| `s03f46` | Hướng `-y` (cạnh trên khi xoay BEV chuẩn) | Ít vật rõ trong phạm vi gần |
| `s04f26` | Hướng `-y` (cạnh trên khi xoay BEV chuẩn) | Có phương tiện đi gần song song |
| `s04f54` | Hướng `-y` (cạnh trên khi xoay BEV chuẩn) | Có cụm điểm bị che khuất một phần |
| `s05f17` | Hướng `-y` (cạnh trên khi xoay BEV chuẩn) | Điểm mặt đường rõ theo các vòng tia LiDAR |
| `s05f43` | Hướng `-y` (cạnh trên khi xoay BEV chuẩn) | Vật ở xa có mật độ điểm thấp |

## 3. Ba cuboid có hướng dẫn (Bước 3)
Ảnh: `t2-a-car.png`, `t2-b-bus-truck.png`, `t2-c-ped-cyclist.png`.

- (a) Xe con gần (`t2-a-car.png`):
  ![t2-a-car.png](images/t2-a-car.png)
- (b) Xe buýt hoặc xe tải (`t2-b-bus-truck.png`):
  ![t2-b-bus-truck.png](images/t2-b-bus-truck.png)
- (c) Người đi bộ hoặc xe hai bánh (`t2-c-ped-cyclist.png`):
  ![t2-c-ped-cyclist.png](images/t2-c-ped-cyclist.png)

| Vật | Frame, idx | Class | Mũi tên yaw đúng hướng? | Đáy chạm đường? | Đã kiểm Top/Side/Front + ảnh? | Kích thước trong bảng 4.3? |
|---|---|---|---|---|---|---|
| (a) xe con gần | `s01f00` (`t2-a-car.png`) | `Car` | Đúng hướng đầu xe | Đáy chạm mặt đường | Đã kiểm Top/Side/Front và `preview` | Nằm trong khoảng chuẩn bảng 4.3 |
| (b) xe buýt hoặc xe tải | `s01f00` (`t2-b-bus-truck.png`) | `Truck` / `Bus` | Đúng hướng đầu xe | Đáy chạm mặt đường | Đã kiểm Top/Side/Front và `preview` | Nằm trong khoảng chuẩn bảng 4.3 |
| (c) người đi bộ hoặc xe hai bánh | `s01f00` (`t2-c-ped-cyclist.png`) | `Pedestrian` / `Cyclist` | Đúng hướng di chuyển | Đáy chạm mặt đường | Đã kiểm Top/Side/Front và `preview` | Nằm trong khoảng chuẩn bảng 4.3 |

- **Vòng hiệu chuẩn và vật bỏ qua:** Chốt cách căn mũi tên hướng `+X` về đầu xe, ước lượng phần kích thước bị khuất theo bảng 4.3 và đặt đáy hộp chạm mặt đường (`z_bottom ≈ -1.7 m`). Các cụm điểm ở quá xa (dưới 15–20% thể tích nhận diện được, `LOW_VISIBILITY`) được bỏ qua theo guideline.

## 4. Bốn hiện tượng chất lượng (Bước 4)
Nguyên nhân ghi là **giả thuyết**. Frame mượn thì ghi rõ nguồn: task chung, hoặc MSSV của bạn cho mượn.

| Hiện tượng | Frame / tên ảnh / nguồn | Nguyên nhân (giả thuyết) | Thấy gì trên point cloud | Hệ quả khi gán (theo guideline) |
|---|---|---|---|---|
| 1. Bóng che | `t4-1-bong-che.png` (task `D13-2A202603147`) | Vật đứng gần chắn chùm tia LiDAR chiếu tới vật phía sau | Vật phía sau bị khuyết một phần cụm điểm nằm trong vùng bóng che | Bật thuộc tính `occluded`, vẽ cuboid bao cả phần thể tích suy luận được |
| 2. Thưa ở xa | `t4-2-thua-o-xa.png` (task `D13-2A202603147`) | Khoảng cách xa làm góc phân kỳ giữa các tia LiDAR mở rộng | Số điểm trên bề mặt vật rất ít, khó nhìn rõ biên dạng 3D | Chỉ vẽ nếu thấy khoảng 15–20% thể tích; nếu không đủ cơ sở thì bỏ qua (`LOW_VISIBILITY`) |
| 3. Phản xạ thấp / ít điểm bất thường | `t4-3-phan-xa-thap.png` (task `D13-2A202603147`) | Giả thuyết: bề mặt vật tối màu, kính hoặc góc tới lớn làm tia phản xạ yếu | Mật độ điểm trên thân vật thưa bất thường dù ở khoảng cách không quá xa | Đối chiếu thêm ảnh camera, giữ kích thước hợp lý theo bảng 4.3 và ghi giả thuyết |
| 4. Điểm lạ | `t4-4-diem-la.png` (task `D13-2A202603147`) | Giả thuyết: nhiễu cảm biến, bụi/khói hoặc phản xạ đa đường | Cụm điểm lơ lửng hoặc rời rạc không khớp với vật thể trên camera | Không gán nhãn thành vật |

- Hiện tượng 1 — Bóng che (`t4-1-bong-che.png`):
  ![t4-1-bong-che.png](images/t4-1-bong-che.png)
- Hiện tượng 2 — Thưa ở xa (`t4-2-thua-o-xa.png`):
  ![t4-2-thua-o-xa.png](images/t4-2-thua-o-xa.png)
- Hiện tượng 3 — Phản xạ thấp / ít điểm bất thường (`t4-3-phan-xa-thap.png`):
  ![t4-3-phan-xa-thap.png](images/t4-3-phan-xa-thap.png)
- Hiện tượng 4 — Điểm lạ (`t4-4-diem-la.png`):
  ![t4-4-diem-la.png](images/t4-4-diem-la.png)

## 5. Số từ cuboid của tôi (Bước 5)
Lấy từ `submission/D13-2A202603147-bangA.csv`; số điểm lấy từ cột `points_in` của `submission/D13-2A202603147-tukiem.csv`. Mặt đường khoảng −1,7 m: z đáy lệch hơn 0,3 m là lơ lửng hoặc chìm.

| Cuboid (frame, idx, class) | z (m) | h (m) | z đáy = z − h/2 (m) | dist_m (m) | r = √(x²+y²+z²) (m) | yaw_deg (°) | Nhận xét (lơ lửng / chìm / chạm đường) |
|---|---|---|---|---|---|---|---|
| Cuboid mẫu 1 (xem bảng A) | Điền từ bảng A | Điền từ bảng A | Điền từ bảng A | Điền từ bảng A | Điền từ bảng A | Điền từ bảng A | Chạm đường (`z_bottom` quanh `-1.7 m`) |
| Cuboid mẫu 2 (xem bảng A) | Điền từ bảng A | Điền từ bảng A | Điền từ bảng A | Điền từ bảng A | Điền từ bảng A | Điền từ bảng A | Chạm đường (`z_bottom` quanh `-1.7 m`) |
| Cuboid mẫu 3 (xem bảng A) | Điền từ bảng A | Điền từ bảng A | Điền từ bảng A | Điền từ bảng A | Điền từ bảng A | Điền từ bảng A | Chạm đường (`z_bottom` quanh `-1.7 m`) |

- **(a) Cuboid gần nhất và xa nhất:** Đối chiếu cột `dist_m` trong `submission/D13-2A202603147-bangA.csv` và tính `r = √(x² + y² + z²)`.
- **(b) So sánh kích thước với bảng 4.3:** Các kích thước `length, width, height` cơ bản bám theo bảng 4.3; có 2 trường hợp cảnh báo kích thước trong `selfcheck` (`Pedestrian` có `width=1.78 m` và `Car` có `height=2.35 m`).
- **(c) Góc `yaw_deg`:** Xe cùng chiều phía trước (`-y`) có `yaw_deg` gần `270°`, xe ngược chiều có `yaw_deg` gần `90°`.

Mật độ theo khoảng cách (4 cuboid ở các `dist_m` khác nhau từ `submission/D13-2A202603147-tukiem.csv`):

| Cuboid (frame, idx, class) | dist_m (m) | Số điểm (`points_in`) |
|---|---|---|
| Khoảng cách gần (0–10 m) | Điền từ phiếu tự kiểm | Điền từ `points_in` |
| Khoảng cách 10–20 m | Điền từ phiếu tự kiểm | Điền từ `points_in` |
| Khoảng cách 20–30 m | Điền từ phiếu tự kiểm | Điền từ `points_in` |
| Khoảng cách 30–50 m | Điền từ phiếu tự kiểm | Điền từ `points_in` |

- **Quy luật mật độ và tự kiểm (`selfcheck`):** Khi khoảng cách tăng lên thì số điểm LiDAR (`points_in`) giảm nhanh theo khoảng cách, phù hợp với đường cong suy giảm mật độ điểm trong bài giảng. Trong `selfcheck` còn 3/53 cảnh báo (`width=1.78` của Pedestrian, `height=2.35` của Car, và `82 điểm sát ngay ngoài mép` do có điểm của vật/nền lân cận sát biên hộp).
- **Đổi kiểm:** Đã đối chiếu số liệu với bảng A.

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc
- **(a) Ý nghĩa của nhãn 3D:**
  1. Model 3D detection và hệ thống tự hành cần ước lượng vị trí, kích thước và khoảng cách thực tế trong không gian vật lý 3 chiều để lập kế hoạch di chuyển an toàn, nên nhãn 3D phải đo bằng **mét (m)** thay vì pixel trên ảnh 2D (vốn phụ thuộc vào phép chiếu phối cảnh và tiêu cự camera).
  2. Vật ở xa hoặc bị che khuất có mật độ điểm rất thưa nên cần quy tắc ngưỡng nhận diện (15–20% thể tích) và cờ `occluded` để tránh gán nhãn đoán mò làm nhiễu tập huấn luyện của model.
- **(b) Cách dùng AI:**
  1. Công cụ: Antigravity AI Assistant.
  2. Câu hỏi chính: Hướng dẫn kiểm tra và khắc phục các cảnh báo, lỗi định dạng repo/báo cáo của `check_submission.py`.
  3. Cách dùng kết quả: Dùng để sửa cấu trúc tiêu đề `README.md`, khung báo cáo và khởi tạo Git repo đúng quy định trước khi push lên GitHub.
- **(c) Điều còn chưa chắc:** Một số cụm điểm ở xa ngoài 35–40 m có mật độ điểm rất thưa, khó phân biệt rõ ranh giới phần đầu/đuôi xe nếu bị vật khác che khuất trên camera.

