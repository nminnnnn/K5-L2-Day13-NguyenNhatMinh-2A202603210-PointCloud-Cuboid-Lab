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

Bản ghi hiện tại có 64 cuboid trên 10 frame, gồm 57 Car, 4 Truck, 1 Bus, 1 Pedestrian và 1 Cyclist. Các số tổng hợp này được giữ từ bản nháp, chưa được em xác nhận lại trong lần sửa báo cáo này. Phản hồi rà soát ghi nhận hai frame có ít hơn 5 cuboid; vì vậy nhận định trước đây rằng mọi frame đều có ít nhất 5 cuboid chưa được xác nhận và được rút lại. Em chưa có kết quả rà lại độ phủ của hai frame đó trên CVAT. Mức 5 cuboid/frame là yêu cầu tối thiểu khi làm bài, không tự động đồng nghĩa bài bị trượt nếu thấp hơn mức này; việc bổ sung nhãn phải dựa trên vật thể đủ cơ sở quan sát, không chỉ để đủ số.

Theo bản nháp, kết quả selfcheck có 4/64 cuboid được cảnh báo, chiếm 6,25%, so với 6 cảnh báo ban đầu; em chưa xác nhận lại kết quả này sau phản hồi rà soát. Phần còn hạn chế là xác định đáy cuboid và kích thước vật thể ở xa.

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

Theo `docs/COORDINATES.md`, hệ VF có +x hướng sang trái xe, +y ra sau, +z lên trên; phía trước xe là −y. Vị trí các trục trên màn hình phụ thuộc góc nhìn. Em chưa xác nhận lại cạnh ảnh và chiều trục trong hai ảnh Top đã ghi ở trên.

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

Các file ảnh hiện có: `t2-a-car.png`, `t2-b-bus-truck.png`, `t2-c-ped-cyclist.png`. Việc có file chưa xác nhận nội dung ảnh hoặc việc kiểm đủ các góc nhìn. Bảng dưới giữ số đo từ bản nháp, chưa xác nhận lại sau phản hồi.

| Vật | Frame, idx | Class | Hướng yaw | Đáy hộp | Top/Side/Front | Kích thước |
|---|---|---|---|---|---|---|
| Xe con gần | s01f30, #1 (chưa xác nhận idx trong ảnh) | Car | 276,5° | Chưa xác nhận lại | Chưa xác nhận đủ Top/Side/Front | 4,335 × 2,047 × 1,603 m |
| Xe buýt | s04f08, #1 | Bus | 84,9° | Chưa xác nhận lại | Chưa xác nhận đủ Top/Side/Front | 12,349 × 2,833 × 3,398 m |
| Xe hai bánh | s05f23, #1 | Cyclist | 31,3° (số cũ; bị phản hồi sai hướng) | Chưa xác nhận lại | Chưa có kết quả kiểm đủ Top/Side/Front và camera | 1,923 × 1,000 × 1,432 m |

Theo bản nháp, các kích thước trên lấy từ bảng A; kích thước Car trong ảnh hướng dẫn ban đầu là 4,23 × 2,05 × 1,55 m, khác số đang dẫn nên idx của ảnh chưa được xác nhận. Em chưa sửa hướng Cyclist sau phản hồi, nên chưa có số yaw hoặc ảnh cập nhật sau sửa hướng. Giá trị 31,3° là số cũ, không được dùng làm kết luận hướng đúng. Việc đối chiếu từng chiều với bảng 4.3 và kiểm đáy chạm đường cũng chưa có kết quả xác nhận; không thể suy ra các tiêu chí này chỉ từ việc không có cờ selfcheck.

Các kết luận về hướng đầu xe, độ khít của hộp và vị trí đáy của ba cuboid mẫu còn chờ em kiểm trực tiếp trên CVAT.

### Kết quả vòng hiệu chuẩn

Em chưa thực hiện vòng hiệu chuẩn trực tiếp với các bạn trong lớp nên chưa có kết quả so sánh mức độ sai lệch giữa các cuboid. Bản nháp có ghi việc tự kiểm qua Top, Side, Front, nhưng chưa có kết quả xác nhận riêng cho từng cuboid mẫu.

### Các vật thể bỏ qua vì không đủ cơ sở

