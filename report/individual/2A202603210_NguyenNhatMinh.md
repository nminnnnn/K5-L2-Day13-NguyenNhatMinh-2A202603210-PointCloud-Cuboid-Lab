# Báo cáo Day 13 — Nguyễn Nhật Minh, 2A202603210

## 1. Kết luận trước

- Họ và tên: Nguyễn Nhật Minh.
- MSSV: 2A202603210.
- Hệ điều hành: Windows.
- Task cá nhân: D13-2A202603210.
- Phiên bản CVAT Server: 2.74.1.
- Trình duyệt: Chrome
- MSSV bạn kiểm chéo: 2A202603119.
- 10 frame: s01f30, s01f55, s02f18, s02f51, s03f06, s03f25, s04f08, s04f39, s05f23, s05f46.

Em đã hoàn thành 64 cuboid trên 10 frame, gồm 57 Car, 4 Truck, 1 Bus, 1 Pedestrian và 1 Cyclist. Các frame s02f51, s03f25, s04f39, s05f23 và s05f46 có ít cuboid nhất, mỗi frame có 5 cuboid. Riêng s03f25 có nhiều vật thể ở xa, mật độ điểm thấp nên việc gán nhãn khó hơn.

Kết quả selfcheck mới nhất có 4/64 cuboid được cảnh báo, chiếm 6,25%, giảm so với 6 cảnh báo ban đầu. Em tự tin nhất về việc phân loại và xác định kích thước xe gần. Phần còn hạn chế là xác định đáy cuboid và kích thước vật thể ở xa.

Cách sử dụng CVAT:

- Thay đổi góc nhìn bằng thao tác xoay, di chuyển, thu phóng và quan sát từ trên xuống.
- Chọn Cuboid Shape, chọn class, nháy đúp vào vật thể và điều chỉnh hộp bằng Top, Side, Front.
- Bật Cuboid orientation để kiểm tra hướng và sử dụng preview để đối chiếu cuboid với ảnh camera.

## 2. Đọc frame (Bước 2)

Hai frame được chọn để đọc kỹ là s01f30 và s01f55.

Ảnh: `t1-s01f30-3d.png`, `t1-s01f30-top.png`, `t1-s01f55-3d.png`, `t1-s01f55-top.png`.

| Frame | Hướng và hệ trục | Mặt đường | Vật xa nhất đã gán | Loại vật thấy | Vật phía sau/hai bên | Cụm chưa đọc được |
|---|---|---|---|---|---|---|
| s01f30 | Phía trước dự kiến ở cạnh dưới; +x sang phải, +y lên trên | Tương đối phẳng, thấp hơn LiDAR khoảng 1,7 m theo mốc tham chiếu | 50,59 m | Car, Truck, cây, công trình | Xe và cây hai bên đường | Cụm điểm thưa ở xa |
| s01f55 | Phía trước dự kiến ở cạnh dưới; +x sang phải, +y lên trên | Tương đối phẳng, thấp hơn LiDAR khoảng 1,7 m theo mốc tham chiếu | 20,55 m | Car, cây, công trình | Nhiều xe ở các làn bên cạnh | Cụm điểm bị che bởi xe khác |

Theo `docs/COORDINATES.md`, hệ VF có +x hướng sang trái xe, +y ra sau, +z lên trên; phía trước xe là −y. Vị trí các trục trên màn hình phụ thuộc góc nhìn. Cần tự xác nhận cạnh ảnh và chiều trục trong hai ảnh Top đã ghi ở trên.

Ba thông tin camera cung cấp rõ hơn point cloud:

- Màu sắc và đặc điểm bên ngoài của phương tiện.
- Vạch kẻ đường, đèn tín hiệu và các chi tiết giao thông.
- Hình dạng đầu và đuôi xe để hỗ trợ xác định hướng.

Ba thông tin LiDAR cung cấp mà ảnh camera đơn lẻ không trực tiếp đo được:

- Khoảng cách thực tế từ cảm biến đến vật thể.
- Kích thước và tọa độ vật thể trong không gian 3D.
- Độ cao của vật thể so với mặt đường.

### Tám frame còn lại

