# Peer feedback + owner response

Phần 1 do **nhóm peer** trả lời (gửi kèm file export). Phần 2 do **nhóm owner** điền.

- **Nhóm peer:** Traffic
- **Người label blind:** Trần Minh Nhật (Annotator đại diện nhóm Traffic)

## 1. Peer trả lời

1. **Rule nào rõ nhất / giúp quyết định nhanh nhất?** Quy tắc phân định `ego_primary` (Mục 4.1): Chọn đầu đèn chính vươn ngang ngã tư thẳng hướng di chuyển của xe.
2. **Rule nào mơ hồ hoặc phải tự suy diễn?** Phân định `relevance` cho mũi tên rẽ trái màu đỏ bên cạnh đèn đi thẳng màu xanh trong ảnh LISA24.
3. **Sample nào khiến guideline "vỡ"?** LISA30: Đèn nhỏ ở khoảng cách xa bị che một phần và lóa ánh mặt trời, annotator đắn đo giữa đoán màu xanh hay chọn `ambiguous` / `unknown`.
4. **Attribute / default nào trong CVAT dễ gây thao tác sai?** Attribute `occluded` điều khiển bằng phím tắt `Q` (không có trong menu dropdown chọn thuộc tính custom) dễ bị bỏ sót.
5. **Một thay đổi cụ thể giúp annotator mới ít hỏi hơn?** Thêm hình minh họa thực tế cho trường hợp LISA24 trực tiếp vào Mục 9 của sổ quy tắc guideline v3.

## 2. Owner phân loại

Owner không tranh luận để bảo vệ guideline. Mỗi feedback và mỗi decision peer làm sai được xếp vào một hướng xử lý.

| Feedback / decision sai | Nguyên nhân (guideline gap / data ambiguity / execution error) | Xử lý (accept + revise / reject with evidence / add escalation rule) | Bằng chứng |
|---|---|---|---|
| LISA24 D05: Gán ego_primary cho mũi tên rẽ trái màu đỏ | guideline_gap | accept + revise (bổ sung hình minh họa LISA24 vào Mục 9 và nhấn mạnh quy tắc ở Mục 4.1 trong guideline v3) | Clarification log dòng 1, `transfer_score.csv` dòng LISA24 D05 |
| LISA30 D09: Đoán state = circle_green thay vì chọn unknown khi đèn bị che lóa | execution_error | add_escalation (bổ sung quy tắc bắt buộc chọn ambiguous + unknown khi không đủ điểm ảnh nhận diện màu) | `transfer_score.csv` dòng LISA30 D09, `03_ontology_and_cvat_setup.md` |