Đối với những cụm điểm ở xa, có mật độ điểm thấp hoặc bị che khuất, em ưu tiên không gán nhãn khi chưa đủ cơ sở xác định loại vật thể, kích thước và hướng. Các trường hợp này có thể thuộc nhóm LOW_VISIBILITY hoặc UNCERTAIN_CLASS.

Em chưa ghi lại danh sách cụ thể về frame và vị trí của các vật thể đã bỏ qua.

## 4. Bốn hiện tượng chất lượng (Bước 4)

| Hiện tượng | Frame / ảnh / nguồn | Nguyên nhân giả thuyết | Quan sát point cloud | Hệ quả khi gán |
|---|---|---|---|---|
| Bóng che | s05f46 / `t4-1-bong-che.png` / task cá nhân | Vật thể phía trước chắn tia LiDAR | Một số xe có phần điểm không đầy đủ; bảng A có hai Car được đánh dấu occluded | Đối chiếu camera, tránh thu nhỏ hộp chỉ theo phần nhìn thấy |
| Thưa ở xa | s05f46 / `t4-2-thua-o-xa.png` / task cá nhân | Mật độ tia giảm theo khoảng cách | Xe xa có ít điểm hơn xe gần | Khó xác định kích thước và hướng chính xác |
| Phản xạ thấp hoặc ít điểm bất thường | s05f23 / `t4-3-phan-xa-thap.png` / task cá nhân (frame theo bản nháp) | Phản xạ thấp là giả thuyết, chưa xác nhận nguyên nhân | Em thấy xe có ít điểm hơn các xe khác ở cùng khoảng cách; chưa ghi lại vị trí cụ thể của xe và chưa loại trừ ảnh hưởng của che khuất hoặc góc quan sát | Số điểm ít làm hạn chế cơ sở xác định hình dạng; quan sát này chưa đủ để kết luận nguyên nhân là phản xạ thấp |
| Điểm lạ — chưa xác định | s02f18 / `t4-4-diem-la.png` / task cá nhân (frame theo bản nháp) | Chưa đủ cơ sở nêu nguyên nhân | Em chỉ thấy các điểm mờ, không nhận biết được đó là gì nếu chỉ dùng LiDAR; chưa có vị trí và độ cao cụ thể được ghi lại | Chưa đủ cơ sở xác định đây là vật thể hay điểm lạ; không dùng quan sát mờ này làm căn cứ kết luận nhiễu |

Hai dòng bóng che và thưa ở xa được giữ theo ghi chép cũ, chưa được xác nhận lại trong lần sửa này. Hai dòng còn lại mô tả điều em thực tế quan sát được. So sánh với xe cùng khoảng cách hỗ trợ nhận định ít điểm bất thường, nhưng chưa chứng minh phản xạ thấp. Việc không nhận ra một cụm điểm bằng LiDAR cũng chưa chứng minh đó là nhiễu hoặc phản xạ nhiều đường; trường hợp điểm lạ còn thiếu bằng chứng để phân loại.

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

Phạm vi gán nhãn của lab là tâm vật cách LiDAR tối đa 50 m. Bản nháp còn dẫn vật ngoài phạm vi; em chưa có kết quả rà lại các vật này trên CVAT. Các giá trị dưới đây là số đã ghi trong bản nháp, không phải kết quả xác nhận lại.

- Gần nhất: s03f06, #6, Car; dist_m = 3,15 m; r = 3,30 m.
- Xa nhất: s03f25, #5, Car; dist_m = 80,63 m; r = 80,63 m.

### So sánh kích thước

- Car s01f30 #1: dài 4,335 m, rộng 2,047 m, cao 1,603 m. Chưa có kết quả đối chiếu từng chiều với đúng loại xe trong bảng 4.3 (`docs/GUIDELINE.md`).
- Cyclist s05f23 #1: dài 1,923 m, rộng 1,000 m, cao 1,432 m. Chưa có kết quả so sánh từng chiều với đúng loại xe hai bánh trong bảng 4.3.
- Car s01f30 #7 có chiều cao 2,658 m, vượt giới hạn tham chiếu 2,3 m theo bản nháp. Chưa có kết quả kiểm lại chiều cao và class trên CVAT.

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

