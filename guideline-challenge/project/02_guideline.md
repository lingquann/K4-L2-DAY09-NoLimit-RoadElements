# 02. Sổ Quy Tắc Gắn Nhãn: Đèn Tín Hiệu Chủ Đạo Điều Khiển Làn Xe Mình
Version: v2

## 1. Mục tiêu bài toán
Xác định và khoanh khung các đầu đèn tín hiệu giao thông đường bộ, phân loại rõ đầu đèn nào là đèn chủ đạo đang điều khiển trực tiếp hướng di chuyển của làn xe hiện tại (Ego-Vehicle Primary Traffic Light), phục vụ hệ thống xe tự hành ra quyết định dừng hay đi.

## 2. Chế độ vẽ và phạm vi đối tượng

- **Chế độ vẽ trên CVAT:** Bắt buộc chọn chế độ **Shape** (vẽ độc lập trên từng frame ảnh tĩnh). **TUYỆT ĐỐI KHÔNG** dùng chế độ **Track**.

### PHẢI VẼ:
- Mọi vỏ hộp đèn tín hiệu giao thông đường bộ dành cho phương tiện cơ giới (đèn bóng tròn, đèn mũi tên) thỏa mãn đồng thời 3 điều kiện:
  1. **Góc quay (Yaw angle $\le$ 45°):** Mặt trước của hộp đèn quay trực diện hoặc quay nghiêng một góc không quá 45° so với phương nhìn thẳng của camera xe mình.
  2. **Dấu hiệu nhận biết mặt trước:** Mắt thường phải nhìn thấy được mặt kính phẳng phía trước và phân biệt rõ ít nhất một thấu kính/vòng tròn đèn hoặc hình dáng mũi tên (dù đang phát sáng hay đang tắt).
  3. **Ngưỡng kích thước tối thiểu:** Chiều cao của hộp đèn (đo từ mép trên cùng của vỏ/chao đèn đến đáy hộp) phải đạt **từ 10 pixels trở lên** trên ảnh gốc. Đối với đèn bị che khuất một phần, phần vỏ nhìn thấy thực tế vẫn phải đạt chiều cao tối thiểu $\ge$ 10 pixels.

### KHÔNG ĐƯỢC VẼ:
- **Đèn quay nghiêng góc lớn hoặc quay lưng:** Đèn của tuyến đường giao cắt quay ngang ($\approx$ 90°) hoặc quay lưng lại với xe mình, nơi chỉ nhìn thấy cạnh sườn kim loại màu đen hoặc lưng chao chắn nắng mà không thấy mặt kính phát sáng.
- **Đèn tín hiệu chuyên biệt khác:** Đèn tín hiệu dành riêng cho người đi bộ (có biểu tượng hình người sang đường), đèn dành cho xe đạp.
- **Nguồn sáng giao thông không phải đèn tín hiệu ngã tư:** Đèn hậu, đèn phanh màu đỏ của các phương tiện lưu thông phía trước; đèn đường chiếu sáng đô thị; đốm phản quang trên cọc tiêu hoặc đèn led trang trí viền đường.
- **Bộ phận kết cấu phụ:** Không khoanh cột đỡ kim loại thẳng đứng, thanh xà ngang vươn ra lòng đường, dây cáp treo hoặc bảng biển báo phụ gắn cạnh hộp đèn (như biển mũi tên rẽ, biển cấm quay đầu).
- **Đèn quá xa hoặc quá mờ:** Hộp đèn ở hậu cảnh có chiều cao nhỏ hơn 10 pixels hoặc chỉ là đốm sáng nhòe không xác định được cấu trúc vỏ hộp chữ nhật.

