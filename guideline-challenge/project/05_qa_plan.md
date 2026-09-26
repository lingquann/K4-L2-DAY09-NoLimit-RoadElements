# QA plan + quality gates

## Flow

Guideline → Calibration → Production → Self-QC → Review → Rework → Quality Gate.

- **Ai review, review bao nhiêu:** Review chéo theo vòng: Phúc → Anh Quân → Gia Linh → Thùy Linh → Thành Long →
  Phúc. Mỗi reviewer xem **100% frame có tag `critical` hoặc `conflict`** và **20% ngẫu nhiên** các frame còn lại
  của người mình review (tối thiểu 5 frame/người/lô).
- **Chọn sample theo rule nào:** Ưu tiên theo rủi ro: (1) frame có box `ego_primary`; (2) frame chuyển pha đèn
  (ví dụ LISA16); (3) frame có cụm đèn nhỏ phía xa; (4) annotator mới hoặc annotator có defect critical ở lô trước
  được review 100% lô kế tiếp.
- **Issue được ghi ở đâu, đóng thế nào:** Mỗi defect là một dòng trong sheet review của lô (`sample_id`, box,
  severity, rule vi phạm, người sửa). Annotator sửa và export lại; reviewer mở lại đúng frame đó xác nhận rồi mới
  đóng. Critical chỉ đóng khi hai người (reviewer + QA owner) cùng xác nhận.
- **Khi phát hiện guideline gap thì update và version ra sao:** Defect được chẩn đoán `guideline_gap` → spec owner
  sửa `02_guideline.md`, tăng Version, ghi dòng vào `08_revision_log.md` kèm bằng chứng, dán lại Guide trên CVAT và
  tạo task mới với tên có version (`nolimit-...-v3`). Frame đã label theo bản cũ được re-check đúng rule vừa đổi.

## Defect severity

| Severity | Định nghĩa cho project này | Ví dụ | Action mặc định |
|---|---|---|---|
| Critical | Sai làm đổi quyết định dừng/đi của xe mình | Bỏ sót đầu đèn đỏ `ego_primary`; gán mũi tên đỏ rẽ trái là `ego_primary`; frame có 2 box `ego_primary` khác màu; gán `circle_green` cho đèn đang đỏ | Rework ngay cả lô của annotator đó, review lại 100% lô kế tiếp |
| Major | Sai attribute hoặc thiếu object nhưng không đổi quyết định của đèn ego | Bỏ sót đèn `other_direction` cao ≥ 10 px; gán sai `state` của đèn không phải ego; còn `__undefined__`; gộp 2 hộp đèn vào 1 box | Sửa trong lô, tính vào metric |
| Minor | Sai hình học hoặc cờ phụ | Box lệch > 2 px; box chứa visor/cần vươn/biển phụ; quên `occluded` hoặc `truncated` | Sửa khi còn thời gian, không chặn lô |
| Question | Không đủ bằng chứng trong ảnh, hoặc guideline chưa cover | Đầu đèn nhỏ bị che không rõ màu; mũi tên đỏ vs xanh không phân biệt được bằng `arrow_left` | Ghi lại, chuyển spec owner quyết định rule; không tính là lỗi annotator |

## Metrics

| Metric | Cách tính | Vì sao phù hợp với bài toán |
|---|---|---|
| Ego recall | Số đầu đèn ego_primary gold được annotator vẽ + gán đúng ego_primary / tổng ego_primary gold | Bỏ sót đèn của làn mình là lỗi nặng nhất theo downstream contract |
| Ego uniqueness | Số frame có **đúng 1** box ego_primary (khi gold có 1) / tổng frame review | Calibration v1: 3/5 người có 3 box ego/frame → downstream nhận hiệu lệnh đỏ + xanh cùng lúc (43 lượt frame) |
| State accuracy | Box có `state` đúng / tổng box khớp gold (IoU ≥ 0.5) | Màu và hình đèn là tín hiệu dừng/đi |
| Attribute completeness | Box không còn `__undefined__` / tổng box | Calibration v1 còn 32 box `__undefined__` |
| Object recall (≥ 10 px) | Hộp đèn gold ≥ 10 px có box / tổng hộp đèn gold ≥ 10 px | Calibration v1 lệch 5–7 box/frame ở cụm đèn xa |
| Geometry pass rate | Box có IoU ≥ 0.7 với gold / tổng box khớp | Box ôm vỏ + visor, cho phép lệch ~2 px |

Metric high-risk tách riêng: **critical defect escape rate** = số defect critical lọt qua review (phát hiện ở gate
hoặc ở blind handoff) / tổng defect critical. Mục tiêu 0.

## Quality gate

Threshold là đề xuất của nhóm, không phải chuẩn ngành.

```text
PASS if:
  critical defect = 0
  AND ego recall = 100% AND ego uniqueness = 100%
  AND attribute completeness = 100%
  AND state accuracy >= 95% AND object recall (>= 10 px) >= 90%
  AND geometry pass rate >= 85%
REWORK if: không có critical nhưng một metric major/minor dưới ngưỡng → annotator sửa, reviewer xem lại đúng các frame lỗi
REJECT / ESCALATE if: có >= 1 critical, HOẶC >= 2 annotator mắc cùng một lỗi trên cùng một frame
  (dấu hiệu guideline gap → spec owner sửa guideline, tăng version, label lại lô)
```

Trade-off: Nhóm đặt ngưỡng 100% cho mọi thứ liên quan tới đèn `ego_primary` vì một lỗi ở đó có thể khiến xe vượt
đèn đỏ hoặc phanh gấp — chi phí review 100% frame critical nhỏ so với rủi ro đó. Ngược lại, geometry và đèn
`other_direction` ở xa chỉ ảnh hưởng độ chính xác vị trí, nên chấp nhận ngưỡng thấp hơn (85–90%) để không tốn
thời gian vẽ lại từng pixel trên các đèn chỉ cao 15–25 px.
