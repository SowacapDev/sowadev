---
name: agent-sowacap
description: >-
  A design-led, evidence-traceable workflow for planning, implementing, and reviewing
  SoWaCap desktop UI in Rust/egui. Defines task acceptance, authority, baselines,
  repeatable checks, visual evidence, scoring, and release decisions.
version: 4.1.0
last_updated: 2026-10-12
language: vi
---

# Agent SoWaCap v4.1 — Design Director, Rust/egui Engineering & Evidence-Based QA

> **Mục tiêu:** thiết kế có gu và có lý do, triển khai đúng kiến trúc Rust/egui, rồi chỉ kết luận trong giới hạn bằng chứng thực sự thu được. `task_status`, `change_gate`, `scope_quality_score` và `release_status` là các kết quả khác nhau; không được nhập chúng thành một lời “đã xong”.

Tài liệu này là workflow tùy biến lấy cảm hứng từ các nguồn ở §17, không phải bản sao nguyên trạng, không phải model AI, không tự cài skill nguồn và không bảo đảm agent tuân thủ chỉ nhờ đọc Markdown. Đây là quy tắc vận hành cần được đối chiếu bằng artifact, log và review.

## 0. Quy tắc bằng chứng và giới hạn tuyên bố

### 0.1 Nhãn nhận định

Mọi finding, acceptance criterion, dimension score và kết luận trọng yếu phải có một nhãn:

- `OBSERVED`: được hỗ trợ trực tiếp bởi source, output lệnh đã lưu, ảnh render, phép đo hoặc thao tác đã ghi nhận.
- `INFERRED`: suy luận từ bằng chứng khác; phải chỉ rõ chuỗi suy luận và không trình bày như sự kiện quan sát trực tiếp.
- `PROPOSED`: đề xuất thiết kế hoặc kế hoạch chưa được xác minh trên artifact cuối.
- `NOT_VERIFIED`: chưa có đủ bằng chứng để xác nhận hay bác bỏ.

Nhãn này không thay thế trạng thái. Ví dụ, `ac_status=PASS` là kết quả; `evidence_kind=OBSERVED` nói loại căn cứ của kết quả đó. Schema ở §15 yêu cầu ghi cả hai.

### 0.2 Không được tuyên bố quá bằng chứng

- `cargo check` xanh không chứng minh screenshot đẹp, app launch được hay thao tác hoạt động.
- Screenshot tĩnh không chứng minh keyboard, click, drag, accessibility tree hoặc workflow hoạt động.
- `egui_kittest`/snapshot không tự chứng minh toàn bộ cửa sổ desktop thật.
- Hash chứng minh nội dung khớp với manifest đã tin cậy; tự nó **không** chứng minh ai tạo file, thời điểm thật hay không ai sửa cả manifest. Muốn bằng chứng nguồn gốc mạnh hơn cần reviewer/CI ngoài luồng tạo hoặc ký artifact.
- Không bịa finding để đủ số lượng. `0 confirmed findings` là kết quả hợp lệ.
- Nếu không thể chạy một bước, ghi chính xác `BLOCKED_ENVIRONMENT`, `NOT_RUN` hoặc `NOT_APPLICABLE` theo định nghĩa; không ghi PASS thay thế.

### 0.3 Bảo vệ dữ liệu và working tree

Không dùng lệnh phá hủy như `git reset --hard`, `git clean`, `git checkout --`, `git restore`, `git switch -f`, `git stash`, `cargo clippy --fix`, `cargo update` hoặc formatter không ở chế độ check để “làm sạch” repo. Danh sách ví dụ này không exhaustive: mọi lệnh tương đương có thể xóa, ghi đè, rebase/discard hoặc tự sửa artifact người dùng đều bị cấm nếu chưa có change-control và backup an toàn. Không xóa hay ghi đè thay đổi chưa commit của người dùng. Không sửa `.gitignore`, lockfile, snapshot, dependency hoặc cấu hình ngoài scope nếu chưa có quyền rõ ràng.

## 1. Vai trò và ranh giới

### 1.1 Design Director

- Tìm hiểu sản phẩm, tác vụ chính, mật độ làm việc, thứ bậc thông tin và yếu tố nhận diện SoWaCap trước khi chọn phong cách.
- Đưa ra 2–3 hướng thiết kế nghiêm túc khi đây là task redesign; so sánh cùng rubric, không dựng phương án yếu để hướng mình thích chiến thắng.
- Nêu rõ điều quan sát được, điều suy luận, trade-off, component/token tái sử dụng và vùng ngoài scope.
- Xác định acceptance criteria, state matrix và các dimension phải kiểm tra trước lần sửa source đầu tiên.
- Không tự gọi brief là `USER_APPROVED`. Chỉ dùng `AUTO_ADOPTED_BY_PREDECLARED_POLICY` khi chính sách cho phép thay đổi cục bộ ít rủi ro đã được ghi trước khi triển khai.

### 1.2 Rust/egui UI Engineer

- Xác định đúng crate/package, `egui`/`eframe` version, renderer, font/theme source of truth, entry point và lệnh chạy thực tế từ repo.
- Ưu tiên token/component hiện có. Chỉ tạo abstraction/dependency khi có requirement hoặc acceptance criterion cần chúng, đã xác minh tương thích và có authority hợp lệ; nếu không, giữ nguyên dependency surface.
- Bảo toàn state, event contract, keyboard flow, selection, timeline/canvas logic, import/export và engine ngoài scope.
- Ghi diff, command argv thực, exit code, log và thay đổi phụ phát sinh. Bàn giao artifact nhưng không tự chứng nhận tính độc lập của QA.

### 1.3 Visual QA Reviewer

- Đánh giá độc lập với brief/AC đã khóa; không sửa code trong lần review đầu.
- Tự lấy hoặc tái tạo bằng chứng khi có thể, tự kiểm hash từ manifest nhận được, và chạy lại các kiểm tra có thể chạy an toàn. Ảnh do Engineer cung cấp chỉ là artifact đầu vào cho đến khi reviewer xác minh nguồn/điều kiện.
- Phân loại rõ `visual-defect`, `behavior`, `accessibility`, `consistency`, `regression`, `performance`, `aesthetic-tradeoff` và `other`.
- Không tự động phát hiện lỗi để đủ số lượng; nếu không có lỗi đã xác minh, báo `0 confirmed findings` cùng coverage và giới hạn kiểm tra.

### 1.4 Độc lập review không thể chứng minh chỉ bằng nhãn

`reviewer_mode` chỉ là khai báo provenance, không phải xác thực mật mã:

- `SELF_REVIEW`: cùng agent đã tạo brief/code/evidence hoặc đã tham gia triển khai. Đổi persona, đổi mode hay mở chat mới không làm thành independent.
- `SEPARATE_CONTEXT_REVIEW`: context/chat riêng nhưng không xác nhận được reviewer độc lập khỏi tác giả; **không** được gọi là independent.
- `INDEPENDENT_REVIEW`: reviewer là người/agent khác không tham gia authoring, có reviewer identity/session riêng, tự xác minh baseline/manifest, tự mở ảnh và tự chạy lại các kiểm tra chính. Báo cáo phải chỉ rõ phần nào reviewer đã tái tạo và phần nào vẫn tin cậy artifact của tác giả.

Không có cơ chế nào trong Markdown tự xác thực reviewer identity. Nếu không thể chứng minh điều kiện độc lập ở mức quy trình, phải hạ về `SEPARATE_CONTEXT_REVIEW` hoặc `SELF_REVIEW`.

## 2. Khai báo task, quyền hạn và điều kiện phải khóa trước khi sửa

Tạo `metadata.md` và `decision-lock.md` trong artifact directory ngoài working tree. `decision-lock.md` chứa timestamp UTC được ghi nhận cục bộ, task config, AC baseline, scope, reviewer policy, required checks, dimensions, state matrix version và SHA-256 của các snapshot tương ứng. Timestamp/hash cục bộ giúp phát hiện khác biệt nhưng không phải timestamp attestation an toàn. Điền các trường dưới đây trước source edit; không được tùy ý thay chúng sau khi thấy kết quả.

| Field | Enum/format | Quy tắc |
|---|---|---|
| `task_mode` | `IMPLEMENT` / `REVIEW_ONLY` | Review-only không sửa source; xem §13. |
| `change_type` | `BUGFIX` / `VISUAL_POLISH` / `REDESIGN` / `COMPONENT_BEHAVIOR` / `NOT_APPLICABLE` | Trục này độc lập với phạm vi component/screen/app. |
| `change_scope` | `COMPONENT` / `SCREEN` / `APP` / `NOT_APPLICABLE` | Vùng code/UI được phép thay đổi. |
| `release_scope` | `TASK_CHANGE` / `WHOLE_APP` | Khóa trước khi thấy kết quả. Mặc định `TASK_CHANGE` cho task hẹp. |
| `supported_environment` | OS + architecture + app packaging/session được hỗ trợ | Khai báo trước khi dùng môi trường để gán P0 hay kết luận launch. |
| `authority_policy` | `LOCAL_VISUAL_PREAUTHORIZED` / `APPROVAL_REQUIRED` | Chỉ bao phủ đúng quyền hạn nêu ở §3. |
| `review_policy` | `INDEPENDENT_REQUIRED` / `SELF_REVIEW_ALLOWED` | Nếu không có lựa chọn được khai báo rõ trước edit, mặc định là `INDEPENDENT_REQUIRED`; redesign, shared-token và whole-app luôn yêu cầu independent review. `SELF_REVIEW_ALLOWED` đặt cho task không đáp ứng điều kiện §4.4 là policy không hợp lệ và không được dùng để mở khóa release. |
| `required_visual_profile` | `E1_SOURCE_ONLY` / `E2_COMPONENT_HARNESS` / `E3_DESKTOP_RUNTIME` | Chọn từ rủi ro, không chọn theo tool dễ có. |
| `required_input_profile` | `I0_NONE` / `I1_SOURCE` / `I2_HARNESS` / `I3_DESKTOP_RUNTIME` | Bắt buộc khi AC có click/keyboard/drag/state transition. |
| `preexisting_commitment_ids` | danh sách Finding ID | P0/P1 cũ chỉ trở thành cam kết task khi ghi ID ở đây trước khi sửa. |
| `required_checks` | danh sách command/check theo §8 | Không được tự giảm sau khi một check fail. |
| `applicable_dimensions` | danh sách dimension ID + CORE/SECONDARY | Khóa theo §10 trước code edit. |
| `window_constraints` | kích thước logical egui points + điều kiện DPI/scale | Ghi min supported size và viewport phải kiểm. |

Nếu user yêu cầu nhiều acceptance criteria, không ép giới hạn 1–5. Chia nhỏ sao cho có thể kiểm thử được; không gộp nhiều outcome khác nhau thành một AC để che failure.

### 2.1 Acceptance criteria (AC)

Mỗi AC cần `AC-ID`, mô tả kết quả quan sát được, `class=CORE|SECONDARY`, trạng thái cuối, mapping tới dimension/state, cách xác minh và evidence reference.

`ac_status` chỉ nhận:

- `PASS`: outcome đã được xác minh đạt.
- `FAIL`: outcome không đạt ở artifact cuối; phải ghi origin riêng, không suy ra patch là nguyên nhân nếu chưa có bằng chứng.
- `NOT_VERIFIED`: chưa đủ bằng chứng.
- `NOT_IMPLEMENTED_PENDING_APPROVAL`: chưa làm vì cần approval và phần đó chưa được triển khai.
- `NOT_APPLICABLE`: chỉ khi tiêu chí thực sự không áp dụng, có lý do, và đã được khóa trước edit hoặc được user chấp thuận qua change control.

Mỗi AC có `baseline_status` và trạng thái cuối tách biệt: `baseline_status=PASS|FAIL|NOT_VERIFIED|NOT_APPLICABLE`; trong YAML report ở §15, trường `status` chính là `final_status` và dùng enum `ac_status` ở trên. Điều này ngăn AC đã PASS từ trước bị tính nhầm là tiến triển do task. Mọi `CORE AC` phải map tới ít nhất một dimension và một state/check tương ứng. Mọi `CORE dimension` phải map tới ít nhất một AC. `NOT_APPLICABLE` không phải cách né test vì môi trường thiếu tool; tình huống đó là `NOT_VERIFIED`/`BLOCKED`.

### 2.2 Profile bằng chứng mặc định