Em đã chỉnh sửa các cuboid sai về chiều dài, chiều rộng hoặc chiều cao (L/W/H). Phần chỉnh sửa đã thực hiện mới giới hạn ở kích thước; em chưa sửa hướng Cyclist được nêu trong phản hồi. Bảng dưới giữ các trường hợp và số trước/sau từ bản nháp; em chưa xác nhận lại từng dòng, nên chưa dùng bảng này để khẳng định chính xác số cuboid đã sửa.

| Cuboid | Trước khi sửa (bản nháp) | Sau khi sửa (bản nháp, chưa xác nhận) |
|---|---|---|
| s03f06, #3, Truck | Chiều cao 1,69 m | Chiều cao 1,750 m, hết cảnh báo kích thước |
| s04f08, #7, Car | Chiều rộng 2,42 m | Chiều rộng 2,300 m, hết cảnh báo kích thước |
| s01f30, #7, Car | Chiều dài 6,20 m | Chiều dài 6,000 m |
| s01f30, #8, Car | Chiều dài 6,20 m | Chiều dài 6,000 m |

Theo bản nháp, kết quả tự kiểm còn 4 cuboid có cảnh báo:

- s01f30 #7: chiều cao 2,658 m vượt khoảng tham chiếu.
- s01f30 #8: có 27 điểm sát ngoài mép hộp.
- s05f46 #2: đáy thấp hơn mặt đường cục bộ 0,25 m.
- s05f46 #3: đáy thấp hơn mặt đường cục bộ 0,27 m.

Em chưa tiếp tục thay đổi các cuboid này chỉ để loại bỏ cảnh báo vì cần đối chiếu thêm với camera và point cloud. Cảnh báo tự động là gợi ý kiểm tra, không phải kết luận chắc chắn về lỗi gán nhãn.

**Kiểm chéo:** Em đã kiểm chéo với bạn có MSSV 2A202603119; kết quả các số được kiểm là khớp. Hiện em không nhớ cụ thể các thông số, giá trị và cuboid đã đối chiếu, nên chưa ghi lại được hai con số theo yêu cầu. Danh sách `h`, `dist_m` và kết luận “3/3 cuboid, 6/6 giá trị” trong bản nháp chưa được xác nhận là nội dung của lần kiểm chéo thực tế, nên không được giữ làm bằng chứng.

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc

Nhãn 3D sử dụng đơn vị mét thay vì pixel vì model cần học vị trí, kích thước và hướng thực tế của vật thể trong không gian. Vật thể ở xa hoặc thưa điểm cần quy tắc riêng để hạn chế việc model học từ các nhãn thiếu chính xác.

Sử dụng AI:

- Công cụ: ChatGPT.
- Câu hỏi chính: theo bản nháp, hướng dẫn sử dụng CVAT, kiểm tra số lượng cuboid, giải thích cảnh báo selfcheck và tổng hợp số liệu báo cáo; lần này là đọc repo Day 13 và sửa báo cáo theo phản hồi.
- Cách sử dụng: bản nháp ghi đã tham khảo AI để thao tác CVAT, đối chiếu ảnh và tính toán từ CSV. Phản hồi rà soát còn nêu việc gợi ý cuboid; nội dung và mức độ sử dụng trước đây chưa được em xác nhận đầy đủ. Trong lần biên tập này, ChatGPT đã đọc tài liệu repo, báo cáo và cả bảng A cùng phiếu tự kiểm ở lượt đọc đầu, sau đó chỉnh văn bản; không mở ảnh dữ liệu, không thao tác cuboid và không sửa hai CSV. Việc nhờ AI đọc bảng A, điền số, viết nhận xét hoặc gợi ý cuboid không phù hợp phạm vi dùng AI trong `RULES.md`. Em chưa có kết quả trao đổi với coach về các việc này; khai báo được giữ để coach xem xét, không coi việc sửa câu chữ là đã giải quyết vấn đề dùng AI.

Điều em còn chưa chắc là cách xác định kích thước đầy đủ của xe bị che khuất, hướng của vật thể ở xa và vị trí đáy trên mặt đường không bằng phẳng. Qua bài lab, em hiểu rõ hơn vai trò của việc kết hợp camera và LiDAR, kiểm tra nhiều góc nhìn và tự kiểm dữ liệu trước khi sử dụng nhãn để huấn luyện model.
