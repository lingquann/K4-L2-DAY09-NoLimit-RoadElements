# Edge-case library

Kho nội bộ của nhóm NoLimit, **không gửi cho peer**. Card dùng ảnh example/calibration đã có rule + ví dụ trong
`02_guideline.md` (mục 7 và 9). Card dùng ảnh blind chỉ nằm ở đây, decision phải có trong `gold_decisions.csv`
trước `make freeze`.

Toạ độ là ước lượng trên ảnh gốc 1280×960 để cả nhóm chỉ cùng một đầu đèn. Tên đầu đèn dùng chung:
**mũi tên trái** (x≈732–762, y≈130–197) · **đầu giữa** (x≈927–955, y≈141–209) · **đầu phải** (x≈1140–1170,
y≈187–255) · **đèn xa trái** (x≈678–690, y≈362–386) · **cụm đèn xa** (x≈810–845, y≈398–431).

---

CASE ID: EC01
Sample: LISA06
Scene: Pha đỏ, xe mình dừng tại vạch dừng, cần vươn 3 đầu đèn đều đỏ
Observation: Đầu giữa sáng tròn đỏ, nằm thẳng hàng quỹ đạo đi thẳng của xe
Decision: LABEL
Expected: `traffic_light`, `relevance = ego_primary`, `state = circle_red`, box ôm vỏ + visor
Rationale: Downstream contract mục 3 — bỏ sót hoặc gán sai đèn đỏ của làn mình là vượt đèn đỏ
Common mistake: Gán `other_direction` vì thấy có 3 đầu đèn giống nhau và không chắc đầu nào là của mình
Diversity: critical

---

CASE ID: EC02
Sample: LISA01
Scene: Cần vươn có mũi tên trái đỏ đứng cạnh biển U-turn và đầu đèn tròn đi thẳng
Observation: Mũi tên trái sáng đỏ; calibration 3/5 người gán mũi tên này là `ego_primary`
Decision: LABEL
Expected: Mũi tên trái: `relevance = other_direction`, `state = arrow_left`; box không chứa biển U-turn
Rationale: Xe mình đi thẳng; ego_primary chỉ dành cho đầu đèn điều khiển luồng đi thẳng. Gán mũi tên đỏ là ego → phanh gấp giữa làn (phantom braking), và sau LISA16 “đèn của xe mình” vừa đỏ vừa xanh (43 lượt frame trong calibration v1)
Common mistake: Coi mọi đèn đỏ trên cùng cần vươn là của làn mình; khoanh cả biển U-turn vào box
Diversity: conflict / critical

---

CASE ID: EC03
Sample: LISA18
Scene: Pha xanh, đầu phải cùng cần vươn, vỏ màu tối chìm vào tán cây
Observation: Chỉ thấy rõ thấu kính xanh, viền vỏ khó thấy; calibration: relevance 3 ego / 2 other, state 2 người vẫn ghi đỏ
Decision: LABEL
Expected: `relevance = other_direction`, `state = circle_green`; box ôm vùng thấu kính + phần chao đèn nhìn thấy (mục 5 guideline)
Rationale: Quy tắc v2 mục 4.1: nhiều đầu đèn đi thẳng → chỉ đầu thẳng hàng nhất (đầu giữa) là ego_primary. Giữ đúng một ego_primary để downstream không phải chọn giữa hai đèn
Common mistake: Copy attribute từ frame trước (vẫn ghi đỏ sau khi đèn đã xanh); box to ra tán cây phía sau
Diversity: ambiguity / low_visibility

---

CASE ID: EC04
Sample: LISA16
Scene: Frame chuyển pha đỏ → xanh
Observation: Đầu giữa: ô xanh sáng rõ, ô đỏ phía trên còn vệt mờ. Đèn xa trái không có ô nào sáng trong frame này
Decision: LABEL
Expected: Đầu giữa: `ego_primary`, `circle_green`. Đèn xa trái: `other_direction`, `state = off`
Rationale: Gán theo ô sáng rõ, bão hoà; vệt mờ không phải tín hiệu. Mỗi frame quyết định độc lập (mục 8), không suy màu từ frame trước/sau
Common mistake: Gán đỏ cho đầu giữa vì còn vệt đỏ; gán đỏ cho đèn xa trái vì frame trước nó đỏ; một người gán đèn xa trái là ego_primary + off → downstream nghĩ đèn của mình đang tắt
Diversity: temporal / low_visibility

---

