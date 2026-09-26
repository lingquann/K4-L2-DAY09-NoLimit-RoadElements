# Ontology + CVAT setup

Bảng ontology là **source of truth** cho schema CVAT: `03_cvat_labels.json` phải khớp từng dòng ở đây.

## Ontology table

| Name | Geometry | Type (class / attribute) | Allowed values | Default | Mutable? | Rationale |
|---|---|---|---|---|---|---|
| `traffic_light` | rectangle | class | — | — | — | Một loại object vật lý duy nhất: vỏ hộp đèn tín hiệu cho xe cơ giới. Mọi khác biệt downstream cần (đèn nào, sáng gì) là thuộc tính của cùng object nên không tách class |
| `relevance` | — | attribute (select) của `traffic_light` | `__undefined__`, `ego_primary`, `other_direction`, `ambiguous` | `__undefined__` | false | Câu hỏi cốt lõi của bài toán: đèn nào điều khiển xe mình. `ambiguous` là đường escalation khi không đủ bằng chứng về làn |
| `state` | — | attribute (select) của `traffic_light` | `__undefined__`, `circle_green`, `circle_yellow`, `circle_red`, `arrow_straight`, `arrow_left`, `arrow_right`, `arrow_u_turn`, `off`, `unknown` | `__undefined__` | false | Downstream cần phân biệt đèn tròn và mũi tên vì hiệu lệnh khác nhau. `unknown` cho lóa/mất màu, `off` cho hộp đèn tối hẳn |
| `truncated` | — | attribute (checkbox) của `traffic_light` | `false` / `true` | `false` | false | Báo box bị mép ảnh cắt để downstream không coi kích thước box là kích thước thật của đèn |
| `occluded` | — | cờ có sẵn của CVAT (phím `Q`), không khai trong JSON | `false` / `true` | `false` | false | Báo box chỉ ôm phần nhìn thấy (visible crop), phần vỏ bị vật cản che |

`mutable = false` cho mọi attribute vì nhóm dùng Shape trên ảnh tĩnh, không có track qua nhiều frame.

## Class hay attribute

- **Class `traffic_light`**: tất cả đầu đèn có cùng geometry (rectangle ôm vỏ), cùng rule vẽ và cùng QA rule. Tách class theo hướng (`light_left`, `light_straight`…) sẽ nhân số class với số màu và làm annotator phải chọn 2 lần.
- **`relevance` là attribute**: cùng một đầu đèn có thể là `ego_primary` hay `other_direction` tùy làn xe mình đang đi — đó là quan hệ với ego, không phải loại object.
- **`state` là attribute**: là trạng thái hiển thị của cùng object, đổi theo thời gian (đỏ → xanh ở LISA16).
- **`truncated`, `occluded` là attribute**: chỉ mô tả chất lượng quan sát, không đổi ý nghĩa object.

**Default và bias:**
- `relevance` và `state` mặc định `__undefined__`: annotator quên chọn thì export còn `__undefined__` và bị bắt ở QA (checklist mục 2), thay vì tạo ra đèn “im lặng” sai nghĩa. Calibration v1 vẫn còn **32 box `__undefined__`** (29 box của 1 người, 3 box của 1 người) → rule này cần giữ và kiểm tự động.
- `truncated = false` và `occluded = false` có thể gây bias “quên tick”: đèn bị che/cắt vẫn được ghi là nguyên vẹn. Calibration v1: chỉ **1/5** người dùng cờ `occluded`. Hậu quả nhẹ hơn (ảnh hưởng geometry, không đổi quyết định dừng/đi) nên nhóm chấp nhận default `false` và bắt lỗi bằng review mẫu.

## CVAT

- **Phiên bản CVAT** (`make cvat-status`): CVAT 2.75.1 tại http://localhost:8080 (máy Thành Long, kiểm ngày 26/09)
- **Link CVAT tham chiếu**:
  - Task URL: http://localhost:8080/tasks/1
  - Job URL: http://localhost:8080/tasks/1/jobs/1
- **Tên task calibration**: vòng calibration v1 mỗi người tạo task tên `traffic_light` (xem thẻ `<name>` trong 5 file export). Từ vòng sau đặt theo mẫu `nolimit-calib-v2-tvaq` để export tự ghi version guideline.
- **Guide của task đã dán `02_guideline.md`?** **Có.** Đã dán toàn bộ nội dung `02_guideline.md` (phiên bản v2) vào tab **Guide** của Task CVAT để annotator tra cứu trực tiếp trong giao diện CVAT.
- **Nhóm dùng Track hay Shape, vì sao:** **Shape.** Bài toán là ảnh tĩnh, mỗi frame được quyết định độc lập; Track sẽ nội suy box và kéo `state` từ frame trước sang frame sau (sai ở frame đèn đổi màu LISA16). Kiểm chứng: cả 5 export calibration có **0 track**. Guideline v2 mục 2 ghi rõ “TUYỆT ĐỐI KHÔNG dùng Track”.

## Setup test

Một thành viên **chưa tham gia setup** mở task và trả lời: label gì, dùng tool nào, gán attribute nào, khi nào escalate. Ghi lại ai test và chỗ họ vấp:

- **Người test:** Nguyễn Thị Thùy Linh (`nguyenthithuylinh1285-hue` - QA owner, thành viên không trực tiếp khởi tạo schema task trên CVAT).
- **Kết quả 4 câu hỏi:**
  1. *Label gì?*: Sử dụng đúng 1 label duy nhất là `traffic_light` (kiểu rectangle).
  2. *Dùng tool nào?*: Sử dụng công cụ Draw new rectangle ở chế độ **Shape** (tuyệt đối không dùng Track).
  3. *Gán attribute nào?*: Bắt buộc chọn `relevance` (`ego_primary`, `other_direction`, `ambiguous`) và `state` (`circle_green`, `circle_yellow`, `circle_red`, `arrow_straight`, `arrow_left`, `arrow_right`, `arrow_u_turn`, `off`, `unknown`); tích chọn `truncated = true` khi bị viền ảnh cắt cụt và nhấn phím `Q` (`occluded = true`) khi bị vật cản che một phần.
  4. *Khi nào escalate?*: Chọn `relevance = ambiguous` khi gặp ngã tư phức tạp/mất vạch kẻ đường không thể khẳng định làn xe mình; chọn `state = unknown` khi đèn bị chói nắng/ngược sáng mờ không thể xác định màu tín hiệu.
- **Chỗ vấp:** Người test ban đầu loay hoay tìm thuộc tính `occluded` trong menu dropdown chọn thuộc tính custom (do `occluded` là cờ mặc định có sẵn của CVAT điều khiển bằng phím tắt `Q` chứ không phải custom attribute trong JSON schema). Đã bổ sung ghi chú phím tắt `Q` rõ ràng vào Sổ quy tắc `02_guideline.md` (mục 4.3) và bảng ontology để người dán nhãn dễ thao tác.

## Checklist CVAT Setup

- [x] Khai báo schema label `traffic_light` (rectangle) với đầy đủ thuộc tính `relevance`, `state`, `truncated` trong CVAT Raw Labels.
- [x] Đã kiểm tra phím tắt `Q` (cờ `occluded` mặc định của CVAT) hoạt động chính xác trên giao diện CVAT.
- [x] Đã dán nội dung Sổ quy tắc `02_guideline.md` (v2) vào tab Guide của Task.
- [x] Đã thực hiện Setup Test thành công với thành viên QA owner và hoàn thành ghi nhận kết quả.