## 3. Kỹ thuật dựng hình (Geometry)
- **Công cụ:** Sử dụng duy nhất Bounding Box (khung hình chữ nhật - Rectangle).
- **Ranh giới hộp đèn (Visible Boundary):**
  - Khung chữ nhật chỉ vẽ bao quanh **phần vỏ hộp đèn thực tế nhìn thấy được bằng mắt thường**.
  - **Mũ che nắng / Chao chắn nắng (Visor):** Là phần mi sắt/nhựa nhô ra phía trên mỗi mắt đèn để che ánh mặt trời. Chỉ ôm sát phần mũ này, **không** kéo khung tràn rộng ra nền trời trống xung quanh.
  - **Tuyệt đối không khoanh:** Cột kim loại thẳng đứng, thanh xà ngang vươn ra đường, dây cáp treo hoặc bảng biển báo phụ gắn bên cạnh.
- **Cụm nhiều hộp đèn:** Trên cùng một giá treo có 2 hoặc nhiều hộp đèn tách rời (ví dụ: hộp đèn rẽ trái riêng, hộp đèn đi thẳng riêng), **phải vẽ từng Bounding Box riêng biệt cho từng hộp đèn**. Không vẽ gộp chung 1 khung to.

## 4. Danh mục nhãn và thuộc tính (Label & Attributes)
Tất cả đối tượng đều dùng chung nhãn: `traffic_light`.

### 4.1. Thuộc tính `relevance` (Bắt buộc chọn)
- `ego_primary`: Đèn điều khiển trực tiếp hướng đi của xe mình. Căn cứ vào vị trí làn xe đang chạy:
  - Nếu xe đang ở làn đi thẳng: Đèn tròn hoặc mũi tên đi thẳng nằm ngay trước mặt/phía trên làn là `ego_primary`.
  - Quy tắc phân xử góc chụp lệch: Nếu trên giá treo có 3 đầu đèn (Trái - Đi thẳng - Phải) mà góc camera bị lệch làn hoặc đang tiến vào tâm giao lộ, chỉ chọn `ego_primary` cho đầu đèn thẳng hàng nhất với quỹ đạo di chuyển hiện tại của xe.
- `other_direction`: Đèn của làn rẽ nhánh độc lập (khi xe không đi vào làn đó), đèn của chiều ngược lại, hoặc đèn của đường cắt ngang.
- `ambiguous`: Trường hợp giao lộ quá phức tạp, góc chụp xiên khiến không thể phân định xe đang thuộc quyền điều khiển của đầu đèn nào, hoặc xe đang ở giữa hai vạch phân làn chưa rõ ý định rẽ.

### 4.2. Thuộc tính `state` (Tách riêng đèn tròn và đèn mũi tên)
- `circle_red`: Đèn tròn sáng đỏ.
- `circle_yellow`: Đèn tròn sáng vàng hoặc nhấp nháy vàng.
- `circle_green`: Đèn tròn sáng xanh.
- `arrow_straight`: Đèn hiển thị mũi tên chỉ hướng đi thẳng.
- `arrow_left`: Đèn hiển thị mũi tên rẽ trái.
- `arrow_right`: Đèn hiển thị mũi tên rẽ phải.
- `arrow_u_turn`: Đèn hiển thị mũi tên quay đầu xe.
- `off`: Hộp đèn tối hoàn toàn, không có mắt đèn nào phát sáng.
- `unknown`: Bị chói lóa nắng (sun glare), ngược sáng trắng, hoặc quá mờ không thể khẳng định chắc chắn hình dạng/màu sắc tín hiệu.
*(Lưu ý về Đỏ nhấp nháy - Flashing Red: Chọn `circle_red` vì hiệu lệnh thực tế của hệ thống xe tự hành vẫn là dừng lại quan sát an toàn trước khi di chuyển).*

### 4.3. Thuộc tính `truncated` và `occluded`
- `truncated`: Checkbox. Tick chọn `true` khi hộp đèn nằm sát mép khung hình và bị đường viền ảnh cắt cụt một phần.
- `occluded`: Tận dụng phím tắt **Q** của CVAT. Bấm `Q` để bật cờ `occluded = true` khi hộp đèn bị vật cản (cành cây, cột khác, biển báo) che mất một phần.