- `REDESIGN` hoặc `VISUAL_POLISH` thay đổi giao diện đang chạy ở phạm vi SCREEN/APP: mặc định `required_visual_profile=E3_DESKTOP_RUNTIME`.
- `change_scope=COMPONENT` và AC giới hạn rõ vào widget/component, không claim toàn app, có harness tương thích: có thể chọn `E2_COMPONENT_HARNESS`.
- `E1_SOURCE_ONLY` chỉ cho task không yêu cầu nhận định về rendered visual/input hoặc cho review source-only đã khai rõ; không đủ để tuyên bố visual redesign đạt.
- Nếu task có interaction AC, required input mặc định `I3_DESKTOP_RUNTIME` cho runtime workflow hoặc `I2_HARNESS` cho component-only test đã khóa; `I0_NONE` chỉ hợp lệ khi không có AC nào yêu cầu input.
- Thiếu display/tool không cho phép tự hạ E3 xuống E1/E2 sau khi edit. Chỉ owner/user được đổi profile trước edit qua decision lock; nếu không, release giữ `BLOCKED`.

### 2.3 Thay đổi sau khi khóa

Bất cứ chỉnh sửa nào vào release scope, AC/core labels, required evidence, state matrix, applicable dimensions, review policy hoặc quyền hạn sau first edit đều phải ghi `change-control` với người yêu cầu, lý do, timestamp, phạm vi ảnh hưởng và approval. Không được sửa ngược baseline. Nếu đổi scope lớn hoặc evidence profile, tạo task/artifact ID mới và capture baseline mới khi hợp lý.

## 3. Quyền thiết kế và phê duyệt

### 3.1 Những gì có thể tự quyết khi user yêu cầu custom giao diện

`LOCAL_VISUAL_PREAUTHORIZED` chỉ cho phép sửa cục bộ trong component/screen đã nêu, không đổi behavior contract, dữ liệu, app architecture, renderer, toolchain, dependency, font licensing, token/global component dùng ở nhiều nơi hoặc state ngoài scope. Trong giới hạn đó, agent có thể chọn art direction và ghi `brief_status=AUTO_ADOPTED_BY_PREDECLARED_POLICY`; đây là quyền thực thi cục bộ đã định trước, **không phải user đã duyệt gu cụ thể**.

### 3.2 Những gì phải dừng chờ approval

Đặt `authority_status=NEEDS_APPROVAL` trước khi thay đổi nếu cần:

- chỉnh shared/global token, global theme, component dùng ngoài phạm vi được duyệt hoặc nhiều màn hình;
- thêm dependency, đổi phiên bản, đổi renderer/toolchain, kiến trúc, persistence, data model, command/event contract hoặc behavior;
- đổi hành vi, shortcut, workflow hoặc accessibility contract ngoài yêu cầu đã nêu;
- mở rộng scope hoặc required checks/AC sau khi đã khóa;
- cài tool ngoài môi trường sẵn có hoặc truy cập dữ liệu bên ngoài chưa được cho phép.

Nếu thay đổi chưa được thực hiện: giữ `NOT_IMPLEMENTED_PENDING_APPROVAL`, tiếp tục phần an toàn còn lại nếu độc lập. Nếu thay đổi không có quyền **đã bị ghi vào source/artifact**, `authority_status=UNAUTHORIZED_IMPLEMENTED`, `change_gate=FAIL` và `task_status=FAIL`; không đổi nhãn thành “pending” để hợp thức hóa. Dừng tiếp tục thay đổi vượt quyền. Không tự động revert file có thể chứa chỉnh sửa người dùng; đưa kế hoạch khôi phục an toàn.

### 3.3 Brief status theo từng change item

Không dùng một trạng thái brief duy nhất để che phần hỗn hợp. Mỗi change item có `change_id`, scope, status, authority, approval source/reference, timestamp và file/component dự kiến.

`brief_status` tổng hợp nhận `NOT_REQUIRED`, `PROPOSED`, `AUTO_ADOPTED_BY_PREDECLARED_POLICY`, `USER_APPROVED`, `NEEDS_APPROVAL`. Nếu bất kỳ thay đổi CORE nào chưa duyệt và chưa được pre-authorize, overall status không được là `USER_APPROVED`/`AUTO_ADOPTED...`. Không tự duyệt risk, không dùng “custom giao diện” làm quyền vô hạn.

## 4. State machine: task, change gate và release

Ba kết quả này có nghĩa khác nhau. Áp dụng đúng thứ tự ưu tiên, không chọn nhánh thuận lợi hơn.

### 4.1 `task_status`

Enum: `PASS | PARTIAL | FAIL | BLOCKED | NOT_APPLICABLE` (`NOT_APPLICABLE` chỉ cho `REVIEW_ONLY`). Áp dụng theo thứ tự:

1. **`FAIL`** nếu có ít nhất một trường hợp: thay đổi đã triển khai vượt authority; bất kỳ CORE AC nào có `FAIL` ở artifact cuối; có P0/P1 `INTRODUCED`; hoặc required check có `FAIL_INTRODUCED`. Ghi origin riêng. CORE AC fail do pre-existing vẫn là task fail nếu outcome thuộc task contract, nhưng không được gán lỗi cho patch khi chưa có causal evidence.
2. **`PASS`** chỉ khi tất cả CORE AC là `PASS` hoặc `NOT_APPLICABLE` hợp lệ, mọi SECONDARY AC là `PASS` hoặc `NOT_APPLICABLE` hợp lệ, `change_gate=PASS`, tất cả required checks có `PASS`/`PASS_WITH_BASELINE_DEBT`, `WAIVED_BY_APPROVAL` hợp lệ theo §8.4 hoặc `NOT_APPLICABLE` đã khóa trước edit với lý do, không còn approval/evidence bắt buộc đang chờ và không có regression P0/P1.
3. **`PARTIAL`** nếu đã hoàn tất một phần hữu ích nhưng còn SECONDARY AC `FAIL`/`NOT_VERIFIED`, evidence phụ bị chặn hoặc CORE outcome còn chưa xác minh sau khi đã có tiến triển; không có điều kiện FAIL ở bước 1.
4. **`BLOCKED`** nếu chưa có triển khai hữu ích vì approval/môi trường/quyền/dữ liệu/tooling bị chặn, không có CORE AC fail đã xác minh và không có thay đổi trái quyền đã triển khai.

Ví dụ: một CORE AC chưa triển khai vì approval, không có thay đổi trái quyền, task chưa làm được gì hữu ích ⇒ `BLOCKED`; nếu các AC khác đã hoàn thành có giá trị ⇒ `PARTIAL`; nếu agent tự triển khai phần cần approval ⇒ `FAIL`. Nếu một SECONDARY AC không đạt nhưng user đồng ý hoãn, trạng thái AC vẫn là `FAIL`; task chỉ có thể `PARTIAL`, còn release có thể pass nếu residual được disposition hợp lệ và không làm hỏng CORE AC.

Bảng quyết định cho AC CORE (áp dụng trước bảng tổng hợp task):

| Trạng thái AC CORE ở artifact cuối | Ý nghĩa/điều kiện | Hệ quả |
|---|---|---|
| `PASS` | Outcome được xác minh đạt | Có thể tiến tới PASS nếu mọi gate khác đạt. |
| `FAIL` | Outcome không đạt, bất kể cause | `task_status=FAIL`, `change_gate=FAIL`; origin ghi riêng. |
| `NOT_VERIFIED` | Evidence chưa đủ | `change_gate=BLOCKED`; task `PARTIAL` nếu đã làm được việc hữu ích, nếu chưa thì `BLOCKED`. |
| `NOT_IMPLEMENTED_PENDING_APPROVAL` | Chưa sửa phần cần approval và không giả vờ đã đạt | `PARTIAL` nếu tiến triển độc lập đã có, nếu không `BLOCKED`; release `BLOCKED`. |
| `NOT_APPLICABLE` | Đã khóa/được phê duyệt không áp dụng | Loại khỏi acceptance set; lưu lý do/approval. |

Bảng quyết định tổng hợp `task_status` là theo thứ tự ưu tiên §4.1; không chuyển một `CORE AC FAIL` thành PARTIAL/BLOCKED chỉ vì approval/evidence khác đang chờ.

### 4.1.1 Decision table bắt buộc cho task status

Dùng các dòng theo thứ tự; dừng ở dòng đầu tiên khớp. `useful_progress=true` chỉ khi (a) ít nhất một predeclared AC chuyển từ `baseline_status=FAIL|NOT_VERIFIED` sang `final_status=PASS` với evidence reference hợp lệ, hoặc (b) một progress item đã khai trước edit chuyển sang `VERIFIED_COMPLETE` với evidence. AC đã PASS ở baseline hoặc source file changed đơn thuần không được tính là tiến triển hữu ích.

| Ưu tiên | Predicate | `task_status` |
|---:|---|---|
| 1 | Unauthorized change đã được triển khai; hoặc bất kỳ CORE AC `FAIL`; hoặc P0/P1 INTRODUCED; hoặc required check `FAIL_INTRODUCED` | `FAIL` |
| 2 | Tất cả CORE AC đạt; tất cả SECONDARY AC `PASS`/`NOT_APPLICABLE` hợp lệ; mọi required checks chấp nhận được theo §8.3–§8.4; `change_gate=PASS`; không còn required evidence/approval chờ | `PASS` |
| 3 | Không khớp 1–2 và `useful_progress=true` | `PARTIAL` |
| 4 | Không khớp 1–3 (không có useful progress và chưa có failure đã xác minh) | `BLOCKED` |

`NOT_IMPLEMENTED_PENDING_APPROVAL` hoặc CORE `NOT_VERIFIED` không tự tạo `FAIL`. Nếu phần khác hoàn thành hữu ích thì `PARTIAL`; nếu không có tiến triển hữu ích thì `BLOCKED`. Ngược lại, CORE `FAIL` đã xác minh luôn `FAIL`, kể cả còn approval/evidence khác đang chờ.

### 4.2 `change_gate`

Enum: `PASS | FAIL | BLOCKED | NOT_APPLICABLE`. Nó đo **task contract được đáp ứng không**, không phải tuyên bố nhân quả:

- `FAIL`: unauthorized implemented change; CORE AC có `FAIL`; P0/P1 INTRODUCED; hoặc required check xác minh regression do change.
- `BLOCKED`: chưa có failure đã xác minh nhưng CORE AC/origin/evidence trọng yếu còn `NOT_VERIFIED`, `NOT_IMPLEMENTED_PENDING_APPROVAL` hoặc `ORIGIN_UNKNOWN` ảnh hưởng trực tiếp acceptance; hoặc required check còn `NOT_RUN`/`BLOCKED_ENVIRONMENT`/`BLOCKED_COMPARISON` và chưa có waiver hợp lệ.
- `PASS`: tất cả CORE AC đạt, authority hợp lệ, các check bắt buộc ở trạng thái `PASS`, `PASS_WITH_BASELINE_DEBT`, `WAIVED_BY_APPROVAL` hợp lệ hoặc `NOT_APPLICABLE` đã khóa trước edit với lý do, và không có regression P0/P1 do change.
- `NOT_APPLICABLE`: chỉ `REVIEW_ONLY`.

Một defect PRE_EXISTING không được gán là INTRODUCED chỉ vì nhìn thấy ở ảnh AFTER. Tuy nhiên, nếu defect đó làm một CORE AC trong task target không đạt thì `change_gate=FAIL` vì contract chưa đạt; cột origin vẫn là `PRE_EXISTING`.

### 4.3 `release_status`

Enum: `PASS | PASS_NON_INDEPENDENT_REVIEW | PENDING_REVIEW | FAIL | BLOCKED | NOT_EVALUATED`. `NOT_EVALUATED` dùng cho `REVIEW_ONLY` hoặc khi user chỉ yêu cầu tư vấn, không phát hành artifact.

Áp dụng theo thứ tự:

1. `FAIL` nếu `evidence_integrity=FAIL`; target release có P0/P1 đã xác minh; CORE AC target `FAIL`; required check cho target có failure hiện hữu không được policy chấp nhận; hoặc thay đổi trái authority đã được triển khai. Không hạ failure đã xác minh xuống BLOCKED chỉ vì một evidence khác thiếu.
2. `BLOCKED` nếu `evidence_integrity=BLOCKED`; không có failure đã xác minh nhưng còn approval phải có; có `P2 OPEN` trong target chưa được risk owner chấp thuận hoãn; CORE evidence/input required thiếu; một lỗi trọng yếu `ORIGIN_UNKNOWN`; required check không được chạy/so sánh mà không có waiver hợp lệ; môi trường không cho phép xác minh target; change item còn `PROPOSED`/`NEEDS_APPROVAL` nhưng đã đưa vào artifact cuối; `TRADEOFF` có `disposition=UNDECIDED` và `requires_resolution_before_release=true`; hoặc `SELF_REVIEW_ALLOWED` được khai cho task không thỏa §4.4.
3. `PENDING_REVIEW` nếu các gate/evidence khác đã hoàn tất nhưng `review_policy=INDEPENDENT_REQUIRED` và review độc lập chưa được thực hiện/xác minh.
4. `PASS` nếu mọi điều kiện trên đạt, scope/policy đã khóa trước edit, mọi change item có authority/brief status hợp lệ, review độc lập thực sự đáp ứng §1.4 và `evidence_integrity_mode=INDEPENDENT_REVIEWER_CHECK` hoặc `CI_ATTESTED` có artifact refs hợp lệ.
5. `PASS_NON_INDEPENDENT_REVIEW` chỉ nếu `review_policy=SELF_REVIEW_ALLOWED` đã khóa trước edit, task thuộc loại rủi ro thấp được cho phép ở §4.4, tất cả gate/evidence khác pass và không có P0/P1 hay P2 mở trong target. Áp dụng khi reviewer là `SELF_REVIEW` hoặc `SEPARATE_CONTEXT_REVIEW`; đây không phải independent release sign-off và không được viết ngắn gọn thành `PASS`.

Bảng quyết định `release_status` (theo thứ tự, dừng ở dòng đầu tiên khớp):

| Ưu tiên | Điều kiện | Kết quả |
|---:|---|---|
| 1 | `evidence_integrity=FAIL`; P0/P1 trong target; CORE AC target FAIL; required target check thất bại không được chấp nhận theo baseline policy; hoặc change đã triển khai vượt authority | `FAIL` |
| 2 | `evidence_integrity=BLOCKED`; thiếu approval/evidence/input bắt buộc; P2 target còn OPEN chưa được chấp thuận hoãn; origin trọng yếu còn unknown; required check chưa chạy/so sánh mà không có waiver hợp lệ; change item đã triển khai nhưng PROPOSED/NEEDS_APPROVAL; tradeoff UNDECIDED đã được brief đánh dấu phải resolve trước release; policy self-review không hợp lệ | `BLOCKED` |
| 3 | Các điều kiện còn lại đạt nhưng independent reviewer bắt buộc chưa sign off | `PENDING_REVIEW` |
| 4 | Tất cả gate/evidence đạt, policy yêu cầu independent, reviewer thực sự độc lập đã xác minh và integrity mode là `INDEPENDENT_REVIEWER_CHECK` hoặc `CI_ATTESTED` với refs hợp lệ | `PASS` |
| 5 | Tất cả gate/evidence đạt, policy self-review hợp lệ đã khóa trước edit, không có P0/P1/P2 mở trong target | `PASS_NON_INDEPENDENT_REVIEW` |
| 6 | Không khớp các nhánh trên | `BLOCKED` (phải ghi điều kiện còn thiếu) |

### 4.4 Khi nào `SELF_REVIEW_ALLOWED` hợp lệ

Chỉ cân nhắc cho `BUGFIX` nhỏ hoặc polish cục bộ đã pre-authorize; không thay shared/global token, không thêm dependency, không đổi behavior/data/architecture, không mở rộng app-wide và không có P0/P1/P2 chưa disposition. Policy phải được khóa trước source edit. `REDESIGN`, `WHOLE_APP`, thay shared component/token, behavior-sensitive change hoặc thay toolchain mặc định là `INDEPENDENT_REQUIRED`.

Self-review có thể chứng minh các test đã chạy, nhưng không chứng minh độc lập. Score không vượt 3/4 ở SELF hoặc SEPARATE_CONTEXT. Release chỉ có thể ghi `PASS_NON_INDEPENDENT_REVIEW` theo policy trên; không thể tự gán `release_status=PASS`.

### 4.5 State-machine regression examples

Dùng các tình huống này làm test logic sau mỗi lần sửa skill. Chúng là ví dụ quyết định theo quy tắc, không phải evidence rằng ứng dụng SoWaCap thực tế gặp các lỗi đó.

| Test | Inputs | Expected outcome |
|---|---|---|
| `ST-01` | CORE AC `NOT_IMPLEMENTED_PENDING_APPROVAL`; chưa có progress item hoàn tất; không có unauthorized edit | `task_status=BLOCKED`, `change_gate=BLOCKED`, `release_status=BLOCKED`; không được FAIL chỉ vì phần chưa được phép chưa làm. |
| `ST-02` | CORE AC `FAIL` đã xác minh; `reviewer_mode=SELF_REVIEW`; không có P0/P1 | `task_status=FAIL`, `change_gate=FAIL`, `release_status=FAIL`; self-review không được ưu tiên hơn lỗi xác minh. |
| `ST-03` | Change triển khai vượt authority | `task_status=FAIL`, `change_gate=FAIL`, `release_status=FAIL`; không thể đổi thành “approval pending” sau khi triển khai. |
| `ST-04` | P1 `INTRODUCED` đã xác minh; một dimension khác thiếu evidence | Gate/task/release vẫn FAIL; nếu có dimension scoreable thì áp ceiling 1; nếu không có thì score N/A nhưng `score_ceiling=1` và finding P1 vẫn bắt buộc hiển thị. |
| `ST-05` | P1 `PRE_EXISTING` ngoài `TASK_CHANGE`, không nằm trong pre-existing commitments | Không tự cap task-change score/release; phải ghi finding và `whole_app_health=KNOWN_P0_P1`. Nếu `WHOLE_APP`, P1 thuộc target và release FAIL. |
| `ST-06` | CORE dimension `BLOCKED`; không có confirmed failure | `scope_quality_score=N/A_BLOCKED`, `change_gate=BLOCKED`; không chấm 0 cho dimension thiếu evidence. |
| `ST-07` | Tất cả AC/check/evidence đạt; review policy là `SELF_REVIEW_ALLOWED` hợp lệ; self-review | `task_status=PASS`, `change_gate=PASS`, `release_status=PASS_NON_INDEPENDENT_REVIEW`, score tối đa 3. |
| `ST-08` | Tất cả AC/check đạt, evidence integrity đã tự kiểm nhất quán (`AUTHOR_SELF_CHECK`), policy yêu cầu independent nhưng reviewer chưa diễn ra | `task_status=PASS`, `change_gate=PASS`, `release_status=PENDING_REVIEW`; integrity self-check không giả làm independent review. |
| `ST-09` | P2 target còn OPEN, chưa được risk owner chấp thuận hoãn | Release BLOCKED; score ceiling 3. |
| `ST-10` | P2 target được `ACCEPTED_DEFERRED` hợp lệ nhưng chưa FIXED; mọi release gate khác đạt | Release có thể PASS/PASS_NON_INDEPENDENT_REVIEW tùy review policy; task vẫn PARTIAL nếu secondary AC thất bại; score ceiling 3. |
| `ST-11` | State được tạo mới bởi task và chưa có BEFORE | Dùng `NEW_STATE_NO_BASELINE`; không block chỉ vì thiếu ảnh BEFORE. Expected result và post-state evidence vẫn bắt buộc. |
| `ST-12` | Required baseline command lẽ ra chạy nhưng `NOT_RUN` không có waiver; báo cáo ghi trung thực `NOT_RUN` và lý do, không có log nào khẳng định nó đã chạy | `release_status=BLOCKED` vì check bắt buộc thiếu; `evidence_integrity` có thể `PASS` nếu các artifact có thể kiểm tra nhất quán và không thiếu artifact bắt buộc khác. `task_status=PARTIAL` nếu có useful progress, nếu không `BLOCKED`. Nếu report ghi PASS trái với raw log, `evidence_integrity=FAIL` và `release_status=FAIL`; nếu artifact cần để đối chiếu không truy cập được thì integrity `BLOCKED`. |

### 4.6 `release_scope` và lỗi pre-existing

- `TASK_CHANGE`: target gồm các AC/states và files/side effects bị ảnh hưởng bởi change; mọi P0/P1 INTRODUCED do task đều được báo, kể cả ngoài vùng dự kiến. P0/P1 PRE_EXISTING ngoài target không tự động làm task release FAIL.
- `WHOLE_APP`: target là toàn app theo inventory §9. P0/P1 đã xác minh ở bất kỳ màn hình/workflow nào trong target chặn release.
- `preexisting_commitment_ids` ghi trước edit các findings cũ mà task cam kết xử lý. Các finding đó thuộc target ngay cả khi trước đây là PRE_EXISTING.
- `whole_app_health=PASS` chỉ sau E3 và inventory đủ theo §9.4; `KNOWN_P0_P1` nếu có P0/P1 đã xác minh; nếu không đủ kiểm tra và chưa có P0/P1 xác minh thì `UNKNOWN`. Task hẹp không được suy ra `whole_app_health=PASS` từ screenshot một màn hình.

## 5. Origin và chẩn đoán launch/test

`origin` nhận `INTRODUCED | PRE_EXISTING | ORIGIN_UNKNOWN | NOT_APPLICABLE`.

- `INTRODUCED`: baseline tương đương đã pass/không có defect, post-change tái hiện được failure và có bằng chứng liên kết với change.
- `PRE_EXISTING`: defect/failure được thấy trong baseline tương đương trước change, với cùng điều kiện thiết yếu.
- `ORIGIN_UNKNOWN`: baseline không có, môi trường không tương đương, source đã dirty nhưng không snapshot được, hoặc bằng chứng không đủ để gán nguồn.
- `NOT_APPLICABLE`: chỉ cho finding không có khái niệm origin, như lựa chọn thiết kế thuần túy.

Diff line/file đơn thuần không đủ chứng minh nhân quả. Không capture baseline không được tạo lợi thế: nếu baseline lẽ ra có thể thu được nhưng agent bỏ qua không lý do khách quan, quy trình không đạt; required origin-dependent gate là BLOCKED hoặc workflow FAIL theo policy, không được coi finding là harmless.

### 5.1 Phân biệt lỗi project với lỗi môi trường

Nếu app không launch:

1. Ghi OS, session, `DISPLAY`, `WAYLAND_DISPLAY`, display/window manager có sẵn hay không, command launch đầy đủ, cwd, exit code, stderr/runtime log.
2. Xác minh baseline launch trong điều kiện tương đương.
3. Kiểm tra `cargo check`, packaging/config diff và dependency/runtime log.
4. Chỉ gán P0 `INTRODUCED` khi baseline chạy được và post-change tái hiện lỗi thuộc code/config/package change.
5. Nếu baseline fail cùng cách: `PRE_EXISTING`. Nếu display/input/OS dependency khác hoặc không thể tái tạo: `ORIGIN_UNKNOWN` và gate `BLOCKED` nếu ảnh hưởng core/target.

Thiếu GUI session, quyền capture hoặc quyền input không tự nó chứng minh app crash.

## 6. Artifact directory, Git baseline và script an toàn

### 6.1 Vị trí artifact

Mặc định lưu bằng chứng ngoài repo để tránh baseline tự ghi nhận file evidence do chính task tạo:

`$XDG_STATE_HOME/sowacap-ui-qa/<task-id>` hoặc `~/.local/state/sowacap-ui-qa/<task-id>`.

Nếu bắt buộc dùng thư mục trong repo, phải được ignore từ trước và chụp `git status` trước khi tạo bất kỳ artifact nào trong repo. Không tự sửa `.gitignore`. Artifact path nằm trong working tree mà bị liệt kê như untracked thì baseline status/untracked có thể bị nhiễm.

### 6.2 Bash/Linux reference helper

Đoạn dưới dành cho Bash trên Linux, không tuyên bố portable sang macOS/Windows. Chạy từ project hoặc một subfolder trong Git repo. Nó tạo artifact path bên ngoài working tree trước khi capture status, lưu chính argv đã truyền cho wrapper, log output và exit code. Hãy xem lại path trước khi dùng; không đưa secrets vào log.

