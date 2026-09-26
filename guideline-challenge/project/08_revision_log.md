# Revision log

Guideline v1 = bản nháp đầu; v2 = sau calibration nội bộ; v3 = sau blind handoff. Mỗi lần tăng `Version` trong
`02_guideline.md`, thêm một hoặc nhiều dòng vào bảng: đổi gì và vì sao, kèm bằng chứng (sample_id, dòng
calibration report, câu hỏi trong clarification log, feedback của peer).

| Version | Đổi gì | Vì sao | Bằng chứng |
|---|---|---|---|
| v1 | Bản nháp đầu: 1 label `traffic_light` (rectangle); `relevance` = ego_primary / other_direction / ambiguous; `state` = red / yellow / green / off / unknown; cờ `occluded`; ngưỡng 10 px; không dùng tracking | Chốt bài toán hẹp: đèn nào điều khiển xe mình tại giao lộ nhiều đầu đèn (LISA dayClip5) | Commit `f4ec385`; 5 export calibration trong `06_calibration_exports/` label theo bản này |
| v2 | Mục 4.1: thêm quy tắc phân xử — chỉ đầu đèn **thẳng hàng nhất** với quỹ đạo xe là `ego_primary`; đèn làn rẽ nhánh độc lập là `other_direction` | Calibration: 3/5 người gán mũi tên đỏ rẽ trái và đầu đèn phải là `ego_primary` → 3 box ego/frame, 43 lượt frame có ego vừa đỏ vừa xanh | `06_calibration_report.csv` dòng LISA10 (mũi tên trái) và LISA18 (đầu phải, relevance) |
| v2 | Mục 4.2: tách `state` thành đèn tròn (`circle_red/yellow/green`) và mũi tên (`arrow_left/right/straight/u_turn`); đỏ nhấp nháy → `circle_red` | v1 chỉ có màu nên không phân biệt được mũi tên rẽ với đèn tròn đi thẳng — đúng chỗ gây nhầm relevance | LISA01, LISA17: mũi tên trái đỏ cạnh đèn đi thẳng; `03_cvat_labels.json` cập nhật cùng lúc |
| v2 | Mục 2: thêm 3 điều kiện bắt buộc vẽ (mặt đèn quay ≤ 45°, thấy mặt kính, cao ≥ 10 px kể cả phần nhìn thấy khi bị che); bắt buộc chế độ Shape | Số box mỗi frame lệch 5 · 5 · 6 · 6 · 7 giữa 5 người, chủ yếu ở cụm đèn xa x≈810–845; `make calib` (6 frame calibration): đồng thuận count 0% | `06_calibration_report.csv` dòng LISA09 (count); `06_calibration_measure.csv` |
| v2 | Mục 5: đổi từ box ước lượng bao trọn (amodal) sang **visible crop** + phím `Q`; thêm rule vỏ đèn chìm vào tán cây | v1 mục 5 (amodal) mâu thuẫn mục 9 (ôm khít ±2 px); chỉ 1/5 người bật `occluded` | `06_calibration_report.csv` dòng LISA10 (occluded) |
| v2 | Mục 7: đốm đỏ tầm thấp không có vỏ hộp/cột treo = đèn xe, không vẽ; không rõ màu → `unknown`, không rõ làn → `ambiguous` | Đầu đèn nhỏ bị che ở cụm xa: 2 người không vẽ, 2 người `unknown`, 1 người `red` | `06_calibration_report.csv` dòng LISA17 (state, data_ambiguity) |
| v2 | Mục 9: thay “Quality standards” bằng ví dụ LISA01, LISA02, LISA03; mục 10: checklist 5 mục (Shape, không `__undefined__`, không đèn xe/biển phụ, mũi tên trái = other_direction, visible crop) | Calibration còn 32 box `__undefined__`; peer chỉ nhận guideline nên ví dụ phải nằm trong file | `06_calibration_report.csv` dòng LISA17 (attribute completeness); commit `dcff3e3`, `dbe1853` |