## 5. Quy tắc xử lý bị che khuất và trùng màu nền (Occlusion & Contrast)
- **Quy tắc khung nhìn thấy (Visible Crop):** Không tự ước lượng kích thước phần bị che khuất. **Chỉ vẽ Bounding Box vừa khít phần vỏ hộp đèn nhìn thấy được bằng mắt thường** và bấm phím **Q** (`occluded = true`).
- Nếu hộp đèn bị vật cản che khuất trên 80% diện tích hoặc không còn nhìn thấy tín hiệu đèn sáng: **Bỏ qua hoàn toàn, không vẽ**.
- **Màu thân đèn trùng màu bóng cây:** Khi vỏ hộp đèn màu đen/xám chìm vào tán cây tối màu phía sau, lấy ranh giới dựa trên vùng phát quang của bóng đèn cộng với khoảng cách ước lệ tối thiểu của chao đèn nhìn thấy được. Nếu không thể xác định được mép ngoài của vỏ đèn: Bỏ qua hoặc chỉ khoanh sát vùng chao đèn phát sáng.

## 6. Quy tắc kích thước và khoảng cách (Scale & Distance)
- **Ngưỡng tối thiểu:** Chiều cao của hộp đèn phải đạt tối thiểu từ **10 pixels** trở lên trên ảnh gốc.
- Đèn ở quá xa chỉ xuất hiện dưới dạng một đốm sáng nhòe (không nhận diện được hình khối vỏ hộp đèn): **Bỏ qua hoàn toàn, không vẽ**.

## 7. Quy tắc khi không chắc chắn (Uncertainty Handling)
- Tuyệt đối không suy đoán theo cảm tính.
- Nếu xác định được là đèn giao thông nhưng không rõ phân làn: Chọn `relevance = ambiguous`.
- Nếu chắc chắn là đèn nhưng bị lóa hoặc mất màu: Chọn `state = unknown`.
- Đốm đỏ ở tầm thấp ngang mặt đường nghi ngờ giữa đèn hậu xe ô tô và đèn giao thông: Nếu không có cấu trúc vỏ hộp chữ nhật hoặc cột treo -> Coi là đèn xe và **bỏ qua không vẽ**.

## 8. Quy tắc liên kết / Ngữ cảnh thời gian
- Bài toán này thực hiện trên tập ảnh tĩnh độc lập theo từng frame.
- Không áp dụng quy tắc theo dõi đối tượng theo thời gian (Tracking ID / Interpolation / Keyframe).

## 9. Ảnh mẫu minh họa quy tắc (Example References)
- **LISA01:** Minh họa cụm đèn trên cần vươn ngang ngã tư. Đèn mũi tên đỏ rẽ trái bên cạnh biển phụ là `other_direction` (`state = arrow_left`). Đèn tròn đỏ ở giữa làn đường đi thẳng là `ego_primary` (`state = circle_red`). Đèn tròn đỏ phía xa bên phải là `other_direction`.
- **LISA02:** Minh họa trường hợp đèn bị cành cây/tán cọ che khuất một phần. Chỉ khoanh phần vỏ nhìn thấy, bấm phím `Q` (`occluded = true`).
- **LISA03:** Minh họa góc chụp camera lệch làn. Phân định `ego_primary` cho đèn thẳng hướng bánh xe di chuyển.

## 10. Checklist kiểm tra nhanh trước khi nộp bài
1. Tất cả đối tượng đã được vẽ ở chế độ **Shape**, không có đối tượng nào ở chế độ **Track** chưa?
2. Đã kiểm tra không còn bounding box nào để giá trị thuộc tính là `__undefined__` chưa?
3. Có vẽ nhầm vào đèn hậu xe hơi hoặc biển báo treo cạnh đèn không?
4. Đèn rẽ trái độc lập đã được tách riêng box và gán đúng `arrow_left` + `other_direction` chưa?
5. Khung vẽ đã ôm sát mép nhìn thấy (Visible Crop) thay vì tự ước lượng kích thước phần bị che chưa?