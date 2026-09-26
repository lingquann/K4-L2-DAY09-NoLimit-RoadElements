# Problem statement + downstream contract

## Bài toán

Gắn nhãn Bounding Box các hộp đèn tín hiệu giao thông đường bộ và phân loại đèn chủ đạo điều khiển hướng di chuyển của xe mình (Ego-Vehicle Primary Traffic Light) trong chuỗi ảnh hành trình giao lộ từ tập dữ liệu LISA, nơi xuất hiện cùng lúc nhiều đầu đèn phân làn (mũi tên rẽ trái, đèn tròn đi thẳng, đèn nhánh rẽ phải) trên cùng giá treo vươn ngang.

## Downstream contract

1. **Downstream task / model / user là ai?** 
   Module Nhận thức và Ra quyết định (Perception & Motion Planning) của hệ thống xe tự hành / ADAS. Mô hình cần phân định rành mạch giữa đèn điều khiển làn đi thẳng của xe mình với các hộp đèn điều khiển làn rẽ nhánh (ví dụ làn rẽ trái có luồng đèn riêng) để quyết định hành vi dừng trước vạch dừng hay được phép rẽ/đi tiếp.

2. **Output annotation nào thực sự cần?**
   - **Geometry:** Bounding Box (Rectangle) bao quanh từng vỏ hộp đèn (housing).
   - **Class:** `traffic_light`.
   - **Attributes:**
     - `relevance`: Phân loại vai trò điều khiển (`ego_primary`, `other_direction`, `ambiguous`).
     - `state`: Trạng thái và hình dạng tín hiệu hiển thị (`circle_red`, `circle_yellow`, `circle_green`, `arrow_straight`, `arrow_left`, `arrow_right`, `arrow_u_turn`, `off`, `unknown`).
     - `truncated`: Đối tượng bị cắt bởi mép khung hình (`true` / `false`).
     - `occluded`: Tận dụng cờ mặc định của CVAT (phím tắt Q) khi đèn bị che khuất bởi cành cây hoặc cột đèn.

3. **Failure nào gây hậu quả lớn nhất?**
   - **False Negative trên `ego_primary` + `circle_red`:** Nhìn thấy đèn tròn đỏ ngay trước làn xe chạy nhưng không vẽ box, hoặc gán nhầm thành `other_direction` (tưởng nhầm là đèn của làn khác), dẫn tới hệ thống không phanh và xe vượt đèn đỏ đâm vào luồng giao thông cắt ngang (mức độ: `critical`).
   - **Nhầm lẫn giữa đèn rẽ nhánh và đèn đi thẳng:** Nhầm đèn mũi tên rẽ trái `arrow_left` thành đèn điều khiển xe đi thẳng `ego_primary`, khiến xe đi thẳng dù luồng đi thẳng chưa được phép (mức độ: `critical`).

4. **Khi ambiguity không resolve được, ai / ở đâu là escalation path?**
   - Nếu có nhiều đầu đèn nằm lơ lửng giữa các làn hoặc xe đang ở vị trí chuyển làn chưa xác định rõ làn chính: Bắt buộc chọn `relevance = ambiguous`.
   - Nếu đèn ở quá xa hoặc ngược sáng trời chiều khiến không phân biệt rõ là đèn tròn hay mũi tên: Chọn `state = unknown`.
   - Hộp đèn ở quá xa hậu cảnh với chiều cao nhỏ hơn 10 pixels hoặc không nhìn rõ viền vỏ hộp: Bỏ qua hoàn toàn, không vẽ.

## Scope

- **Trong scope (bắt buộc label):**
  - Toàn bộ các hộp đèn tín hiệu giao thông cơ giới treo trên giá vươn ngang (overhead mast arm) hoặc gắn trên cột ngã tư quay mặt về phía xe.
  - Phân tách từng hộp đèn riêng lẻ: Hộp đèn mũi tên rẽ và hộp đèn tròn đi thẳng cạnh nhau phải vẽ thành 2 Bounding Box tách biệt.
  - Hộp đèn bị che một phần bởi cành cây/lá cọ nhưng vẫn nhận ra tín hiệu (vẽ hộp bao ngoài và bật `occluded = true`).
  - Hộp đèn sát mép viền ảnh bị cắt (bật `truncated = true`).

- **Ngoài scope (ignore):**
  - Biển báo giao thông treo cạnh đèn (như biển cấm rẽ, biển mũi tên rẽ gắn kèm trên giá treo).
  - Cần kim loại vươn ngang, cột đỡ bằng thép, dây cáp chịu lực.
  - Đèn tín hiệu dành riêng cho người đi bộ (biểu tượng hình người sang đường).
  - Đèn hậu hoặc đèn pha của các phương tiện đang lưu thông phía đối diện.
  - Các đầu đèn phụ ở khoảng cách quá xa (chiều cao hộp đèn < 10 pixels trên ảnh gốc).

- **Geometry tolerance:**
  - Box ôm khít phần viền vỏ hộp đèn (housing) kim loại, không khoanh trùm ra khoảng trời trống xung quanh. Sai lệch mép không quá 2 pixels.

## Output chấm được

Mọi quyết định đều kiểm tra được qua file xuất `CVAT for images 1.1`:
- **LABEL:** Tồn tại thẻ `<box label="traffic_light">`.
- **IGNORE:** Không xuất hiện box trên các biển báo phụ gắn trên cần ngang, không khoanh đèn xe hơi ngược chiều.
- **DECISION RELEVANCE:** Phân biệt chính xác giữa `<attribute name="relevance">ego_primary</attribute>` (đèn tròn đỏ đi thẳng) và `<attribute name="relevance">other_direction</attribute>` (đèn mũi tên rẽ trái).
- **DECISION STATE:** Ghi nhận đúng giá trị `arrow_left`, `circle_red` thay vì để giá trị rỗng `__undefined__`.
- **CRITICAL DECISION:** Bắt buộc có box cho đèn đỏ điều khiển làn xe chạy với thuộc tính `relevance="ego_primary"` và `state="circle_red"`.

## Dữ liệu và giới hạn

- **Nguồn ảnh:** Tập dữ liệu clip giao lộ `data/lisa` (chuỗi ảnh hành trình xe tiến vào ngã tư).
- **Số ảnh dự kiến dùng:** Sử dụng các frame trích từ sequence LISA (khoảng 3–4 frame làm `example`, 6 frame làm `calibration`, 5 frame làm `blind` test).
- **Giới hạn đã biết:** 
  - Ảnh chụp lúc trời chiều/ngược sáng làm hậu cảnh bầu trời bị cháy sáng (overexposed), trong khi phần tán cây và vỏ hộp đèn bị tối màu (underexposed), dễ gây khó khăn khi phân định mép viền hình học của vỏ đèn.
  - Các frame là chuỗi thời gian liên tiếp của một xe đang di chuyển.