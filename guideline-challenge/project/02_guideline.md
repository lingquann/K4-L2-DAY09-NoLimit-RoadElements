# 02. Sổ Quy Tắc Gắn Nhãn: Đèn Tín Hiệu Chủ Đạo Điều Khiển Làn Xe Mình
Version: v1

## 1. Mục tiêu bài toán
Xác định và khoanh khung các đầu đèn tín hiệu giao thông đường bộ, phân loại rõ đầu đèn nào là đèn chủ đạo đang điều khiển trực tiếp hướng di chuyển của làn xe hiện tại (Ego-Vehicle Primary Traffic Light), phục vụ hệ thống xe tự hành ra quyết định dừng hay đi.

## 2. Đối tượng cần vẽ và không được vẽ
- **PHẢI VẼ:**
  - Mọi hộp đèn tín hiệu giao thông đường bộ dành cho xe cơ giới (đèn 3 màu tròn, đèn mũi tên) quay mặt trực diện hoặc hơi chếch về phía xe mình.
- **KHÔNG ĐƯỢC VẼ:**
  - Đèn tín hiệu dành riêng cho người đi bộ (có biểu tượng hình người) hoặc đèn dành riêng cho xe đạp.
  - Hộp đèn quay lưng lại hoàn toàn (mặt sau của đèn chiều giao cắt).
  - Bóng đèn đường chiếu sáng, đốm sáng phản quang hoặc bóng đèn led trang trí đô thị.
  - Hộp đèn ở quá xa hoặc quá mờ với chiều cao nhỏ hơn 10 pixels.

## 3. Kỹ thuật dựng hình (Geometry)
- **Công cụ:** Sử dụng duy nhất Bounding Box (khung hình chữ nhật - Rectangle).
- **Ranh giới:** Khung phải vẽ ôm sát mép ngoài của **vỏ hộp đèn** (housing).
- **Tuyệt đối không khoanh:** Cột đỡ kim loại, cần vươn ngang, dây cáp treo hoặc bảng biển báo phụ gắn bên cạnh.
- **Chóa chắn nắng (Visor/Hood):** Chỉ lấy sát mép viền chóa nhô ra của hộp đèn, không kéo rộng khung sang vùng trời hay nền xung quanh.
- **Cụm nhiều hộp đèn:** Nếu trên cùng một cột có 2 hay nhiều hộp đèn tách rời (ví dụ: 1 hộp đèn đi thẳng, 1 hộp đèn mũi tên rẽ riêng), phải vẽ từng Bounding Box riêng biệt cho từng hộp đèn. Không vẽ gộp chung 1 khung to.

## 4. Danh mục nhãn và thuộc tính (Label & Attributes)
Tất cả đối tượng đều dùng chung nhãn: `traffic_light`.

### 4.1. Thuộc tính `relevance` (Bắt buộc chọn)
- `ego_primary`: Đèn treo trực tiếp trên làn xe đang đi hoặc ở cột bên phải ngay phía trước giao lộ, có hiệu lực điều khiển trực tiếp hướng đi của xe mình.
- `other_direction`: Đèn dành cho làn rẽ nhánh khác (khi xe mình ở làn đi thẳng), đèn của chiều xe ngược lại, hoặc đèn của tuyến đường cắt ngang ngã tư.
- `ambiguous`: Trường hợp giao lộ quá phức tạp, góc chụp lệch khó xác định làn đường, hoặc vị trí đèn nằm lơ lửng không thể khẳng định chắc chắn áp dụng cho làn nào.

### 4.2. Thuộc tính `state` (Bắt buộc chọn)
- `red`: Đèn đang sáng tín hiệu đỏ (đèn tròn đỏ hoặc mũi tên đỏ).
- `yellow`: Đèn đang sáng tín hiệu vàng (hoặc đèn vàng nhấp nháy).
- `green`: Đèn đang sáng tín hiệu xanh (đèn tròn xanh hoặc mũi tên xanh).
- `off`: Hộp đèn đang tắt hoàn toàn, không có mắt đèn nào phát sáng.
- `unknown`: Ánh sáng mặt trời chiếu chói lóa (sun glare), ngược sáng, hoặc ban đêm quá tối/mờ không thể nhận biết chính xác màu nào đang sáng.

### 4.3. Thuộc tính `occluded` (Bị che khuất)
- `false` (Mặc định): Nhìn thấy trọn vẹn toàn bộ vỏ hộp đèn.
- `true`: Hộp đèn bị che khuất một phần bởi cành cây, biển báo, dây điện hoặc thân xe khác.

## 5. Quy tắc xử lý bị che khuất (Occlusion)
- Nếu hộp đèn bị che khuất một phần nhưng vẫn quan sát được ít nhất một mắt đèn đang phát sáng: Vẫn vẽ Bounding Box ước lượng bao trọn toàn bộ kích thước của hộp đèn đó và đánh dấu tích `occluded = true`.
- Nếu hộp đèn bị vật cản che khuất trên 80% diện tích và không thể xác định được trạng thái sáng: Bỏ qua hoàn toàn, không vẽ.

## 6. Quy tắc kích thước và khoảng cách (Scale & Distance)
- **Ngưỡng tối thiểu:** Chiều cao của hộp đèn phải đạt tối thiểu từ **10 pixels** trở lên trên ảnh gốc.
- Đối với các đầu đèn nằm ở cuối ngã tư, quá xa chỉ thấy một đốm sáng nhòe không phân biệt được hình dáng vỏ hộp đèn: Bỏ qua hoàn toàn, không vẽ.

## 7. Quy tắc khi không chắc chắn (Uncertainty Handling)
- Tuyệt đối không đoán mò theo cảm tính.
- Nếu chắc chắn là đèn giao thông nhưng không rõ phân làn: Chọn `relevance = ambiguous`.
- Nếu chắc chắn là đèn nhưng không phân biệt được màu sắc đang bật: Chọn `state = unknown`.
- Nếu nhìn một đốm sáng mà không chắc đó là đèn giao thông hay biển phản quang / đèn trang trí: Bỏ qua hoàn toàn, không vẽ.

## 8. Quy tắc liên kết / Ngữ cảnh thời gian
- Bài toán này thực hiện trên tập ảnh tĩnh độc lập.
- Không áp dụng quy tắc theo dõi đối tượng theo thời gian (Tracking ID / Interpolation).

## 9. Tiêu chuẩn đánh giá chất lượng (Quality Standards)
- Khung vẽ phải ôm khít tối đa vỏ hộp đèn (sai lệch tọa độ không quá 2 pixels ở các mép viền).
- Không được bỏ sót đèn đỏ điều khiển làn xe mình (`relevance = ego_primary`, `state = red`) — đây là lỗi nghiêm trọng (Critical).
- Không được nhầm lẫn giữa đèn xe cơ giới và đèn tín hiệu người đi bộ.

## 10. Checklist kiểm tra nhanh trước khi nộp bài
1. Đã kiểm tra không còn bounding box nào để giá trị thuộc tính là `__undefined__` chưa?
2. Có khoanh nhầm cột đèn hay thanh đỡ kim loại không?
3. Đã loại trừ hết các đèn dành cho người đi bộ ở góc vỉa hè chưa?
4. Đèn đỏ trực tiếp của xe mình đã được gán đúng `ego_primary` chưa?
5. Các hộp đèn bị cành cây/biển báo che khuất đã được tick `occluded = true` chưa?