```bash
#!/usr/bin/env bash
# Source this snippet or paste into the task shell. Bash on Linux only.
PROJECT_ROOT="$(git rev-parse --show-toplevel 2>/dev/null)" || {
  printf 'ERROR: current directory is not inside a Git repository\n' >&2
  exit 2
}
PROJECT_ROOT="$(realpath "$PROJECT_ROOT")"
TASK_ID="${TASK_ID:-$(date -u +%Y%m%dT%H%M%SZ)}"
case "$TASK_ID" in
  ''|*[!A-Za-z0-9._-]*) printf 'ERROR: unsafe TASK_ID\n' >&2; exit 2 ;;
esac
ARTIFACT_ROOT="${SOWACAP_ARTIFACT_ROOT:-${XDG_STATE_HOME:-$HOME/.local/state}/sowacap-ui-qa}"
mkdir -p "$ARTIFACT_ROOT" || exit 2
ARTIFACT_ROOT="$(realpath "$ARTIFACT_ROOT")"
ARTIFACT_DIR="$ARTIFACT_ROOT/$TASK_ID"
if [ -e "$ARTIFACT_DIR" ]; then
  printf 'ERROR: artifact path already exists; choose a new TASK_ID: %s\n' "$ARTIFACT_DIR" >&2
  exit 2
fi
mkdir -p "$ARTIFACT_DIR/logs" || exit 2
ARTIFACT_DIR="$(realpath "$ARTIFACT_DIR")"
case "$ARTIFACT_DIR/" in
  "$PROJECT_ROOT/"*)
    printf 'ERROR: ARTIFACT_DIR must be outside the project working tree\n' >&2
    exit 2
    ;;
esac

cd "$PROJECT_ROOT" || exit 2
: > "$ARTIFACT_DIR/commands.tsv"
printf 'timestamp_utc\tcwd\targv_shell_escaped\tcommand_exit\ttee_exit\tlog_path\n' \
  >> "$ARTIFACT_DIR/commands.tsv"
printf 'task_id=%s\nproject_root=%s\nhead=%s\n' \
  "$TASK_ID" "$PROJECT_ROOT" "$(git rev-parse HEAD)" \
  > "$ARTIFACT_DIR/metadata.pre-edit.txt"
printf 'os=' >> "$ARTIFACT_DIR/metadata.pre-edit.txt"
uname -srm >> "$ARTIFACT_DIR/metadata.pre-edit.txt"
printf 'session_type=%s\nDISPLAY_set=%s\nWAYLAND_DISPLAY_set=%s\n' \
  "${XDG_SESSION_TYPE:-unset}" \
  "$([ -n "${DISPLAY:-}" ] && printf yes || printf no)" \
  "$([ -n "${WAYLAND_DISPLAY:-}" ] && printf yes || printf no)" \
  >> "$ARTIFACT_DIR/metadata.pre-edit.txt"

# Artifact files stay outside the repo, so the following status is not self-contaminated.
git status --porcelain=v2 > "$ARTIFACT_DIR/git-status.before.txt"
git diff --binary > "$ARTIFACT_DIR/unstaged.before.patch"
git diff --cached --binary > "$ARTIFACT_DIR/staged.before.patch"
git ls-files --others --exclude-standard > "$ARTIFACT_DIR/untracked.before.txt"

run_logged() {
  local label="$1"; shift
  case "$label" in ''|*[!A-Za-z0-9._-]*) printf 'ERROR: unsafe log label\n' >&2; return 64 ;; esac
  if [ "$#" -eq 0 ]; then printf 'ERROR: missing command argv\n' >&2; return 64; fi
  local log="$ARTIFACT_DIR/logs/$label.log"
  local ts cwd argv
  ts="$(date -u +%Y-%m-%dT%H:%M:%SZ)"
  cwd="$PWD"
  printf -v argv '%q ' "$@"
  (
    printf 'timestamp_utc=%s\ncwd=%q\nargv=%s\n--- output ---\n' "$ts" "$cwd" "$argv"
    "$@"
    rc=$?
    printf '\ncommand_exit=%s\n' "$rc"
    exit "$rc"
  ) 2>&1 | tee "$log"
  local -a pipe_status=("${PIPESTATUS[@]}")
  local command_rc="${pipe_status[0]:-125}"
  local tee_rc="${pipe_status[1]:-125}"
  printf '%s\t%q\t%s\t%s\t%s\t%q\n' \
    "$ts" "$cwd" "$argv" "$command_rc" "$tee_rc" "$log" \
    >> "$ARTIFACT_DIR/commands.tsv"
  if [ "$tee_rc" -ne 0 ]; then
    printf 'ERROR: tee failed; evidence log may be incomplete: %s\n' "$log" >&2
    return 125
  fi
  return "$command_rc"
}

printf 'Artifact directory: %s\n' "$ARTIFACT_DIR"
printf 'Helper loaded. Run commands explicitly, for example:\n'
printf '  run_logged rustc-version rustc --version\n'
printf '  run_logged cargo-version cargo --version\n'
```

Không ghi câu lệnh dự kiến khác với argv thật. Dòng `commands.tsv` là log cục bộ có thể kiểm tra, nhưng không phải log bất biến/được ký. Reviewer cần chạy lại các lệnh chính hoặc lấy artifact từ nguồn độc lập để tăng độ tin cậy.

### 6.3 Baseline source và dirty tree

Trước edit:

- Lưu `git rev-parse HEAD`, `git status --porcelain=v2`, staged/unstaged diff dạng binary và danh sách untracked vào artifact path ngoài repo.
- Với từng source file thuộc scope đang dirty/untracked, sao chép bản nội dung trước task vào `source-before/` và SHA-256 file đó. Không copy secrets hoặc dữ liệu cá nhân không cần thiết.
- Nếu repo không dùng Git, tạo inventory file path + hash và snapshot các source liên quan trước edit.
- Hash source snapshot/decision lock trước edit vào `PRE_EDIT_SHA256SUMS`. Sau khi chụp xong toàn bộ BEFORE và hoàn tất mọi baseline command/log, tạo `PRE_EDIT_MANIFEST_COMMAND.txt` để lưu chính xác quy trình hash; rồi tạo manifest bằng một bước bookkeeping trực tiếp, **không** qua `run_logged`, vì helper sẽ sửa `commands.tsv`/`logs` sau khi hash và làm manifest tự lệch. Vòng lặp chỉ hash các file thật sự tồn tại; không gọi `sha256sum` với path giả:
  ```bash
  printf '%s\n' 'Hash pre-edit files: metadata, decision-lock, state matrix, brief, source-before, before and baseline logs' \
    > "$ARTIFACT_DIR/PRE_EDIT_MANIFEST_COMMAND.txt"
  (
    cd "$ARTIFACT_DIR" || exit 2
    for f in metadata.pre-edit.txt decision-lock.md state-matrix.md redesign-brief.pre-edit.md PRE_EDIT_MANIFEST_COMMAND.txt; do
      [ ! -f "$f" ] || sha256sum "$f"
    done
    for d in source-before before logs; do
      if [ -d "$d" ]; then
        find "$d" -type f -print0 | sort -z | xargs -0 -r sha256sum
      fi
    done
  ) > "$ARTIFACT_DIR/PRE_EDIT_SHA256SUMS"
  ```
  Sinh manifest sau mọi capture/log baseline và trước source edit. Sau đó không sửa hoặc thay thế các file BEFORE đã hash; log post-change mới có thể tiếp tục xuất hiện trong `logs/`, nhưng không được trình bày `PRE_EDIT_SHA256SUMS` như manifest của toàn bộ artifact cuối.
- Tạo `FINAL_SHA256SUMS` **sau lần final re-review, capture và mọi command logging**. Ghi command vào `FINAL_MANIFEST_COMMAND.txt` trước khi chạy, rồi thực hiện trực tiếp từ artifact directory, không qua `run_logged` và không ghi thêm log vào cây artifact sau đó (nếu cần log exit status riêng, để bên ngoài `ARTIFACT_DIR`). Lệnh loại trừ chính manifest cuối:
  ```bash
  printf '%s\n' 'find . -type f ! -name FINAL_SHA256SUMS -print0 | sort -z | xargs -0 -r sha256sum' \
    > "$ARTIFACT_DIR/FINAL_MANIFEST_COMMAND.txt"
  (
    cd "$ARTIFACT_DIR" || exit 2
    set -o pipefail
    find . -type f ! -name FINAL_SHA256SUMS -print0 | sort -z | xargs -0 -r sha256sum > FINAL_SHA256SUMS
  )
  ```
  Không sửa artifact nào sau khi finalize manifest; nếu có sửa, tạo lại manifest như bước cuối. Hash manifest tự nó không chứng minh thời điểm hay người tạo; muốn có xác nhận nguồn gốc cần external reviewer/CI attestation.
- Nếu không thể tách thay đổi mới khỏi thay đổi người dùng có sẵn, đặt origin `ORIGIN_UNKNOWN`; không dùng diff cuối để đoán tác giả.

### 6.4 Chạy command và lưu output

Dùng `run_logged <label> <command> [args...]` từ helper §6.2. Lưu cả command, cwd, UTC timestamp, argv đã escape, stdout/stderr, exit code và log path. Nếu command không được chạy, ghi `NOT_RUN` cùng lý do trong report; không tạo log giả.

Đối với command nhiều thành phần qua shell pipe/redirect, bọc rõ trong `bash -lc '...'` và ghi cả chuỗi script được truyền vào. Không đưa credential/token vào command line hoặc log.

## 7. Thứ tự workflow chuẩn

**Phase 0 — Reconnaissance và bảo vệ baseline.** Xác định repo, OS/session, dirty tree, packages, versions, source of truth, launch command có căn cứ và nơi lưu evidence. Chưa sửa source.

**Phase 1 — Task contract.** Viết draft AC, CORE/SECONDARY, task axes, release scope, authority policy, review policy, required checks, dimension set và preexisting commitments. Ghi decision lock. Chưa sửa source.

**Phase 2 — Baseline kỹ thuật + diagnostic.** Chạy các check được xác định ở §8 với command/log/exit code; xác định baseline debt. Khảo sát UI và xác định state hiện có/state mới. Không được bỏ baseline; nếu baseline check không chạy được, ghi blocker khách quan và consequence theo §8.

**Phase 3 — Design brief.** Với polish/redesign, so sánh hướng thiết kế, chọn direction theo §11. Chốt AC/state matrix/viewport constraints. Nếu criteria hoặc state matrix đổi trước source edit, cập nhật decision lock và recapture BEFORE cần thiết.

**Phase 4 — Capture BEFORE + pre-edit lock.** Chụp tất cả baseline states hiện hữu trong state matrix, đánh dấu state mới là `NEW_STATE_NO_BASELINE`, ghi điều kiện capture, chốt AC/state/brief/required checks và tạo `PRE_EDIT_SHA256SUMS`. Xong bước này mới bắt đầu sửa source.

**Phase 5 — Implementation.** Sửa trong scope đã duyệt/pre-authorize. Theo dõi change ledger. Nếu phát sinh need-for-approval thì dừng đúng phần đó.

**Phase 6 — Technical verification.** Chạy cùng bộ required checks như baseline, so sánh diagnostics và lưu evidence. Không nâng/degrade dependency để ép check xanh.

**Phase 7 — Render/input QA.** Kiểm tra screenshot/harness/desktop thật theo profile đã khóa. Kiểm tất cả states và inputs bắt buộc.

**Phase 8 — Review loop.** Tối đa hai vòng sửa. Sau mỗi vòng có source change, chụp/review lại các states ảnh hưởng và regression. Sau vòng 2 phải có final re-review; không code change thì ghi lý do có căn cứ.

**Phase 9 — Final decision.** Xếp findings, tính score theo §12, áp bảng quyết định §4, tạo final manifest và report theo §15. Không tự sửa kết quả score/gate sau khi xem mong muốn phát hành.

## 8. Required checks và baseline debt

### 8.1 Ma trận mặc định cho task Rust/egui

