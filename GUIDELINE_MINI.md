# Mini guideline — bài làm cá nhân

> Điền nội dung này trước khi bắt đầu gán nhãn. Giữ lại các quyết định sau khi self-QC.

## Định nghĩa lớp

| Lớp | Định nghĩa của bạn | Ví dụ/điểm phân biệt |
| --- | --- | --- |
| `car` |Xe chở người cỡ nhỏ, thân thấp: sedan, hatchback, SUV, crossover, taxi, xe bán tải cabin nhỏ. |Nhìn từ trên cao thấy rõ 3 khối: nắp capo – mui/kính – cốp (hoặc đuôi ngắn). Mui xe thấp, chiều dài ≈ 1–1.3 lần kích thước các xe con xung quanh. SUV cao hơn sedan nhưng vẫn có capo nhô rõ → vẫn là `car`. Taxi/xe đỗ ven đường vẫn là `car`. |
| `truck` |Xe chở hàng có cabin tách biệt với thùng hàng: xe tải thùng kín, thùng hở, xe ben, xe bồn, xe đầu kéo (có/không kéo rơ-moóc). |Thấy khe/ranh giới rõ giữa cabin và thùng hàng; thùng thường vuông, cao, không có dãy cửa sổ hành khách. Phân biệt với `bus`: bus là một khối liền từ đầu tới đuôi và có nhiều cửa sổ dọc thân. Xe bán tải nhỏ có thùng hở → `car`, không phải `truck`.  |
| `bus` |Xe chở khách cỡ lớn/trung: xe buýt thành phố, xe khách, minibus (khoảng từ 16 chỗ trở lên). |Thân dài, liền một khối hình hộp, có dãy cửa sổ hành khách chạy dọc thân, mui phẳng và dài; thường dài gấp ≥ 2 lần xe con. Phân biệt với `van`: bus dài hơn rõ rệt và có cửa lên xuống ở giữa/đầu xe; van ngắn, gần cỡ xe con.  |
| `van` |Xe van/minivan hình hộp cỡ nhỏ, chở hàng hoặc chở người (≈ 7–16 chỗ). | Đầu xe ngắn (capo rất ngắn hoặc gần như không có), thân cao và vuông, mui cao hơn hẳn xe con bên cạnh; chiều dài chỉ nhỉnh hơn xe con. Phân biệt với `car`: SUV có capo dài và mui thấp hơn, còn van thì như "một khối hộp". Phân biệt với `truck`: van không có khe tách cabin và thùng. |

## Quy tắc chung

- Box bám sát vật thể nghĩa là: 4 cạnh box chạm đúng điểm ngoài cùng của phần thân xe nhìn thấy (kể cả gương chiếu hậu nếu nhìn rõ), không chừa khoảng trống, không lấy bóng đổ trên mặt đường, vùng đèn pha loá (ảnh ban đêm) hay xe bên cạnh. Sau khi vẽ, zoom vào kiểm tra lại cả 4 cạnh; lệch tối đa ~2–3 px.
- Vật thể bị che khuất: Nếu còn nhìn thấy đủ đặc điểm để nhận ra lớp (ví dụ thấy được mui + kính hoặc đầu/đuôi xe), vẽ box quanh phần nhìn thấy, không đoán phần bị che. Hai xe chồng lên nhau → hai box riêng, mỗi box chỉ ôm phần của xe đó. Xe bị cột điện/biển báo che một dải mảnh ở giữa → vẫn vẽ một box liền cho cả xe.
- Vật thể bị cắt bởi rìa ảnh: Vẫn gán nếu phần còn lại nhận ra được là xe thuộc schema; cạnh box trùng đúng mép ảnh (tọa độ 0 hoặc 1), không kéo box ra ngoài và không đoán kích thước thật.
- Vật thể rất nhỏ/mơ hồ: Xe ở xa vẫn gán nếu nhận ra là xe (tham khảo nhãn eval: có box chỉ ~13×23 px). Chỉ bỏ qua khi quá nhỏ hoặc quá mờ đến mức không chắc đó là xe. Nếu chắc là xe nhưng không chắc lớp (van/bus, van/car) → ghi vào bảng ca mơ hồ và hỏi giảng viên, không tự đoán.
- Ngoài phạm vi: Không gán xe máy, xe đạp, người đi bộ, xe ba gác, xe trên biển quảng cáo, xe phản chiếu trên kính.

## Ghi lại quyết định cho ít nhất ba ca mơ hồ

| Ảnh | Tình huống | Quyết định cuối | Lý do theo schema |
| --- | --- | --- | --- |
| `drive_001` (xe trắng nhỏ đỗ bên trái vỉa hè) | Thân hình hộp, dễ nhầm với `van` | `car` | Kích thước nhỏ hơn sedan bên cạnh, đây là xe con cỡ mini; `van` phải có thân dài và mui cao hơn xe con |
| `drive_019` (xe trắng hình hộp đỗ cạnh sedan xám) | Xe cỡ nhỏ, dễ nhầm với `car` như ca `drive_001` | `van` | Dài gần bằng sedan bên cạnh nhưng mui cao, phẳng kéo dài tới đuôi, capo rất ngắn, cửa hông lớn → minivan theo schema; khác `drive_001` vì xe mini ở đó ngắn hơn sedan |
| `drive_028` (xe trắng thân dài đỗ sau SUV trắng, mép trái ảnh) | Bị SUV phía trước và ô che màu cam che phần đầu xe; không rõ là xe tải nhỏ hay van | `van` | Phần nhìn thấy là một khối thân liền, mui phẳng kéo dài, không thấy khe tách cabin – thùng hàng nên không đủ căn cứ gán `truck`; kích thước dưới 2 lần xe con nên không phải `bus` → theo schema là van chở hàng |
