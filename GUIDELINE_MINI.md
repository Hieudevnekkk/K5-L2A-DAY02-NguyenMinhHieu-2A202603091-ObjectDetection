# Mini guideline — bài làm cá nhân

> Điền nội dung này trước khi bắt đầu gán nhãn. Giữ lại các quyết định sau khi self-QC.

## Định nghĩa lớp

| Lớp | Định nghĩa của bạn | Ví dụ/điểm phân biệt |
| --- | --- | --- |
| `car` | Xe con chở người dưới 9 chỗ: sedan, hatchback, SUV, taxi, xe địa hình nhỏ. Thân thấp, cabin liền thân xe. | Taxi vàng-xanh ở `drive_005`, xe SUV đen ở `drive_005`. SUV vẫn là `car` dù thân cao. |
| `truck` | Xe tải/bán tải có cabin riêng và thùng chở hàng tách khỏi cabin (thùng hở hoặc thùng kín). Dùng chở hàng. | Xe tải nhỏ màu trắng giữa ảnh `drive_005`. Phân biệt với `van` ở chỗ thùng hàng tách khỏi cabin. |
| `bus` | Xe buýt/xe khách cỡ lớn chở nhiều người: thân dài, nhiều cửa sổ và cửa lên xuống, kể cả xe buýt ghép nối (articulated). | Các xe buýt ở `drive_029`. Xe nhỏ chở 9-16 chỗ không tính là `bus`. |
| `van` | Xe thùng/xe chở khách cỡ nhỏ hoặc vừa, thân liền khối cao, cabin và khoang sau liền nhau, không có thùng hàng tách riêng. Gồm minivan và xe van chở hàng. | Xe van trắng ở góc trái `drive_005`; xe van xanh đậm phía xa ở `drive_018`. |

## Quy tắc chung

- Box bám sát vật thể nghĩa là: cạnh box chạm sát phần thân xe nhìn thấy được (kể cả gương, cản), không lấy thêm mặt đường, bóng xe hay xe bên cạnh. Mỗi xe một box riêng, xe buýt ghép nối vẫn là một xe nên một box.
- Vật thể bị che khuất: vẫn gán box quanh phần nhìn thấy được nếu nhận ra được loại xe; hai xe chồng nhau thì vẽ hai box riêng cho từng phần nhìn thấy.
- Vật thể bị cắt bởi rìa ảnh: box chạm đúng rìa ảnh, không đoán phần nằm ngoài ảnh. Ví dụ xe buýt đỏ ở góc dưới bên phải `drive_029`.
- Vật thể rất nhỏ/mơ hồ: nếu nhìn thấy rõ là xe thuộc bốn lớp thì vẫn gán, kể cả xe nhỏ ở xa. Nếu chỉ còn mảnh quá nhỏ, không xác định chắc được lớp thì không tự đoán, ghi vào bảng bên dưới và hỏi giảng viên.
- Không gán: người đi bộ, xe đạp, xe máy/mô tô (kể cả xe có người ngồi), biển báo, đèn giao thông, xe trong ảnh quảng cáo hoặc phản chiếu trong kính.

## Ghi lại quyết định cho ít nhất ba ca mơ hồ

| Ảnh | Tình huống | Quyết định cuối | Lý do theo schema |
| --- | --- | --- | --- |
| `drive_039` | Xe buýt ghép nối (hai khoang nối bằng khớp xoay, vàng-trắng ở bên trái) có thể bị nhầm là hai xe. | Một box `bus` bao toàn bộ xe, gồm cả hai khoang. | Schema quy định mỗi xe hợp lệ là một box. Xe ghép nối là một phương tiện duy nhất. |
| `drive_005` | Xe SUV màu đen thân cao (giữa ảnh) trông giống xe van. | `car`. | Có mũi xe và khoang hành khách liền khối thấp hơn xe van; SUV là xe con chở người nên thuộc `car`. |
| `drive_005` | Xe màu trắng ở giữa ảnh có cabin và thùng phía sau, dễ nhầm giữa `truck` và `van`. | `truck`, vì thùng hàng tách khỏi cabin. Nếu khi xem lại thấy thân liền khối thì chuyển sang `van` và ghi lại ở đây. | Tiêu chí phân biệt đã ghi ở bảng định nghĩa lớp. |