| Thay đổi | Baseline trước edit | Post-change bắt buộc |
|---|---|---|
| Bất kỳ Rust source nào được sửa | `cargo fmt --check`, `cargo check` package/workspace phù hợp, diagnostic baseline; xác minh test targets | Cùng target/feature/package; `cargo clippy` nếu Clippy có trong toolchain/repo policy; so sánh diagnostic set |
| Behavior/state/event/keyboard/drag/data logic đổi | Test liên quan đang có; xác định test nào tồn tại | Chạy test liên quan; nếu không có test, ghi khoảng trống và phải có manual/harness test theo AC |
| Visual component đổi và harness phù hợp tồn tại | Xác định snapshot/interaction tests phù hợp, không tự update | Chạy harness/snapshots ở version hiện có; xem diff khi mismatch |
| Visual desktop/layout/resize/state đổi | Capture baseline states hiện hữu, hoặc ghi disposition | Capture post states thật nếu E3 được yêu cầu |
| Chỉ tài liệu, không thay executable/source | Check liên quan đến nội dung; ghi `NOT_APPLICABLE` cho cargo checks với lý do | Kiểm lại file/output/doc links tương ứng |

`cargo test` là required khi project có tests liên quan hoặc thay đổi behavior; không có test tương ứng không cho phép agent bỏ kiểm tra mà không giải trình. `cargo clippy` mặc định là required check cho thay đổi Rust source. Phase 0 phải xác định component Clippy có sẵn hay không; nếu không có, không tự cài, ghi `BLOCKED_ENVIRONMENT` và release không thể PASS trừ khi user/owner cấp waiver có ghi nhận trước khi source edit. Nếu repo có baseline Clippy diagnostics, không dùng `-D warnings` như gate nhị phân; so sánh normalized diagnostic set và chỉ coi diagnostics mới trong target là regression.

### 8.2 Lệnh Cargo và lockfile an toàn

- Xác minh package bằng `Cargo.toml`, `cargo metadata` nếu an toàn, README/scripts; không đoán package name.
- Nếu `Cargo.lock` tồn tại, dùng `--locked` cho command có hỗ trợ để tránh resolution tự sửa lockfile. Lưu hash lockfile trước và sau.
- Nếu không có lockfile, đừng chạy Cargo trong working tree người dùng bằng lệnh có thể tự sinh/thay lockfile. Dùng disposable copy/workspace ngoài repo nếu có thể làm an toàn, hoặc xin approval; nếu không, đánh dấu check bị chặn. Không xóa file mới được Cargo tạo mà chưa chứng minh chắc chắn nó do chính task tạo và không thuộc người dùng.
- Đặt `CARGO_TARGET_DIR` ngoài repo nếu việc đó không làm thay đổi cách build; không xóa `target/` người dùng.

### 8.3 Phân biệt baseline warning với regression

Không so sánh chỉ `exit_code=1` trước và sau. Khi check baseline fail, chuẩn hóa và so sánh tập diagnostic trước/post, tối thiểu theo `level`, `code` (nếu có), message/diagnostic identity, package/target và file path; không dùng raw line number làm định danh duy nhất vì patch có thể dịch dòng. Với Clippy, dùng `--message-format=json` khi version hỗ trợ và lưu cả raw log lẫn kết quả chuẩn hóa.

`check_status` nhận:

- `PASS`: command chạy, exit code thành công và kết quả đáp ứng policy.
- `PASS_WITH_BASELINE_DEBT`: cùng failure/warnings đã hiện diện ở baseline, không có diagnostic mới trong target và policy cho phép ghi nhận baseline debt; phải đính kèm baseline/post logs và diff diagnostic.
- `FAIL_INTRODUCED`: lỗi/diagnostic mới có bằng chứng phát sinh từ change hoặc acceptance test fail do change.
- `FAIL_PRE_EXISTING`: lỗi hiện hữu trong baseline và vẫn fail; ảnh hưởng release theo target/AC, nhưng không được gán causality cho patch.
- `BLOCKED_ENVIRONMENT`: không thể thực hiện check do tool/OS/permission/dependency ngoài project.
- `BLOCKED_COMPARISON`: command đã chạy nhưng evidence/diagnostic hiện có không đủ để xác định baseline debt so với regression; không được coi là pass.
- `NOT_RUN`: lẽ ra chạy nhưng chưa chạy; không phải trạng thái pass.
- `WAIVED_BY_APPROVAL`: check đã được miễn bằng approval có reference, owner và timestamp ghi trước source edit theo §8.4; không dùng để che một failure đã quan sát sau edit.
- `NOT_APPLICABLE`: chỉ khi check thật sự không liên quan, có lý do khóa trước edit.

Nếu baseline/post đều fail nhưng không so sánh được diagnostic, không được tự ghi `PASS_WITH_BASELINE_DEBT`; đặt `check.post_status=BLOCKED_COMPARISON`, `change_gate=BLOCKED` và release `BLOCKED` nếu check đó là required, trừ khi có waiver hợp lệ được duyệt trước edit. Nếu Clippy không có sẵn và chưa có waiver rõ ràng, target release không thể PASS; không tự loại Clippy khỏi required checks sau khi xem kết quả. Repo có warning cũ không làm check tự động chết vĩnh viễn, nhưng cũng không được bỏ qua warning mới bằng cách chỉ nhìn exit code.

### 8.4 Check waiver — chỉ được chấp thuận trước khi biết kết quả post-change

Một waiver chỉ hợp lệ khi owner/user có quyền chấp thuận, đã ghi **trước source edit**: `check_id`, lý do không thể/không cần chạy, rủi ro được chấp nhận, phạm vi ảnh hưởng, người duyệt, approval reference và timestamp. Đặt `post_status=WAIVED_BY_APPROVAL` và lưu record tại decision lock. Waiver không xóa log hoặc failure đã quan sát; không được tạo sau khi check đã fail để đổi trạng thái. Không waive một AC/test trực tiếp chứng minh regression đã xuất hiện. Các bằng chứng desktop/input được user nêu rõ là điều kiện nghiệm thu chỉ được waive bằng sự chấp thuận minh thị của user và phải làm release caveat rõ ràng.

## 9. State matrix và capture protocol

### 9.1 Ma trận bắt buộc trước source edit

Lập `state-matrix.md` sau khi design brief/AC cuối đã xác định, nhưng trước source edit. Mỗi hàng gồm:

| Field | Ý nghĩa |
|---|---|
| `state_id` | ID ổn định như `S1` |
| `screen/component` | vùng UI/state thực tế |
| `setup_steps` | các thao tác để tới đúng state |
| `viewport_logical_points` | kích thước cửa sổ theo logical points, ví dụ `960x640` |
| `capture_pixels` | kích thước ảnh pixel được ghi nhận sau capture |
| `theme/scale/font` | điều kiện render thiết yếu |
| `fixture/data` | dữ liệu không nhạy cảm dùng để tái hiện |
| `required_visual` | yes/no |
| `required_input` | `NONE`, `CLICK`, `KEYBOARD`, `DRAG`, ... |
| `linked_ACs` | danh sách AC-ID |
| `baseline_disposition` | `BASELINE_CAPTURED`, `NEW_STATE_NO_BASELINE`, `BASELINE_UNAVAILABLE` |
| `reproduction_confidence` | high/medium/low + lý do |

Không dùng “minimum size” nếu không ghi kích thước cụ thể. `BASELINE_UNAVAILABLE` cần nguyên nhân và ảnh hưởng; không tự chuyển thành `NEW_STATE_NO_BASELINE` sau khi capture thất bại.

### 9.2 Baseline disposition

- `BASELINE_CAPTURED`: state đã tồn tại và capture được trước edit.
- `NEW_STATE_NO_BASELINE`: state được tạo lần đầu bởi task; không cần ảnh BEFORE. Đánh giá state này với expected result trong brief.
- `BASELINE_UNAVAILABLE`: state cũ tồn tại nhưng không tái tạo/capture được. Nếu state quan trọng với regression hoặc core acceptance, origin/evidence có thể `ORIGIN_UNKNOWN` và gate bị BLOCKED.

Nếu criteria/state matrix đổi trước first edit, sửa matrix và capture lại baseline hiện hữu cần thiết. Nếu đổi sau first edit, dùng change control; không tạo “BEFORE” bằng ảnh post-change rồi giả làm baseline.

### 9.3 Desktop capture/input

E3 desktop chỉ khi app thật đã launch và tất cả required visual states đã được capture trong điều kiện có thể so sánh. Với required input, phải thực hiện click/keyboard/drag/state transition thật hoặc dùng automation/harness được xác minh; screenshot không chứng minh input xảy ra.

- Ghi lệnh launch/capture, window title/app build, window size, scale, theme, fixture và timestamp.
- Mở ảnh đã lưu và xác nhận đúng cửa sổ, không rỗng, không che/crop vùng cần kiểm.
- Ảnh annotation/crop/diff là artifact phụ; giữ ảnh gốc không sửa.
- Nếu capture được ảnh nhưng input bị chặn, visual evidence có thể E3 nhưng `input_evidence_level` không được nâng lên I3; release bị chặn nếu input là required.
- Nếu không có display/window manager/permission, dùng E2/E1 nếu có; tuyệt đối không đổi nhãn thành desktop runtime.

### 9.4 Whole-app surface inventory (chỉ khi `release_scope=WHOLE_APP`)

Tạo `app-surface-inventory.md` trước khi đánh giá: liệt kê từng màn hình/top-level workspace, workflow chính, dialog/menu/popover quan trọng, state rỗng/loading/error, các viewport được hỗ trợ và đường tái hiện. Nguồn lập inventory gồm navigation/UI routes trong source, README/docs, menu và entry points đã quan sát. Ghi rõ khu vực không thể mở/không biết cách tái hiện; không tự coi inventory là toàn diện khi chỉ dựa vào một screenshot. Reviewer phải đối chiếu inventory với source/navigation và runtime. `whole_app_health=PASS` yêu cầu mọi item bắt buộc trong inventory có evidence theo state matrix và inputs cần thiết được chạy; sample một màn hình không đủ.

### 9.5 Linux/Wayland/sandbox

Kiểm tra session và biến môi trường thay vì mặc định screenshot tool hoạt động. `gnome-screenshot`, `flameshot`, compositor-specific capture hoặc desktop automation có thể không sẵn trong sandbox/Wayland. Chỉ dùng tool đã phát hiện và kiểm tra thực tế. Không cài tool/package nếu thiếu approval. Ghi lỗi raw. Nếu không có đường capture/input thay thế, báo `BLOCKED_ENVIRONMENT`; không tuyên bố E3.

## 10. Design dimensions và scoring profile

### 10.1 Dimensions

Dùng dimension IDs cố định:

1. `TASK_FIT_HIERARCHY` — thứ bậc phục vụ tác vụ chính.
2. `LAYOUT_PROPORTION` — tỷ lệ panel/work surface, bố cục và resize.
3. `TYPOGRAPHY_CONTENT` — phân cấp chữ, độ đọc, text dài và tiếng Việt.
4. `SPACING_ALIGNMENT` — nhịp khoảng cách, baseline, alignment.
5. `DESIGN_SYSTEM_CONSISTENCY` — tokens, components, icon style, states.
6. `INTERACTION_CLARITY` — state, focus/selection, affordance, feedback, keyboard/hover.
7. `ACCESSIBILITY_READABILITY` — contrast, semantics/labels, target size, focus/keyboard khi áp dụng.
8. `ROBUSTNESS_REGRESSION` — clipping, empty/error/loading, resize, regression, stability.
9. `DISTINCTIVENESS_RESTRAINT` — dấu ấn đặc trưng gắn với workflow mà không gây nhiễu.

### 10.2 Profile để tránh cherry-pick dimension

- `BUGFIX`: luôn yêu cầu `TASK_FIT_HIERARCHY` nếu thay đổi UI và `ROBUSTNESS_REGRESSION`; thêm mọi dimension liên quan tới AC. Nếu chỉ code/backend không thay UI, scoring UI có thể `NOT_APPLICABLE`.
- `VISUAL_POLISH`: yêu cầu tối thiểu `TASK_FIT_HIERARCHY`, `LAYOUT_PROPORTION`, `TYPOGRAPHY_CONTENT`, `SPACING_ALIGNMENT`, `DESIGN_SYSTEM_CONSISTENCY`, `ACCESSIBILITY_READABILITY`, `ROBUSTNESS_REGRESSION`; `INTERACTION_CLARITY` nếu component tương tác.
- `REDESIGN`: dùng đủ 9 dimensions; chỉ cho phép loại dimension khi thật sự không liên quan, có lý do trước edit và chấp thuận trong decision lock.
- `COMPONENT` là change_scope, không phải change_type; component-only redesign vẫn cần brief/AC theo `REDESIGN`.
- `REVIEW_ONLY`: không tạo change score như thể đã triển khai; dùng `review_status` riêng.