| Frame | Hướng phía trước theo hệ tọa độ VF | Đặc điểm ghi nhận |
|---|---|---|
| s02f18 | −y | Có người đi bộ và nhiều xe con |
| s02f51 | −y | Các xe phân bố ở nhiều khoảng cách |
| s03f06 | −y | Có xe gần LiDAR khoảng 3,15 m |
| s03f25 | −y | Nhiều xe xa, số điểm thấp |
| s04f08 | −y | Có xe buýt và xe tải |
| s04f39 | −y | Có xe gần với mật độ điểm cao |
| s05f23 | −y | Có Cyclist và xe bị che khuất |
| s05f46 | −y | Có hai Car được đánh dấu occluded |

## 3. Ba cuboid có hướng dẫn (Bước 3)

Ảnh: `t2-a-car.png`, `t2-b-bus-truck.png`, `t2-c-ped-cyclist.png`.

| Vật | Frame, idx | Class | Hướng yaw | Đáy hộp | Top/Side/Front | Kích thước |
|---|---|---|---|---|---|---|
| Xe con gần | s01f30, #1 (đối chiếu dự kiến) | Car | 276,5° | Gần mặt đường | Có ảnh ba góc nhìn | 4,335 × 2,047 × 1,603 m |
| Xe buýt | s04f08, #1 | Bus | 84,9° | Không có cảnh báo tự kiểm | Có ảnh ba góc nhìn | 12,349 × 2,833 × 3,398 m |
| Xe hai bánh | s05f23, #1 | Cyclist | 31,3° | Gần mặt đường | Có ảnh ba góc nhìn | 1,923 × 1,000 × 1,432 m |

Các kích thước trên lấy từ bảng A cuối cùng. Kích thước Car trong ảnh hướng dẫn ban đầu là 4,23 × 2,05 × 1,55 m, vì vậy cần xác nhận lại idx của ảnh này. Theo bản nháp, cả ba cuboid đều không có cảnh báo selfcheck về kích thước. Không có cảnh báo chưa đủ để kết luận kích thước phù hợp bảng 4.3 hoặc đáy chạm đường; cần tự kiểm tra riêng các tiêu chí này.

Khi hiệu chỉnh cuboid, em chú trọng hướng đầu xe, độ khít của hộp và vị trí đáy so với mặt đường. Với vật thể bị che khuất hoặc có quá ít điểm, cần tránh suy đoán kích thước khi không đủ cơ sở.

### Kết quả vòng hiệu chuẩn

Em chưa thực hiện vòng hiệu chuẩn trực tiếp với các bạn trong lớp nên chưa có kết quả so sánh mức độ sai lệch giữa các cuboid. Trong quá trình thực hành, em đã tự kiểm tra cuboid bằng các góc nhìn Top, Side, Front và kết quả selfcheck để điều chỉnh kích thước, hướng và vị trí đáy hộp.

### Các vật thể bỏ qua vì không đủ cơ sở

Đối với những cụm điểm ở xa, có mật độ điểm thấp hoặc bị che khuất, em ưu tiên không gán nhãn khi chưa đủ cơ sở xác định loại vật thể, kích thước và hướng. Các trường hợp này có thể thuộc nhóm LOW_VISIBILITY hoặc UNCERTAIN_CLASS.

Em chưa ghi lại danh sách cụ thể về frame và vị trí của các vật thể đã bỏ qua.

## 4. Bốn hiện tượng chất lượng (Bước 4)

| Hiện tượng | Frame / ảnh / nguồn | Nguyên nhân giả thuyết | Quan sát point cloud | Hệ quả khi gán |
|---|---|---|---|---|
| Bóng che | s05f46 / `t4-1-bong-che.png` / task cá nhân | Vật thể phía trước chắn tia LiDAR | Một số xe có phần điểm không đầy đủ; bảng A có hai Car được đánh dấu occluded | Đối chiếu camera, tránh thu nhỏ hộp chỉ theo phần nhìn thấy |
| Thưa ở xa | s05f46 / `t4-2-thua-o-xa.png` / task cá nhân | Mật độ tia giảm theo khoảng cách | Xe xa có ít điểm hơn xe gần | Khó xác định kích thước và hướng chính xác |
| Phản xạ thấp | s05f23 / `t4-3-phan-xa-thap.png` / task cá nhân | Có thể do đặc tính phản xạ của bề mặt hoặc góc tới | Một số cuboid chứa ít điểm hơn các cuboid khác; chưa tách được ảnh hưởng của khoảng cách và che khuất | Kết hợp camera, không khẳng định vật thể chỉ từ số điểm |
| Điểm lạ | s02f18 / `t4-4-diem-la.png` / task cá nhân | Có thể do nhiễu hoặc phản xạ nhiều đường | Có các cụm điểm rời rạc ngoài những vật thể đã gán nhãn; chưa xác nhận được là nhiễu | Không gán cuboid cho cụm điểm chưa đủ cơ sở |

