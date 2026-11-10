# Mini guideline — bài làm cá nhân

> Điền nội dung này trước khi bắt đầu gán nhãn. Giữ lại các quyết định sau khi self-QC.

## Định nghĩa lớp

| Lớp | Định nghĩa của bạn | Ví dụ/điểm phân biệt |
| --- | --- | --- |
| `car` | Phương tiện cá nhân cỡ nhỏ/vừa chở người hoặc chở hàng nhẹ dưới 9 chỗ ngồi. | Xe sedan, hatchback, SUV, CUV, taxi, xe bán tải (pickup cabin nhỏ như Ranger, Hilux). |
| `truck` | Xe cơ giới chuyên dụng chở hàng hóa tải trọng lớn, thùng xe tách biệt hoặc xe đầu kéo. | Xe tải thùng kín, xe tải thùng bạt, xe ben chở vật liệu xây dựng, xe đầu kéo container. Phân biệt với pickup (`car`) nhờ kích thước lớn và khung gầm tải chuyên dụng. |
| `bus` | Phương tiện giao thông công cộng hoặc dịch vụ vận tải hành khách số lượng lớn. | Xe buýt nội đô (City bus), xe buýt 2 tầng, xe khách liên tỉnh, minibus chở khách cỡ lớn. Khác với `van` ở chiều dài thân xe và mục đích chở khách công cộng. |
| `van` | Xe thân hộp liền khối (monovolume), từ 7 đến 16 chỗ dùng chở hàng nhẹ hoặc hành khách gia đình/dịch vụ. | Xe minivan, Ford Transit, Toyota HiAce, Hyundai Solati, xe cóc Suzuki Carry. Không nhầm với SUV thông thường (`car`) hoặc xe khách cỡ lớn (`bus`). |

## Quy tắc chung

- **Box bám sát vật thể nghĩa là:** 4 cạnh của Bounding Box phải tiếp xúc chính xác với các mép ngoài cùng nhìn thấy được của thân xe (nóc, lốp tiếp đất, mũi trước, đuôi sau). Không để khoảng trống thừa mặt đường/bầu trời và không cắt lẹm vào thân xe.
- **Vật thể bị che khuất (Occlusion):** Chỉ vẽ box ôm trọn phần thân xe mắt người thực tế nhìn thấy được (visible part) nếu phần nhìn thấy đủ để nhận diện loại xe. Tuyệt đối không tự suy đoán hoặc kéo box mở rộng sang phần bị che khuất.
- **Vật thể bị cắt bởi rìa ảnh (Truncation):** Box kéo chạm sát mép biên ảnh $[0, 1]$, không tự vẽ tràn ra ngoài hoặc đoán phần thân xe nằm ngoài khung hình.
- **Vật thể rất nhỏ/mơ hồ:** Nếu xe ở quá xa (chiều rộng chỉ vài pixel) không đủ chi tiết để phân biệt chắc chắn loại xe, hoặc xe nằm sâu trong bãi đỗ phụ sau rào chắn ngoài luồng giao thông chính: không tự đoán mò để tránh gây nhiễu cho mô hình.

## Ghi lại quyết định cho ít nhất ba ca mơ hồ

| Ảnh | Tình huống | Quyết định cuối | Lý do theo schema |
| --- | --- | --- | --- |
| `drive_001` | Chiếc xe sedan màu trắng ở giữa làn đường bị cột đèn giao thông che khuất cắt đôi thân xe. | Vẽ 1 box ôm trọn toàn bộ thân xe màu trắng theo biên ngoài nhìn thấy. | Xe chỉ bị cột đèn mỏng che một phần thân giữa, cấu trúc đầu và đuôi xe vẫn thể hiện rõ là một chiếc `car`. |
| `drive_008` | Xe ben chở đất đá màu đỏ (`truck`) và xe buýt khớp nối 2 toa màu vàng-xanh (`bus`) đi song song gần nhau. | Tách làm 2 box riêng biệt bám sát từng xe, gán đúng nhãn `truck` cho xe ben và `bus` cho xe buýt. | Đây là hai loại phương tiện lớn khác biệt rõ về hình học và công năng: xe ben có thùng tải hở phía sau, xe buýt có cửa kính đón khách dọc thân xe. |
| `drive_012` | Chiếc xe van màu bạc và xe con màu đen lưu thông sát nhau ở ngã tư, xe van bị che mất cản trước bởi dòng xe. | Gán nhãn `van` cho xe màu bạc và chỉ vẽ box quanh phần thân xe nhìn thấy được từ kính lái trở về sau. | Đuôi xe dạng hộp vuông vức đặc trưng của minivan/van thương mại, dù bị che cản trước vẫn nhận diện chắc chắn là `van`. Tuân thủ quy tắc vẽ quanh phần visible part. |