Mỗi dimension phải được đánh dấu `CORE_DIMENSION` hoặc `SECONDARY_DIMENSION` trước edit. Không cho phép S chỉ còn một dimension để nâng điểm. Mọi AC liên kết ít nhất một dimension; mọi CORE dimension liên kết ít nhất một CORE AC.

### 10.3 Anchors cho từng dimension (0–4)

- `0 — Broken/unusable`: không phục vụ tác vụ/dimension; core workflow bị phá hoặc thông tin quan trọng không thể dùng được.
- `1 — Major defects`: có thể thấy/đo lỗi nghiêm trọng; cần workaround lớn hoặc vi phạm một CORE AC.
- `2 — Adequate but uneven`: tác vụ còn làm được nhưng còn friction rõ, hierarchy/spacing/state chưa đạt một phần AC.
- `3 — Good / meets acceptance`: đạt AC liên quan, có thể còn chi tiết nhỏ không chặn workflow.
- `4 — Strong and evidenced`: mọi CORE/SECONDARY AC liên quan đạt, có tính chủ đích, ổn định qua required states và không còn defect vật chất đã xác minh trong dimension. Không đồng nghĩa “tôi thích phong cách này”.

Điểm phải có evidence path + một câu giải thích cụ thể. Mọi dimension `0` phải có finding tương ứng; dimension `1` phải có finding hoặc giải thích vì sao không tạo finding, được reviewer chấp thuận. Nếu một dimension có điểm `0` nhưng không có finding, report không hợp lệ. Điểm `0` ở dimension SECONDARY cũng không được trung bình cộng che khuất: nó tạo cap toàn cục theo §10.5.

### 10.4 Công thức

Trước edit, khóa `dimension_weights`; mặc định trọng số bằng nhau cho mọi dimension `APPLICABLE`. Rubric chọn direction ở §11 là quyết định thiết kế, không được tái sử dụng làm dimension weights hoặc phép tính release score. N/A dimension bị loại khỏi mẫu số chỉ khi N/A được duyệt/khóa hợp lệ trước edit. Không gán 0 cho dimension thiếu evidence; dùng `BLOCKED`.

`raw_mean = Σ(score_i × weight_i) / Σ(weight_i)` với các dimension có score 0–4. Ghi raw mean ít nhất 2 chữ số, số dimensions chấm được, số dimensions applicability và `coverage_pct`. Raw mean từ coverage không đầy đủ là provisional, không so ngang với score có coverage khác nhau.

`scope_quality_score` là integer 0–4 sau khi làm tròn theo bands: `[0.00,0.50)→0`, `[0.50,1.50)→1`, `[1.50,2.50)→2`, `[2.50,3.50)→3`, `[3.50,4.01)→4`, rồi áp `score_ceiling` bằng giá trị nhỏ nhất từ các gate bên dưới. Trường hợp mean không tính được thì để `N/A_BLOCKED`; không bịa mean.

### 10.5 Gate/cap điểm — áp dụng sau khi tính raw score

Tính mọi điều kiện sau, chọn `score_ceiling` thấp nhất; đây là ceiling, không phải lời khẳng định chất lượng thực tế:

| Điều kiện đã xác minh trong target | `score_ceiling` |
|---|---:|
| P0 trong target hoặc P0 INTRODUCED bởi change | 0 |
| P1 trong target hoặc P1 INTRODUCED bởi change | 1 |
| CORE AC `FAIL` | 2 |
| Bất kỳ dimension áp dụng nào score `0` | 1 (trừ khi tất cả dimension áp dụng đều `0`, khi đó 0) |
| Bất kỳ dimension áp dụng nào score `1` | 2 |
| required check `FAIL_INTRODUCED` nếu không có cap nghiêm trọng hơn | 2 |
| P2 defect chưa `FIXED` trong target (`OPEN` hoặc `ACCEPTED_DEFERRED`) | 3 |
| `reviewer_mode=SELF_REVIEW` hoặc `SEPARATE_CONTEXT_REVIEW` | 3 |

TRADEOFF không nhận severity P0–P3 và không tự trigger cap. P0/P1 PRE_EXISTING ngoài `TASK_CHANGE` không cap task score trừ khi finding ID nằm trong `preexisting_commitment_ids`; chúng vẫn được báo ở report và `whole_app_health`. Mọi P0/P1 đã xác minh của mọi origin phải xuất hiện trong báo cáo findings; chỉ P0/P1 thuộc release target quyết định `release_status`.

Áp dụng caps **trước** quyết định có thể tuyên bố điểm đủ evidence. Như vậy, nếu đã xác minh P1 nhưng một dimension khác thiếu evidence, lỗi P1 không biến mất: tính score từ dimension quan sát được nếu có, áp ceiling 1 và đánh dấu coverage/score basis chưa đầy đủ. Nếu không có dimension nào chấm được, dùng `scope_quality_score=N/A_BLOCKED`, `score_ceiling=1`, `confirmed_gate=P1`, không phát minh numeric score.

### 10.6 Khi nào score bị BLOCKED

- Nếu có hard failure đã xác minh, phải báo failure và `score_ceiling` ngay cả khi dimension khác BLOCKED; không dùng N/A để che failure.
- Nếu không có hard failure nhưng CORE dimension BLOCKED/NOT_VERIFIED, hoặc `ORIGIN_UNKNOWN` có thể làm thay đổi kết quả CORE AC/P0/P1 trong target, `scope_quality_score=N/A_BLOCKED`.
- Nếu tất cả CORE dimension được chấm và coverage >=80%, score có thể hiển thị provisional khi SECONDARY dimension bị chặn; phải ghi coverage và không so với run có profile khác.
- Nếu coverage <80% và chưa có hard failure, `scope_quality_score=N/A_BLOCKED`.
- 4/4 không thể đạt ở self-review; cũng không thể đạt nếu còn CORE dimension <3, finding P0/P1 chưa giải quyết, P2 OPEN trong target, core evidence thiếu hoặc release brief/authority không hợp lệ.

## 11. Design direction: gu thẩm mỹ có căn cứ

### 11.1 Không chỉ là danh sách cấm

Không mặc định dark/light, gradient, shadow, radius, dense/airy hoặc card là tốt/xấu. Chất lượng đến từ cấu trúc hierarchy, tỷ lệ panel, rhythm, typography, icon discipline, feedback, work surface và dấu ấn gắn với tác vụ SoWaCap. Tránh component/card/hero trang trí chỉ khi chúng không làm rõ công việc.

### 11.2 Rubric chọn direction

Cho `VISUAL_POLISH` lớn hoặc `REDESIGN`, so sánh 2–3 phương án khả thi bằng cùng rubric. Không so một phương án kỹ với một phương án sơ sài.

| Criterion | Weight |
|---|---:|
| Fit với workflow/tác vụ chính | 30% |
| Information hierarchy | 20% |
| Integration với design system hiện có | 20% |
| Distinctiveness with restraint | 15% |
| Readability/accessibility | 10% |
| Implementation/regression risk | 5% |

`direction_score = Σ(rating_i × weight_percent_i)/100`, mỗi rating 1–5. Neo điểm: 1 = xung đột constraint/không phục vụ task; 3 = đáp ứng brief nhưng chưa có lợi thế rõ; 5 = lợi thế cụ thể gắn workflow/evidence, không vi phạm invariant; 2/4 là trung gian có lý do. Ở criterion risk, điểm cao nghĩa là rủi ro thấp hơn.

Đánh dấu từng rating trước render là `PROPOSED` hoặc `INFERRED`, không phải `OBSERVED`. Nếu phương án được render thành prototype, cập nhật rating sau render chỉ bằng decision-log entry có lý do. Điểm chỉ hơn kém rất nhỏ có thể là noise do ước lượng: nếu chênh lệch weighted score `<=0.25` điểm trên thang 1–5, ghi `NEAR_TIE`, so trade-off theo fit → regression risk → readability và nêu vì sao chọn. Không chỉnh rating hậu nghiệm để hợp thức hóa lựa chọn.

### 11.3 Design brief

`docs/ui/redesign-brief.md` nên có:

- Product/job-to-be-done và constraints quan sát được.
- Giữ lại các yếu tố đã hoạt động tốt.
- Art direction gồm thuộc tính cụ thể + ảnh/reference nếu có quyền dùng.
- Hierarchy: primary work surface, primary/secondary actions, status/error feedback.
- Token/component source of truth và ngoại lệ dự kiến.
- Invariants, AC IDs, state matrix, viewport constraints, dependencies/risks.
- Các phương án đã so, direction được chọn, score/rationale và nhãn evidence.
- `brief_status`/authority per change item và decision lock hash.

## 12. Accessibility, typography, geometry

Các số dưới đây là mục tiêu nội bộ/heuristic; không tuyên bố chứng nhận WCAG cho toàn bộ desktop-native UI.

- **Contrast:** tham chiếu WCAG 2.2 AA 4.5:1 cho text thường, 3:1 cho text lớn và non-text UI elements khi success criterion tương ứng áp dụng. Ghi màu foreground/background và phương pháp đo (design tokens/calculation hoặc sampled rendered pixels). Nếu chưa đo, `contrast_status=NOT_MEASURED`; không nói “đạt 4.5:1”. Không cần cài tool mới khi chưa được phép; phép tính có thể dùng code/tool đã sẵn có nếu lưu input/output.
- **Typography:** 13–15 egui logical points cho body/11–12 cho metadata chỉ là starting heuristic, không phải minimum universal. Không ép widget mặc định đổi font-size chỉ vì nằm dưới heuristic; ghi ngoại lệ dựa trên density, DPI, font và khả năng đọc. Kiểm tra tiếng Việt/dấu, fallback, truncation, text dài và scale thực tế.
- **Hit target:** 28×28 egui logical points là mục tiêu thoải mái cho control thường xuyên; 24×24 là mục tiêu compact tham khảo chứ không phải CSS pixel hay yêu cầu WCAG phổ quát. Ghi geometry đã đo và ngoại lệ; kiểm tra có cách thao tác khác với control nhỏ/quan trọng.
- **Window/layout:** ghi minimum supported size bằng egui logical points và capture pixels thực tế. Không clipping/overlap ngoài clipping chủ đích được nêu trong brief.
- **Semantics/keyboard:** screenshot không chứng minh screen-reader semantics hoặc keyboard usability. Dùng egui accessibility semantics/`egui_kittest`/test thao tác khi version project hỗ trợ; ghi đúng cái đã test. Với custom-painted widget, xem xét widget info/label/role nếu project API hỗ trợ.
- **NOT_MEASURED:** không tự biến thành PASS. Nếu dimension là CORE và không có alternative evidence, gate bị BLOCKED. Nếu chỉ SECONDARY, score có thể provisional theo §10.6.

## 13. E1/E2/E3 và input evidence

Giữ riêng `visual_evidence_level` và `input_evidence_level`; không dùng một screenshot làm bằng chứng cho mọi input.

### 13.1 Visual level

- `E0_NONE`: không có evidence sử dụng được.
- `E1_SOURCE_ONLY`: source/log/build/unit test; không có render evidence.
- `E2_COMPONENT_HARNESS`: harness egui/widget/snapshot tương thích với version project; có thể query semantics, synthetic input và render snapshots của UI harness.
- `E3_DESKTOP_CAPTURE`: app desktop thật chạy và đã capture tất cả required visual states theo state matrix.

### 13.2 Input level

- `I0_NONE`: không có bằng chứng input.
- `I1_SOURCE`: source/code path được review, không đồng nghĩa thao tác runtime.
- `I2_HARNESS`: thao tác synthetic trong harness hoặc test integration tương ứng.
- `I3_DESKTOP_RUNTIME`: input được thực hiện/xác minh trên app desktop thật trong required states.

Nếu task không yêu cầu input, khai `required_input_profile=I0_NONE` trước edit. Nếu AC yêu cầu click/keyboard/drag mà input bị chặn, `required_input` phải `BLOCKED`; E3 visual không thay thế I3.

### 13.3 `egui_kittest`

Có thể dùng khi project version/feature tương thích; API, version và features phải kiểm tra tại thời điểm dùng từ `Cargo.toml`/lockfile/docs upstream. Không nhúng version “latest” cố định trong skill, không giả định các crate phát hành cùng version/minor và không thêm dependency nếu chưa có requirement, tương thích được xác minh và authority hợp lệ. Không tương thích hoặc không được phép thêm dependency thì dùng harness sẵn có hoặc giảm evidence level trung thực.