Các nguyên nhân trên là giả thuyết dựa trên quan sát. Hai hiện tượng phản xạ thấp và điểm lạ cần thêm bằng chứng để xác nhận.

## 5. Số từ cuboid của tôi (Bước 5)

Dữ liệu lấy từ `D13-2A202603210-bangA.csv` và `D13-2A202603210-tukiem.csv`.

Công thức:

- z đáy = z − h/2.
- r = √(x² + y² + z²).
- Mốc mặt đường tham chiếu: −1,7 m.

| Cuboid | z (m) | h (m) | z đáy (m) | dist_m (m) | r (m) | yaw_deg | Nhận xét theo mốc −1,7 m |
|---|---:|---:|---:|---:|---:|---:|---|
| s01f30, #1, Car | −0,892 | 1,603 | −1,693 | 8,02 | 8,07 | 276,5° | Gần chạm đường |
| s01f30, #4, Car | −1,318 | 1,788 | −2,212 | 24,17 | 24,21 | 89,7° | Thấp hơn mốc 0,512 m |
| s01f55, #1, Car | −0,962 | 1,494 | −1,709 | 6,77 | 6,84 | 268,5° | Gần chạm đường |

Mốc −1,7 m chỉ mang tính tham chiếu. Mặt đường cục bộ có thể khác, ví dụ s01f30 #4 có ground_gap_m = 0,04 m và không bị selfcheck cảnh báo chìm.

### Khoảng cách gần nhất và xa nhất

Phạm vi gán nhãn của lab là tâm vật cách LiDAR tối đa 50 m. Cần tự rà lại các vật ngoài phạm vi trong bản nháp; nếu sửa task thì Save, chạy lại `sync` và tự cập nhật các số liên quan trong báo cáo.

- Gần nhất: s03f06, #6, Car; dist_m = 3,15 m; r = 3,30 m.
- Xa nhất: s03f25, #5, Car; dist_m = 80,63 m; r = 80,63 m.

### So sánh kích thước

- Car s01f30 #1: dài 4,335 m, rộng 2,047 m, cao 1,603 m. Cần tự đối chiếu từng chiều với đúng loại xe trong bảng 4.3 (`docs/GUIDELINE.md`); chưa xác nhận kết luận cả ba chiều đều trong khoảng.
- Cyclist s05f23 #1: dài 1,923 m, rộng 1,000 m, cao 1,432 m. Theo bản nháp, phiếu tự kiểm không có cảnh báo kích thước; cần tự so sánh từng chiều với đúng loại xe hai bánh trong bảng 4.3.
- Car s01f30 #7 có chiều cao 2,658 m, vượt giới hạn tham chiếu 2,3 m. Cần kiểm tra lại chiều cao và class.

### Góc yaw

- Car s02f18 #1: yaw = 268,5°, gần 270°, phù hợp hướng cùng chiều theo quy ước bài lab.
- Car s01f30 #4: yaw = 89,7°, gần 90°, phù hợp hướng ngược chiều theo quy ước bài lab.

### Mật độ theo khoảng cách

| Cuboid | dist_m (m) | Số điểm |
|---|---:|---:|
| s01f30, #1, Car | 8,02 | 1683 |
| s01f30, #2, Car | 16,79 | 505 |
| s01f30, #4, Car | 24,17 | 245 |
| s03f25, #4, Car | 68,12 | 31 |

Khi khoảng cách tăng khoảng 8,49 lần, từ 8,02 m lên 68,12 m, số điểm giảm từ 1683 xuống 31, chỉ còn khoảng 1,84%.