CASE ID: EC05
Sample: LISA10
Scene: Cụm đèn nhỏ phía xa sau giao lộ
Observation: 2–3 hộp đèn cao 18–27 px đứng sát nhau; calibration vẽ 0–2 box ở cụm này, tổng số box/frame 5 · 5 · 6 · 6 · 7
Decision: LABEL
Expected: Mỗi hộp đèn thấy mặt kính và cao ≥ 10 px là một box riêng, `relevance = other_direction`; không vẽ một box chung cho cả cụm
Rationale: v2 mục 2 (3 điều kiện vẽ) và mục 3 (tách cụm). Đèn giữa cần vươn đã là đèn thẳng hàng nhất, các đầu đèn xa là other_direction
Common mistake: Bỏ cả cụm vì “quá xa” dù cao > 10 px; gộp 2 hộp vào một box
Diversity: small_far

---

CASE ID: EC06
Sample: LISA30
Scene: Cụm đèn xa, hộp bên phải (x≈835–845) bị hộp bên cạnh che một phần
Observation: Phần nhìn thấy cao ~18 px, chỉ thấy một đốm sáng nhỏ, không chắc màu; calibration: 2 người không vẽ, 2 người `unknown`, 1 người `red`
Decision: ESCALATE
Expected: Box visible crop, bật `occluded` (Q), `relevance = ambiguous`, `state = unknown`
Rationale: Không đủ điểm ảnh để khẳng định màu và luồng điều khiển; `ambiguous`/`unknown` là đường escalation của guideline (mục 7), tốt hơn đoán một giá trị sai
Common mistake: Đoán `red` theo đèn bên cạnh; ước lượng box bao cả phần bị che
Diversity: escalation / occlusion / small_far

---

CASE ID: EC07
Sample: LISA24
Scene: Pha xanh ổn định
Observation: Mũi tên trái vẫn đỏ, đầu giữa xanh — hai hiệu lệnh ngược nhau trong cùng khung hình
Decision: LABEL
Expected: Mũi tên trái: `other_direction`, `arrow_left`. Đầu giữa: `ego_primary`, `circle_green`. Frame có đúng 1 box ego_primary
Rationale: Frame critical cho quyết định đi: nếu mũi tên đỏ bị gán ego thì downstream nhận đỏ + xanh cùng lúc
Common mistake: Gán cả hai là ego_primary
Diversity: conflict / critical

---

CASE ID: EC08
Sample: LISA13
Scene: Làn đối diện bên trái có xe chạy về phía camera, bật đèn pha
Observation: Nhiều đốm sáng ở tầm thấp gần mặt đường (y≈480–560), không có vỏ hộp hay cột treo
Decision: IGNORE
Expected: Không có box `traffic_light` nào ở vùng y > 430
Rationale: v2 mục 7: đốm sáng tầm thấp không có vỏ hộp/cột treo coi là đèn xe, không vẽ. Box nhầm tạo false positive đèn đỏ → phantom braking
Common mistake: Khoanh đèn hậu đỏ của xe phía xa thành đèn giao thông
Diversity: negative

---

CASE ID: EC09
Sample: LISA02
Scene: Đầu đèn trên cần vươn, nền trời cháy sáng và tán cọ phía sau
Observation: Visor nhô ra phía trên mỗi ô đèn; cần vươn và biển phụ nằm sát hộp đèn
Decision: LABEL
Expected: `geometry:` box ôm sát vỏ + visor, lệch ≤ 2 px mỗi cạnh; không chứa cần vươn, biển phụ hay nền trời
Rationale: v2 mục 3 — box sai kích thước làm lệch vị trí đèn khi downstream khớp đèn với làn
Common mistake: Box rộng ra vùng trời trống hoặc ăn vào cần vươn
Diversity: geometry

---

CASE ID: EC10
Sample: LISA17
Scene: Pha xanh ngay sau chuyển
Observation: Mũi tên trái đang đỏ nhưng `state = arrow_left` không ghi màu; nếu mũi tên chuyển xanh, nhãn vẫn là `arrow_left`
Decision: ESCALATE
Expected: Vẫn gán `other_direction`, `arrow_left` theo v2; ghi lại để quyết định ở v3 có thêm màu cho mũi tên (ví dụ `arrow_left_red` / `arrow_left_green`)
Rationale: Downstream không phân biệt được mũi tên đỏ (cấm rẽ) với mũi tên xanh (được rẽ) — lỗ hổng ontology, không phải lỗi annotator
Common mistake: Tự chọn `circle_red` cho mũi tên đỏ để “giữ màu”
Diversity: ambiguity / guideline gap