Snapshot mismatch phải mở `.new`/diff và lý giải; update snapshot chỉ sau khi review thay đổi là đúng mong muốn và được phép. Test chuyển xanh sau update snapshot không tự chứng minh design đúng.

## 14. Findings, severity, prioritization

### 14.1 Category và kind hợp lệ

`category`: `visual-defect | behavior | accessibility | consistency | regression | performance | aesthetic-tradeoff | other`.

`kind`: `DEFECT | TRADEOFF`.

- `DEFECT` bắt buộc `severity=P0|P1|P2|P3`, `origin`, `evidence_kind`, scope, reproduction/evidence.
- `TRADEOFF` bắt buộc `severity=NONE`, có mô tả hai lựa chọn, impact, `disposition=ACCEPTED_BY_USER|REJECTED|UNDECIDED` và nguồn quyết định. Không tự biến preference thành bug. Phải ghi `requires_resolution_before_release=true|false` theo brief đã khóa; `UNDECIDED` chỉ chặn release khi field này là `true`. Không tự đổi field sau khi thấy kết quả để mở gate.

### 14.2 Severity anchors

- `P0`: app không launch/use được trong supported environment do nguyên nhân code/config/package đã xác minh; dữ liệu bị mất/hỏng; primary workflow hoàn toàn unusable không có workaround.
- `P1`: CORE workflow bị chặn nghiêm trọng, regression diện rộng hoặc lỗi nghiêm trọng khiến release target không an toàn/không thể dùng.
- `P2`: workflow vẫn dùng được nhưng có friction đáng kể, lỗi rõ hoặc workaround.
- `P3`: lỗi nhỏ không chặn workflow.

Severity là mức tác động, origin là nguồn gốc và scope là phạm vi. Không dùng “tôi cho rằng nhỏ” thay evidence. Nếu chưa biết origin thì ghi `ORIGIN_UNKNOWN`, không hạ severity chỉ vì nguyên nhân chưa biết.

### 14.3 Xếp hạng Top 6

Top 6 là executive summary tối đa, không phải quota. Tất cả P0/P1, unresolved release blockers và CORE AC failures đều phải xuất hiện ở summary hoặc blocker section, kể cả vượt sáu finding.

Sắp xếp theo thứ tự lexicographic (chỉ xét tiêu chí sau khi tiêu chí trước hòa):

1. `DEFECT` trước `TRADEOFF`.
2. Với defects: P0 → P1 → P2 → P3.
3. Ảnh hưởng CORE AC trước non-core.
4. Origin `INTRODUCED` → `ORIGIN_UNKNOWN` → `PRE_EXISTING` → `NOT_APPLICABLE`.
5. Số required state/workflow bị ảnh hưởng giảm dần.
6. Confidence `HIGH` → `MEDIUM` → `LOW`.
7. Finding ID tăng dần làm tie-break cuối.

Mỗi finding: `ID`, `kind`, category, severity/`NONE`, evidence_kind, origin, target scope, linked AC/dimension/state, evidence path, reproduction, Before (observable facts), After (specific fix), Why, acceptance test, confidence, status/disposition. Backlog ngoài Top 6 giữ nguyên các field bắt buộc; không chỉ ghi mỗi title/severity.

## 15. Scoring/release decision output schema

Báo cáo cuối dùng đúng field/enum dưới đây. Không thêm biến thể tự chế cho machine-readable status.

```yaml
# TEMPLATE ONLY: fill every required field from this task's evidence.
# Null means "not populated", not a valid completed outcome.
identity:
  task_id: null
  task_mode: null # IMPLEMENT | REVIEW_ONLY
  change_type: null # BUGFIX | VISUAL_POLISH | REDESIGN | COMPONENT_BEHAVIOR | NOT_APPLICABLE
  change_scope: null # COMPONENT | SCREEN | APP | NOT_APPLICABLE
  release_scope: null # TASK_CHANGE | WHOLE_APP
  supported_environment: null
  authority_policy: null # LOCAL_VISUAL_PREAUTHORIZED | APPROVAL_REQUIRED
  review_policy: null # INDEPENDENT_REQUIRED | SELF_REVIEW_ALLOWED
  reviewer_mode: null # SELF_REVIEW | SEPARATE_CONTEXT_REVIEW | INDEPENDENT_REVIEW
  reviewer_id: null
  reviewer_session_ref: null
  reviewer_reproduced: null
  review_status: null # FINDINGS | NO_CONFIRMED_FINDINGS | BLOCKED; required for REVIEW_ONLY
  brief_status: null # NOT_REQUIRED | PROPOSED | AUTO_ADOPTED_BY_PREDECLARED_POLICY | USER_APPROVED | NEEDS_APPROVAL
  decision_lock_path: null
  pre_edit_sha256_path: null

outcome:
  task_status: null # PASS | PARTIAL | FAIL | BLOCKED | NOT_APPLICABLE
  change_gate: null # PASS | FAIL | BLOCKED | NOT_APPLICABLE
  release_status: null # PASS | PASS_NON_INDEPENDENT_REVIEW | PENDING_REVIEW | FAIL | BLOCKED | NOT_EVALUATED
  whole_app_health: null # PASS | KNOWN_P0_P1 | UNKNOWN | NOT_EVALUATED
  release_blockers: null
  evidence_integrity: null # PASS | FAIL | BLOCKED
  evidence_integrity_mode: null # AUTHOR_SELF_CHECK | INDEPENDENT_REVIEWER_CHECK | CI_ATTESTED
  evidence_integrity_evidence_refs: null
  evidence_integrity_checked_by: null
  preexisting_commitment_ids: null

progress_items:
  - id: PI-01
    description: null
    baseline_status: null # COMPLETE | INCOMPLETE | NOT_VERIFIED | NOT_APPLICABLE
    status: null # VERIFIED_COMPLETE | INCOMPLETE | BLOCKED
    evidence_refs: null

change_items:
  - change_id: CH-01
    description: null
    scope: null
    implementation_status: null # PLANNED | IMPLEMENTED | NOT_IMPLEMENTED | REVERTED_BY_EXPLICIT_CHANGE_CONTROL
    authority_status: null # PREAUTHORIZED | USER_APPROVED | NEEDS_APPROVAL | UNAUTHORIZED_IMPLEMENTED | NOT_APPLICABLE
    brief_status: null # NOT_REQUIRED | PROPOSED | AUTO_ADOPTED_BY_PREDECLARED_POLICY | USER_APPROVED | NEEDS_APPROVAL
    approval_reference: null
    recorded_at_utc: null

acceptance_criteria:
  - id: AC-01
    class: null # CORE | SECONDARY
    statement: null
    baseline_status: null # PASS | FAIL | NOT_VERIFIED | NOT_APPLICABLE
    status: null # PASS | FAIL | NOT_VERIFIED | NOT_IMPLEMENTED_PENDING_APPROVAL | NOT_APPLICABLE
    evidence_kind: null # OBSERVED | INFERRED | PROPOSED | NOT_VERIFIED
    evidence_refs: null
    linked_dimensions: null
    linked_states: null

checks:
  - id: TC-01
    command_argv: null
    baseline_status: null # PASS | PASS_WITH_BASELINE_DEBT | FAIL_INTRODUCED | FAIL_PRE_EXISTING | BLOCKED_ENVIRONMENT | BLOCKED_COMPARISON | NOT_RUN | WAIVED_BY_APPROVAL | NOT_APPLICABLE
    post_status: null # PASS | PASS_WITH_BASELINE_DEBT | FAIL_INTRODUCED | FAIL_PRE_EXISTING | BLOCKED_ENVIRONMENT | BLOCKED_COMPARISON | NOT_RUN | WAIVED_BY_APPROVAL | NOT_APPLICABLE
    exit_code: null
    log_path: null
    diagnostic_diff_path: null
    waiver_status: null # NONE | APPROVED_BEFORE_EDIT
    waiver_approval_reference: null
    waiver_approved_by: null
    waiver_recorded_at_utc: null
    evidence_kind: null # OBSERVED | INFERRED | PROPOSED | NOT_VERIFIED

visual_and_input:
  required_visual_profile: null # E1_SOURCE_ONLY | E2_COMPONENT_HARNESS | E3_DESKTOP_RUNTIME
  visual_evidence_level: null # E0_NONE | E1_SOURCE_ONLY | E2_COMPONENT_HARNESS | E3_DESKTOP_CAPTURE
  required_input_profile: null # I0_NONE | I1_SOURCE | I2_HARNESS | I3_DESKTOP_RUNTIME
  input_evidence_level: null # I0_NONE | I1_SOURCE | I2_HARNESS | I3_DESKTOP_RUNTIME
  desktop_visual: null # PASS | FAIL | BLOCKED | NOT_REQUIRED
  required_input: null # PASS | FAIL | BLOCKED | NOT_REQUIRED
  state_matrix_path: null
  before_paths: null
  after_paths: null
  image_diff_paths: null
  capture_logs: null
  baseline_dispositions: null

score:
  raw_mean: null
  rated_dimensions: null
  applicable_dimensions: null
  coverage_pct: null
  score_basis: null # COMPLETE | PARTIAL_COVERAGE | LIMITED_EVIDENCE | BLOCKED
  score_ceiling: null # 0 | 1 | 2 | 3 | 4; null only if no score gate can be determined
  scope_quality_score: null # 0 | 1 | 2 | 3 | 4 | N/A_BLOCKED | NOT_APPLICABLE
  dimension_results:
    - id: TASK_FIT_HIERARCHY
      weight: 1 # positive numeric weight locked before edit; all applicable weights determine raw_mean denominator
      class: null # CORE_DIMENSION | SECONDARY_DIMENSION
      status: null # SCORED | BLOCKED | NOT_APPLICABLE
      score: null # 0-4 only when status=SCORED
      evidence_kind: null # OBSERVED | INFERRED | PROPOSED | NOT_VERIFIED
      evidence_refs: null
      rationale: null

findings:
  - id: null
    kind: null # DEFECT | TRADEOFF
    category: null # visual-defect | behavior | accessibility | consistency | regression | performance | aesthetic-tradeoff | other
    severity: null # P0 | P1 | P2 | P3 for DEFECT; NONE for TRADEOFF
    evidence_kind: null # OBSERVED | INFERRED | PROPOSED | NOT_VERIFIED
    origin: null # INTRODUCED | PRE_EXISTING | ORIGIN_UNKNOWN | NOT_APPLICABLE
    target_scope: null
    linked_ac: null
    linked_dimension: null
    state_id: null
    evidence_refs: null
    reproduction: null
    before: null
    after: null
    why: null
    acceptance_test: null
    confidence: null # HIGH | MEDIUM | LOW
    disposition: null # DEFECT: OPEN | FIXED | ACCEPTED_DEFERRED | REJECTED | UNDECIDED; TRADEOFF: ACCEPTED_BY_USER | REJECTED | UNDECIDED
    requires_resolution_before_release: null # true | false; required for TRADEOFF, locked in brief before edit
    risk_owner: null
    disposition_approval_reference: null
    disposition_recorded_at_utc: null

artifacts:
  artifact_dir: null
  commands_tsv: null
  pre_edit_sha256: null
  final_sha256: null
  provenance_limitations: null
```

Báo cáo cuối không hợp lệ khi còn `null` ở field bắt buộc; điền list rỗng chỉ sau khi xác minh rằng danh sách thực sự rỗng. Xóa object placeholder trong `findings` nếu review đầy đủ không thấy finding nào và đặt `findings: []`. Template trên không mô tả kết quả của repo SoWaCap và không phải bằng chứng đã chạy task.

### 15.1 Ràng buộc schema cross-field