Kết quả phù hợp với xu hướng giảm mật độ điểm trong slide bài giảng: khoảng 1968 điểm ở 0–10 m, 718 điểm ở 10–20 m, 214 điểm ở 20–30 m và 19 điểm ở 60–80 m. Số điểm thực tế còn phụ thuộc vào kích thước, góc quan sát và mức độ che khuất của vật thể.

### Tự kiểm và chỉnh sửa

Sau khi chạy preview và selfcheck, em kiểm tra lại các cuboid có cảnh báo về kích thước và vị trí đáy.

| Cuboid | Trước khi sửa | Sau khi sửa |
|---|---|---|
| s03f06, #3, Truck | Chiều cao 1,69 m | Chiều cao 1,750 m, hết cảnh báo kích thước |
| s04f08, #7, Car | Chiều rộng 2,42 m | Chiều rộng 2,300 m, hết cảnh báo kích thước |
| s01f30, #7, Car | Chiều dài 6,20 m | Chiều dài 6,000 m |
| s01f30, #8, Car | Chiều dài 6,20 m | Chiều dài 6,000 m |

Kết quả tự kiểm cuối cùng còn 4 cuboid có cảnh báo:

- s01f30 #7: chiều cao 2,658 m vượt khoảng tham chiếu.
- s01f30 #8: có 27 điểm sát ngoài mép hộp.
- s05f46 #2: đáy thấp hơn mặt đường cục bộ 0,25 m.
- s05f46 #3: đáy thấp hơn mặt đường cục bộ 0,27 m.

Em chưa tiếp tục thay đổi các cuboid này chỉ để loại bỏ cảnh báo vì cần đối chiếu thêm với camera và point cloud. Cảnh báo tự động là gợi ý kiểm tra, không phải kết luận chắc chắn về lỗi gán nhãn.

**Kiểm chéo:** Bạn kiểm chéo có MSSV 2A202603119. Hai thông số được đối chiếu giữa báo cáo của em và Bảng A của chính em là chiều cao cuboid (`h`) và khoảng cách (`dist_m`).

| Frame, idx | Số được kiểm | Kết quả |
|---|---|---|
| s01f30, #1 | h = 1,603 m; dist_m = 8,02 m | Khớp |
| s01f30, #4 | h = 1,788 m; dist_m = 24,17 m | Khớp |
| s01f55, #1 | h = 1,494 m; dist_m = 6,77 m | Khớp |

Kết quả: 3/3 cuboid và 6/6 giá trị đều khớp, không phát hiện sai lệch, không cần chỉnh sửa.

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc

Nhãn 3D sử dụng đơn vị mét thay vì pixel vì model cần học vị trí, kích thước và hướng thực tế của vật thể trong không gian. Vật thể ở xa hoặc thưa điểm cần quy tắc riêng để hạn chế việc model học từ các nhãn thiếu chính xác.

Sử dụng AI:

- Công cụ: ChatGPT.
- Câu hỏi chính: hướng dẫn sử dụng CVAT, kiểm tra số lượng cuboid, giải thích cảnh báo selfcheck và tổng hợp số liệu báo cáo.
- Cách sử dụng đã ghi trong bản nháp: tham khảo hướng dẫn để thao tác trên CVAT, đối chiếu ảnh và tính toán từ các file CSV do bài lab xuất ra. Trong lần biên tập này, ChatGPT đọc tài liệu repo và chỉnh bản nháp do em cung cấp, không đọc Bảng A, phiếu tự kiểm hoặc ảnh dữ liệu; các số liệu và quan sát trong bản nháp cần em tự xác nhận. Phần sử dụng AI trước đó cần được khai báo đúng thực tế theo `RULES.md`.

Điều em còn chưa chắc là cách xác định kích thước đầy đủ của xe bị che khuất, hướng của vật thể ở xa và vị trí đáy trên mặt đường không bằng phẳng. Qua bài lab, em hiểu rõ hơn vai trò của việc kết hợp camera và LiDAR, kiểm tra nhiều góc nhìn và tự kiểm dữ liệu trước khi sử dụng nhãn để huấn luyện model.