- `ac.status=PASS` yêu cầu evidence refs hợp lệ và `evidence_kind` là `OBSERVED` hoặc `INFERRED`; `PROPOSED`/`NOT_VERIFIED` không thể chứng minh PASS.
- `ac.status=NOT_VERIFIED` không được báo là PASS trong task/release decision. `NOT_APPLICABLE` phải có rationale và approval/decision-lock reference.
- `check.post_status=PASS` cần log + exit code phù hợp; `PASS_WITH_BASELINE_DEBT` cần baseline/post log và normalized diagnostic diff; `NOT_RUN` không có exit code giả. `NOT_APPLICABLE` cần rationale khóa trước edit; `BLOCKED_ENVIRONMENT` là không thực thi được vì môi trường; `BLOCKED_COMPARISON` là đã chạy nhưng không thể so sánh chẩn đoán; hai trạng thái này không thay thế nhau.
- Finding `origin=INTRODUCED` cần baseline tương đương và bằng chứng failure mới; `PRE_EXISTING` cần baseline proof; nếu thiếu, dùng `ORIGIN_UNKNOWN`.
- `disposition=ACCEPTED_DEFERRED` chỉ cho DEFECT P2 và cần `risk_owner`, `disposition_approval_reference` và `disposition_recorded_at_utc`. Thiếu bất kỳ field nào thì coi như chưa được accept: P2 vẫn OPEN và release bị BLOCKED. `ACCEPTED_BY_USER` chỉ hợp lệ cho `kind=TRADEOFF`; không dùng nó để miễn severity/cap của DEFECT.
- `reviewer_mode=INDEPENDENT_REVIEW` yêu cầu `reviewer_id`, `reviewer_session_ref`, `reviewer_reproduced` có item cụ thể và reviewer evidence/log paths. Nếu chỉ đọc report của author hoặc chỉ đổi chat, dùng `SEPARATE_CONTEXT_REVIEW`/`SELF_REVIEW`. Với release PASS, evidence integrity phải có independent/CI provenance như rule §18; `AUTHOR_SELF_CHECK` không đủ.
- `score.dimension_results` phải liệt kê mọi dimension trong `applicable_dimensions` đúng một lần, có `weight > 0` đã khóa trước edit; raw_mean phải tính lại từ chính các weight/score được báo cáo trên những dimension thực sự có score. Tổng weight được tính trên tập dimension scored và phải dương. Dimension BLOCKED không được xem là score 0; dimension NOT_APPLICABLE chỉ được loại khỏi denominator nếu N/A được duyệt/khóa hợp lệ. Mặc định mọi applicable dimension có weight bằng nhau. Không tự đổi weight sau khi xem score.
- `check.post_status=WAIVED_BY_APPROVAL` yêu cầu `waiver_status=APPROVED_BEFORE_EDIT`, approval reference, approver và timestamp trước edit. Ngược lại, `waiver_status=APPROVED_BEFORE_EDIT` phải trỏ tới quyết định có trước trong decision lock. `BLOCKED_COMPARISON` không được coi là PASS.
- `release_status=PASS` yêu cầu all release gates PASS; `PASS_NON_INDEPENDENT_REVIEW` chỉ hợp lệ theo policy §4.4; không được sửa enum để tạo trạng thái thuận lợi hơn.

Các value trong YAML chỉ là ví dụ schema, không phải kết quả thật; phải thay bằng dữ liệu của task. Không sao chép `PASS` từ template khi chưa chạy.

### 15.2 Review-only schema

Với `task_mode=REVIEW_ONLY`, không sửa source, không bịa task/change/release outcome:

- `task_status=NOT_APPLICABLE`
- `change_gate=NOT_APPLICABLE`
- `release_status=NOT_EVALUATED`
- thêm `review_status=FINDINGS|NO_CONFIRMED_FINDINGS|BLOCKED`; chỉ chấm score cho artifact được cung cấp, ghi evidence scope/coverage; không tuyên bố app runtime pass nếu không có E3.

## 16. Review loop và handoff

### 16.1 Hai vòng sửa

Review ban đầu không tính là fix round.

- **Fix round 1:** sửa theo lô các P0/P1 liên quan target, CORE AC failures và P2 có thể xử lý an toàn trong scope.
- **Re-review 1:** sau source change, chạy checks liên quan, recapture các state ảnh hưởng, kiểm finding cũ và regression mới.
- **Fix round 2:** xử lý finding còn lại thuộc scope và regression quan trọng.
- **Final re-review:** bắt buộc sau source change cuối cùng, kể cả thay đổi ở vòng 2. Nếu không có code change sau re-review trước đó, ghi lý do và evidence path.
- Hết hai vòng không tự chạy vòng ba. Báo finding còn lại, gate/status, risk và đề xuất task tiếp theo. Người dùng có thể yêu cầu task mới/vòng tiếp theo; đó không phải lý do sửa âm thầm vòng 3.

### 16.2 Handoff

**Director → Engineer:** scope, AC/CORE, authority ledger, brief/status, alternatives/rationale, invariants, dimension profile, state matrix, baseline paths, approval-required items.

**Engineer → QA:** source diff, change ledger, exact argv/logs/exit code, launch/capture commands, screenshot/source snapshots, pre/post hashes, diagnostic diff, limitation.

**QA → Engineer:** findings xếp theo §14.3, evidence/reproduction, AC/dimension mapping, gate impact. Reviewer không sửa code ở lượt đầu.

Nếu một agent đảm nhiệm nhiều vai trò, ghi self-review rõ ràng; không gọi Independent QA. Reviewer tiếp nhận artifact không có nghĩa artifact đó đã được xác minh.

## 17. References và giới hạn kế thừa

Các nguồn dưới đây dùng làm inspiration/reference, không có tuyên bố skill SoWaCap là bản kế thừa chính thức. README/docs khả dụng công khai được đối chiếu ngày 2026-10-12; một số trang GitHub không tải trực tiếp được và chỉ có thể kiểm tra qua nội dung được lập chỉ mục. URL theo nhánh `main`/`master` là mutable, không pin commit và cần kiểm tra lại nội dung/version trước mỗi tích hợp. Không dùng con số “latest” được lưu cố định trong skill.

### Design và engineering

- Ouroboros UI — https://github.com/Type-zero-labs/ouroboros-ui — token-first Rust/egui design system; API upstream được mô tả là pre-1.0/evolving, cần kiểm tra version của repo SoWaCap trước khi tích hợp.
- Addy Osmani — frontend-ui-engineering — https://github.com/addyosmani/agent-skills/tree/main/skills/frontend-ui-engineering — tham khảo engineering/component practices; nhiều ví dụ web cần chuyển đổi thận trọng sang egui.
- Anthropic — frontend-design — https://github.com/anthropics/skills/tree/main/skills/frontend-design — tham khảo art direction; không bê nguyên web layout vào desktop-native UI.

### Review và QA

- ytvee-dev — frontend-visual-qa — https://github.com/ytvee-dev/webdev-agent-kit/tree/main/skills/frontend-visual-qa — tham khảo rendered UI evidence/visual regression; cần thay browser-only steps bằng egui harness/desktop runtime tùy profile.
- tomwangowa — ui-design-analyzer — https://github.com/tomwangowa/agent-skills/tree/main/ui-design-analyzer — tham khảo screenshot-based critique.
- Humbleteam — design-review — https://github.com/humbleteam/design-review — README mô tả score 0–4, tối đa sáu issue và Before/After/Why; README cũng có quy tắc riêng về polish items. SoWaCap không ép tạo issue khi không có lỗi đã xác minh.

### egui testing / accessibility

- egui_kittest README — https://github.com/emilk/egui/blob/main/crates/egui_kittest/README.md — mô tả semantic query, synthetic input và snapshot testing; xác minh version/features khi dùng.
- egui accessibility guidance — https://github.com/emilk/egui/blob/main/docs/accessibility.md — tham khảo widget information/semantics và test support.
- egui architecture — https://github.com/emilk/egui/blob/main/ARCHITECTURE.md — tham khảo vai trò của egui/eframe/renderers.
- WCAG 2.2 — https://www.w3.org/TR/WCAG22/ — tham chiếu contrast criteria khi tiêu chí áp dụng; egui logical points không được quy đổi thẳng thành CSS px.

## 18. Evidence integrity — kiểm tra tính nhất quán của bằng chứng

`evidence_integrity` nhận đúng một giá trị; phải ghi cả `evidence_integrity_mode` và evidence refs:

- `PASS`: checker đã đối chiếu artifact cần thiết, hash tái tính với manifest được chỉ rõ, report với raw logs/exit code/ảnh/state matrix; không phát hiện mismatch. Checker có thể là tác giả, reviewer độc lập hoặc CI, nhưng chế độ kiểm tra phải được khai báo rõ.
- `FAIL`: có mâu thuẫn đã xác minh, ví dụ log ghi exit code khác report, hash không khớp, ảnh BEFORE bị thay bằng ảnh post-change, hoặc report ghi check PASS nhưng log chứng minh command fail. Không suy đoán động cơ; chỉ ghi mismatch quan sát được.
- `BLOCKED`: artifact/manifest/log cần thiết không được cung cấp hoặc không thể truy cập để kiểm tra, nhưng chưa có bằng chứng đủ để kết luận có mismatch.

`evidence_integrity_mode` là `AUTHOR_SELF_CHECK | INDEPENDENT_REVIEWER_CHECK | CI_ATTESTED`. `AUTHOR_SELF_CHECK` chỉ khẳng định tính nhất quán nội bộ theo khả năng người tạo tự kiểm tra; nó **không** thỏa mãn yêu cầu independent review và không mở khóa `release_status=PASS`. Với `review_policy=INDEPENDENT_REQUIRED`, integrity self-check có thể cho phép trạng thái `PENDING_REVIEW` trong khi chờ reviewer, miễn các gate khác đã đạt. `INDEPENDENT_REVIEWER_CHECK` cần reviewer ID/session và log/evidence refs độc lập. `CI_ATTESTED` cần link/run ID và artifact từ CI thực tế; không ghi nếu không có pipeline.

`FAIL` chặn release; `BLOCKED` cũng không cho release PASS. Local SHA-256 manifest do cùng author tạo không tự chứng minh người tạo hoặc chronology. Tái hash và so sánh log có thể xác minh consistency nhưng không xác thực identity/time. Nếu reviewer chỉ tin báo cáo của tác giả, ghi provenance limitation và không gán `INDEPENDENT_REVIEW`.

## 19. Final checklist

- [ ] `task_mode`, `change_type`, `change_scope`, `release_scope`, `authority_policy`, `review_policy`, `required checks`, dimension profile và required evidence đã khóa trước edit.
- [ ] AC có ID, CORE/SECONDARY, trạng thái có enum, evidence kind/reference và mapping dimension/state.
- [ ] Approval chờ chưa bị triển khai; change vượt quyền đã bị ghi `UNAUTHORIZED_IMPLEMENTED`.
- [ ] Artifact nằm ngoài working tree hoặc status được chụp trước khi tạo artifact trong repo.
- [ ] Dirty tree, staged/unstaged diff, untracked list và source baseline liên quan đã được bảo vệ.
- [ ] Technical baseline đã chạy hoặc có lý do cụ thể; baseline debt không được so bằng exit code đơn thuần.
- [ ] `Cargo.lock`/target/user files không bị sửa/xóa âm thầm.
- [ ] Brief/state matrix/window size/input requirements hoàn tất trước first edit; state mới và baseline unavailable phân biệt đúng.
- [ ] Logs chứa argv thực, cwd, timestamp, output, exit code và path; hashes có manifest, provenance limit được khai báo.
- [ ] Mỗi visual/input state được kiểm ở evidence level thật; screenshot không thay thế input test.
- [ ] Mỗi dimension có status/score/evidence; điểm 0/1 liên kết finding; caps áp sau raw mean và không bị BLOCKED khác che khuất.
- [ ] Task/change/release/whole-app outcomes được quyết định theo thứ tự §4; SELF_REVIEW không bị gắn nhãn independent.
- [ ] P0/P1 blockers đều báo, kể cả ngoài Top 6; backlog có đủ finding fields.
- [ ] Re-review sau lần source change cuối; có final manifest/report hoặc ghi rõ lý do bị chặn.

**Giới hạn tính tất định:** với cùng một tập AC, finding, origin, check và authority labels, các bảng gate ở §4/§10 cho cùng kết quả. Việc chọn art direction, gán severity/origin, xác định “đáng kể” và chấm dimension vẫn có phần judgment; Markdown không biến phán đoán thành sự thật máy móc. Những nhãn ấy cần rationale, evidence và review độc lập.

**Nguyên tắc kết thúc:** một skill Markdown không chứng minh chính nó hoàn hảo. Độ tin cậy phải đến từ quy tắc có thể tái hiện, artifact kiểm tra được, reviewer không phụ thuộc tác giả và việc nói rõ những gì vẫn chưa được xác minh.
