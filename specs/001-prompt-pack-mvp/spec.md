# Feature Specification: Prompt Pack MVP (lát L0 + L1)

**Feature Branch**: `001-prompt-pack-mvp`

**Created**: 2026-11-08

**Status**: Draft

**Input**: User description: "Dựa vào docs/Brand Image Prompt Studio — Mô tả tính năng & Đặc tả build.md lấy thông tin và tạo spec"

**Phạm vi spec này**: lát L0 (nền dữ liệu, phân quyền, đăng nhập bằng lời mời) và lát L1 (luồng Creator
C1–C6 end-to-end trên một dự án thật). Các lát L2–L4 sẽ có spec riêng. Tài liệu nguồn:
`docs/Brand Image Prompt Studio — Mô tả tính năng & Đặc tả build.md` (mục 1–12).

## Clarifications

### Session 2026-11-08

- Q: Admin được làm gì với phiên làm việc? → A: Admin tạo được phiên (để test) ở mọi dự án, không cần
  là thành viên; với phiên của người khác chỉ được xem, không sửa, không xoá.
- Q: Trong L1, Creator xử lý lỗi kiểm tra trên thẻ prompt bằng cách nào? → A: Cả hai: sửa cấu hình chỗ
  ảnh rồi tạo lại cả pack, hoặc sửa tay nội dung prompt trên thẻ và được kiểm tra lại ngay; prompt sửa
  tay được đánh dấu.
- Q: Dự án thử nghiệm của L1 là gì, và có cần bảng màu/font/logo không? → A: Print Your Wear. App chỉ tạo
  prompt ảnh nên không cần nhiều về font và màu; chỉ cần nắm phong cách hình ảnh của brand. Bảng màu,
  font, logo là tuỳ chọn.
- Q: Phong cách hình ảnh của dự án được tạo ra theo cách nào? → A: Mỗi danh mục blog của mỗi dự án có một
  file Markdown tham chiếu (Style Guide danh mục) do Admin viết; mẫu tại `docs/style-guides/_template.md`.
- Q: Sau khi đã tạo prompt, nếu Creator sửa outline hoặc bấm "Phân tích" lại thì xử lý thế nào? → A: Cho
  phân tích lại trong cùng phiên, có cảnh báo chỗ ảnh và prompt hiện có (kể cả sửa tay) sẽ bị thay; vẫn dùng
  bản chụp preset của lần phân tích đầu tiên.
- Q: Giới hạn lượt gọi AI mỗi ngày của Creator tính theo đơn vị nào? → A: Không giới hạn theo Creator; giữ
  giới hạn tổng mỗi ngày của nhà cung cấp AI làm chốt chặn chi phí.
- Q: Trong L1, app có lưu file thật cho các tham chiếu (logo, sản phẩm, design, người mẫu) không? → A: Không.
  App chỉ phân tích văn bản (Style Guide danh mục và nội dung bài) để tạo Prompt Pack; mỗi tham chiếu chỉ có
  tên và mô tả, Creator tự gắn file khi tạo ảnh.
- Q: Ai viết nội dung cuối cùng của từng prompt? → A: AI viết toàn bộ phần mô tả của prompt dựa trên Style
  Guide danh mục và nội dung bài; code chỉ gắn thêm phần bắt buộc (không nên, negative, placeholder, tỉ lệ
  khung) và chạy bộ kiểm tra.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Sinh Prompt Pack đúng brand từ keyword và outline (Priority: P1)

Creator đăng nhập, chọn một dự án được cấp và một danh mục, nhập keyword, dán outline markdown
(H2/H3) và ghi chú tuỳ chọn, rồi bấm "Phân tích". Hệ thống đề xuất danh sách chỗ ảnh (gắn với heading,
mục đích ảnh, số ảnh, lý do). Creator giữ nguyên đề xuất và bấm "Tạo prompt". Hệ thống trả về Prompt
Pack: mỗi ảnh một thẻ prompt tiếng Anh hoàn chỉnh, có màu và quy tắc brand của dự án, placeholder tham
chiếu được tô nổi bật, thứ tự tạo ảnh, alt text và tên file gợi ý, kèm danh sách "File cần chuẩn bị".

**Why this priority**: đây là giá trị cốt lõi của sản phẩm; không có nó thì không có gì để đo Goal Card.

**Independent Test**: với một dự án đã có preset, một Creator được cấp quyền đi từ đăng nhập tới Prompt
Pack hoàn chỉnh cho một outline 6 H2 mà không chỉnh gì, và Pack không còn lỗi kiểm tra.

**Acceptance Scenarios**:

1. **Given** Creator được cấp dự án có Brandkit đã xuất bản, **When** Creator chọn dự án, **Then** hệ
   thống hiện cây danh mục của dự án kèm ghi chú use case của danh mục được chọn.
2. **Given** Creator đã dán outline, **When** nội dung outline thay đổi, **Then** hệ thống hiện ngay
   danh sách heading H2/H3 đã nhận được, trước khi Creator bấm "Phân tích".
3. **Given** outline hợp lệ có 6 H2, **When** Creator bấm "Phân tích", **Then** hệ thống ghi lại bản
   chụp preset hiện hành vào phiên và trả về danh sách chỗ ảnh, mỗi chỗ có heading gắn với, mục đích ảnh,
   số ảnh và lý do đề xuất.
4. **Given** danh sách chỗ ảnh đã có, **When** Creator bấm "Tạo prompt", **Then** mỗi ảnh có một thẻ
   prompt gồm prompt, negative, danh sách tham chiếu, ảnh phụ thuộc, thứ tự tạo, alt text và tên file.
5. **Given** một prompt cần ảnh khác của pack làm tham chiếu, **When** Prompt Pack hiển thị, **Then**
   thẻ đó có nhãn "cần ảnh <mã> trước" và các thẻ được xếp theo thứ tự tạo hợp lệ.
6. **Given** mục đích ảnh cần design thật chưa có file, **When** Prompt Pack hiển thị, **Then** danh sách
   "File cần chuẩn bị" liệt kê từng placeholder design kèm ảnh nào dùng nó.
7. **Given** Brandkit có quy tắc "không nên" và từ cấm, **When** prompt được sinh, **Then** mọi prompt
   chứa quy tắc "không nên" của brand và không chứa từ cấm nào, bất kể ghi chú của Creator.
8. **Given** một thẻ prompt báo lỗi, **When** Creator sửa tay nội dung thẻ, **Then** thẻ được kiểm tra lại
   ngay, mang nhãn "đã sửa tay" và vẫn xem được nội dung gốc.

---

### User Story 2 - Creator chỉ thấy và dùng dự án được cấp (Priority: P1)

Admin mời Creator bằng email và cấp cho họ một hoặc nhiều dự án (trong L1, Admin thao tác trực tiếp trên
công cụ quản trị dữ liệu, chưa có màn hình Admin). Creator chỉ nhìn thấy, đọc và tạo phiên trong dự án
được cấp; mọi thao tác ghi lên dữ liệu cấu hình (brand, style, danh mục, tham chiếu) đều bị từ chối.

**Why this priority**: nhiều store dùng chung một hệ thống; lộ dữ liệu giữa dự án là lỗi không chấp nhận
được và là tiêu chí nghiệm thu L0.

**Independent Test**: tạo 2 Creator, mỗi người được cấp một dự án khác nhau; đăng nhập từng người và
truy vấn trực tiếp dữ liệu (không qua giao diện) để xác nhận chỉ đọc được dữ liệu dự án của mình.

**Acceptance Scenarios**:

1. **Given** Creator A chỉ được cấp dự án X, **When** A truy vấn danh mục của dự án Y, **Then** kết quả
   rỗng, kể cả khi truy vấn không đi qua giao diện.
2. **Given** Creator bất kỳ, **When** họ cố ghi vào Brandkit, Style Set, danh mục hoặc tham chiếu, **Then**
   thao tác bị từ chối.
3. **Given** một người chưa được mời, **When** họ cố đăng ký tài khoản, **Then** không có cách tự đăng ký.
4. **Given** tài khoản bị khoá, **When** người đó đăng nhập hoặc gọi chức năng sinh prompt, **Then** bị
   từ chối.
5. **Given** Creator A, **When** A xem danh sách phiên, **Then** A chỉ thấy phiên của chính mình.

---

### User Story 3 - Rà và cấu hình từng chỗ ảnh trước khi tạo prompt (Priority: P2)

Sau khi "Phân tích", Creator thêm, xoá, kéo đổi thứ tự chỗ ảnh, đổi số ảnh của gallery. Với từng chỗ,
Creator chọn mục đích ảnh, tỉ lệ khung, Style Set (trong danh sách được phép) và ghi chú. Nếu mục đích
thuộc nhóm sản phẩm, hệ thống hiện thêm: loại sản phẩm của store, persona người mẫu hoặc "không người",
design tham chiếu (chọn từ danh sách tham chiếu đã khai báo hoặc "gắn sau") và bối cảnh. Không chỉnh gì thì
dùng mặc định của danh mục.

**Why this priority**: tăng tỉ lệ prompt dùng được; nhưng User Story 1 đã cho giá trị với mặc định.

**Independent Test**: sau "Phân tích", xoá 1 chỗ, thêm 1 chỗ sản phẩm, đổi 1 chỗ thành gallery 3 ảnh,
rồi "Tạo prompt" và xác nhận Pack phản ánh đúng các thay đổi.

**Acceptance Scenarios**:

1. **Given** danh sách chỗ ảnh, **When** Creator đổi số ảnh của một chỗ thành 3, **Then** Pack có 3
   prompt cho chỗ đó (S<n>-1..3) với bố cục khác nhau.
2. **Given** một chỗ ảnh, **When** Creator chọn mục đích thuộc nhóm sản phẩm, **Then** các trường loại
   sản phẩm, persona, design tham chiếu và bối cảnh xuất hiện; với mục đích khác thì không.
3. **Given** Creator chỉ được phép dùng một số Style Set, **When** chọn Style Set, **Then** danh sách chỉ
   gồm các Style Set được phép.
4. **Given** chỗ ảnh A được chỗ ảnh B dùng làm tham chiếu, **When** Creator xoá A, **Then** hệ thống cảnh
   báo trước khi xoá và B không còn tham chiếu tới A sau khi tạo prompt.

---

### User Story 4 - Copy và export Prompt Pack (Priority: P2)

Creator copy từng prompt, copy cả pack, hoặc export cả pack ra JSON, CSV hoặc Markdown để dán vào công
cụ tạo ảnh hay bảng tính của team.

**Why this priority**: là cầu nối sang công cụ tạo ảnh; MVP không tạo ảnh trong app.

**Independent Test**: export một pack 10 prompt ra 3 định dạng; mở từng file và xác nhận đủ 10 dòng/mục
với đủ trường.

**Acceptance Scenarios**:

1. **Given** một Prompt Pack, **When** Creator bấm copy trên một thẻ, **Then** clipboard chứa đúng prompt
   của thẻ đó, giữ nguyên placeholder.
2. **Given** một Prompt Pack, **When** Creator export CSV, **Then** file có một dòng mỗi ảnh với mã ảnh,
   thứ tự tạo, prompt, negative, tham chiếu, ảnh phụ thuộc, alt text, tên file; mở đúng trong bảng tính.
3. **Given** một Prompt Pack, **When** Creator export Markdown hoặc JSON, **Then** nội dung có cùng tập
   trường và cùng thứ tự tạo như CSV.

---

### User Story 5 - Admin theo dõi ai làm gì và chi phí AI (Priority: P3)

Mỗi hành động chính được ghi lại (ai, lúc nào, dự án, phiên, số prompt, lượng sử dụng AI và chi phí ước
tính). Trong L1, Admin đọc log qua công cụ quản trị dữ liệu; màn hình Log thuộc L3.

**Why this priority**: cần để đo Goal Card và chặn chi phí, nhưng không chặn luồng Creator.

**Independent Test**: chạy một phiên đầy đủ, rồi xác nhận có đủ dòng log cho tạo phiên, phân tích, tạo
prompt và export, và Creator không đọc được log.

**Acceptance Scenarios**:

1. **Given** Creator hoàn thành một phiên, **When** Admin xem log, **Then** có các dòng `session.create`,
   `slots.plan`, `prompts.generate`, `pack.export` với thời điểm, người, dự án, phiên.
2. **Given** một lượt gọi AI, **When** ghi log, **Then** dòng log có lượng sử dụng vào/ra, model và chi
   phí ước tính.
3. **Given** Creator, **When** cố đọc hoặc sửa log, **Then** bị từ chối.

### Edge Cases

- Outline không có H2 nào hoặc chỉ có văn bản thường: hệ thống báo cần ít nhất một heading H2 và không
  gọi AI.
- Outline vượt giới hạn độ dài (15.000 ký tự): bị chặn trước khi gọi AI, kèm thông báo số ký tự hiện tại.
- Outline hoặc ghi chú chứa câu lệnh nhắm vào AI ("bỏ qua hướng dẫn…"): bị coi là dữ liệu; quy tắc brand
  vẫn được áp dụng.
- AI trả kết quả sai định dạng: hệ thống thử lại một lần; vẫn sai thì báo lỗi dễ hiểu, giữ nguyên dữ liệu
  Creator đã nhập, không trừ thêm lượt.
- AI không phản hồi hoặc dịch vụ AI lỗi: báo lỗi, cho phép thử lại, không mất outline và chỗ ảnh đã chỉnh.
- Toàn hệ thống chạm giới hạn tổng lượt gọi AI trong ngày của nhà cung cấp: mọi người bị chặn phân tích và
  tạo prompt, thông báo khi nào dùng lại được; sửa tay và export vẫn hoạt động.
- Bài có hơn 20 ảnh: hệ thống vẫn sinh đủ prompt, các ảnh không lặp bố cục giữa các lô.
- Phụ thuộc vòng giữa các ảnh (A cần B, B cần A): Pack bị chặn, chỉ ra các chỗ ảnh gây vòng.
- Prompt vượt giới hạn ký tự (2.000): Validator báo lỗi trên thẻ đó.
- Mục đích ảnh cần tham chiếu bắt buộc (ví dụ sản phẩm) nhưng Creator chưa chọn loại sản phẩm: Validator
  báo thiếu tham chiếu trên thẻ đó.
- Prompt có chữ trên ảnh (`[[TEXT:…]]`): thẻ luôn kèm cảnh báo "kiểm chữ trước khi dùng".
- Admin xuất bản preset mới trong lúc Creator đang làm phiên: phiên đang làm vẫn dùng bản chụp preset của
  lần "Phân tích" đầu tiên, kể cả khi phân tích lại; muốn dùng preset mới thì tạo phiên mới.
- Creator phân tích lại sau khi đã tạo prompt: cảnh báo trước, xác nhận mới thay chỗ ảnh và prompt.
- Dự án chưa có Brandkit đã xuất bản: Creator không thể bấm "Phân tích", thông báo liên hệ Admin.
- Danh mục chưa có Style Guide đã xuất bản: Creator vẫn phân tích được, hệ thống dùng Style Set mặc định của
  dự án và hiện cảnh báo "danh mục chưa có Style Guide".
- Brandkit không có bảng màu hoặc logo: prompt vẫn sinh bình thường, không có dòng màu brand; mục đích ảnh
  có logo tuỳ chọn thì bỏ tham chiếu logo.
- Creator sửa tay làm mất placeholder bắt buộc hoặc thêm từ cấm: thẻ báo lỗi ngay, vẫn giữ nội dung gốc để
  khôi phục.
- Creator đã sửa tay một số prompt rồi bấm tạo lại cả pack: hệ thống cảnh báo các prompt sửa tay sẽ bị thay.
- Creator bị gỡ quyền dự án khi đang làm: thao tác tiếp theo bị từ chối.

## Requirements *(mandatory)*

### Functional Requirements

**Tài khoản và phân quyền**

- **FR-001**: Hệ thống MUST chỉ tạo tài khoản qua lời mời của Admin bằng email; không có đăng ký công khai.
- **FR-002**: Người dùng MUST đăng nhập được bằng email + mật khẩu hoặc liên kết đăng nhập gửi qua email.
- **FR-003**: Mỗi tài khoản MUST có đúng một vai trò (Admin hoặc Creator) và một trạng thái (hoạt động
  hoặc bị khoá); tài khoản bị khoá không dùng được chức năng nào.
- **FR-004**: Quyền truy cập MUST được thực thi ở tầng lưu trữ dữ liệu, không chỉ ẩn trên giao diện:
  Creator chỉ đọc được dự án, danh mục, preset, sản phẩm, persona và tham chiếu của dự án được cấp.
- **FR-005**: Creator MUST NOT tạo, sửa hoặc xoá dự án, danh mục, Brandkit, Style Set, mục đích ảnh,
  sản phẩm, persona hay tham chiếu.
- **FR-006**: Creator MUST chỉ xem và sửa được phiên làm việc của chính mình, trong dự án được cấp.
- **FR-007**: Admin MUST xem được mọi dự án và mọi phiên, và tạo được phiên của chính mình ở mọi dự án
  mà không cần là thành viên. Admin MUST NOT sửa hoặc xoá phiên của người khác (chỉ xem).
- **FR-008**: Mỗi Creator MUST có thể được cấp nhiều dự án; trong mỗi dự án, Admin chọn được các Style Set
  Creator được dùng và một Style Set mặc định. Không cấp Style Set nào trong một dự án = Creator dùng được
  mọi Style Set đang hoạt động của dự án đó.
- **FR-009**: Hệ thống MUST kiểm lại quyền thành viên dự án ngay trước mỗi lượt gọi AI.

**Chọn dự án và nhập bài (C1, C2)**

- **FR-010**: Creator MUST thấy danh sách dự án được cấp và chọn được một danh mục trong cây danh mục của
  dự án; ghi chú use case của danh mục được hiển thị khi chọn.
- **FR-011**: Creator MUST nhập được keyword (bắt buộc), outline markdown (bắt buộc) và ghi chú (tuỳ chọn).
- **FR-012**: Hệ thống MUST hiển thị các heading H2/H3 nhận được từ outline trước khi phân tích.
- **FR-013**: Hệ thống MUST từ chối outline không có H2 hoặc dài hơn 15.000 ký tự, trước khi gọi AI.

**Đề xuất và cấu hình chỗ ảnh (C3, C4)**

- **FR-014**: Khi Creator bấm "Phân tích" lần đầu trong phiên, hệ thống MUST ghi bản chụp preset hiện hành (Brandkit đã xuất
  bản, Style Guide của danh mục, Style Set được phép, mục đích ảnh, sản phẩm, persona, cùng phiên bản của
  chúng) vào phiên; mọi bước
  sau của phiên chỉ dùng bản chụp này.
- **FR-014a**: Creator MUST sửa được keyword, outline, ghi chú và bấm "Phân tích" lại trong cùng phiên ở bất
  kỳ trạng thái nào. Nếu phiên đã có chỗ ảnh hoặc prompt, hệ thống MUST cảnh báo rằng chúng (kể cả prompt sửa
  tay) sẽ bị thay và chỉ tiếp tục khi Creator xác nhận. Phân tích lại MUST dùng bản chụp preset của lần phân
  tích đầu tiên, không chụp lại; phiên quay về trạng thái "đã phân tích".
- **FR-015**: Hệ thống MUST đề xuất danh sách chỗ ảnh, mỗi chỗ có mã (S1, S2…), heading gắn với, mục đích
  ảnh thuộc bộ mục đích đã định nghĩa, số ảnh và lý do đề xuất.
- **FR-016**: Creator MUST thêm, xoá, đổi thứ tự chỗ ảnh và đổi số ảnh của một chỗ.
- **FR-017**: Với mỗi chỗ ảnh, Creator MUST chọn được mục đích, tỉ lệ khung, Style Set (trong danh sách
  được phép) và ghi chú; trường không chỉnh lấy mặc định theo thứ tự: mặc định hệ thống → Brandkit → danh
  mục → Style Set → mục đích ảnh → tuỳ chọn của Creator.
- **FR-018**: Với mục đích thuộc nhóm sản phẩm, hệ thống MUST hiện thêm trường loại sản phẩm, persona
  (hoặc "không người"), design tham chiếu (chọn từ danh sách tham chiếu đã khai báo hoặc "gắn sau") và bối
  cảnh; các mục đích khác không hiện các trường này.
- **FR-018a**: Mỗi danh mục MUST có thể gắn một Style Guide danh mục dạng Markdown, gồm phần thông số (mục
  đích ảnh mặc định, quy tắc mật độ ảnh, chất liệu, ánh sáng, bố cục, mood, góc máy, xử lý màu, negative,
  bật/tắt kiểm màu HEX, điều không nên và từ cấm riêng của danh mục) và phần hướng dẫn bằng lời. Thông số
  phong cách và phần hướng dẫn được đưa cho AI làm đầu vào khi đề xuất chỗ ảnh và viết prompt; negative,
  điều không nên và từ cấm do code gắn/kiểm (FR-021, FR-026). Điều không nên và từ cấm của Brandkit vẫn luôn
  thắng.
- **FR-018b**: Style Guide danh mục MUST có phiên bản và trạng thái (nháp/xuất bản/lưu trữ); chỉ bản đã xuất
  bản được dùng, và phiên bản đang dùng được ghi vào bản chụp preset của phiên.
- **FR-019**: Bộ mục đích ảnh khởi đầu MUST gồm 9 mục: hero (16:9), scene (3:2), howto_step (4:3),
  product_flatlay (1:1), product_on_model (4:5), product_lifestyle (3:2), design_gallery (1:1),
  comparison (3:2), pinterest_pin (2:3), mỗi mục có tham chiếu bắt buộc và tuỳ chọn như tài liệu nguồn
  mục 7.2.

**Sinh Prompt Pack (C5)**

- **FR-020**: Khi Creator bấm "Tạo prompt", AI MUST viết phần mô tả của prompt cho mọi ảnh (chủ thể, bối cảnh,
  bố cục, góc máy, cách áp dụng phong cách của Style Guide, alt text, tên file) trong một lần xử lý cho cả bài
  (chia lô 10 khi bài có hơn 20 ảnh) để các ảnh không lặp bố cục. Đầu vào của AI gồm: keyword, outline, ghi
  chú, cấu hình từng chỗ ảnh, Style Guide danh mục, Style Set, hướng dẫn của mục đích ảnh, sản phẩm và persona
  đã chọn.
- **FR-021**: Sau phần mô tả do AI viết, code MUST gắn thêm các phần bắt buộc, không do AI viết: quy tắc "không
  nên" (Brandkit + Style Guide), negative, hướng dẫn tham chiếu kèm placeholder, tỉ lệ khung và — nếu Brandkit
  có bảng màu — dòng màu brand. Brandkit không có bảng màu thì bỏ dòng màu brand, không bịa màu.
- **FR-022**: Quy tắc "không nên" và từ cấm của Brandkit MUST luôn có hiệu lực; không tuỳ chọn hay ghi chú
  nào của Creator gỡ được.
- **FR-023**: Placeholder MUST theo cú pháp `[[LOẠI:KHOÁ]]` với các loại: logo, sản phẩm, người mẫu,
  design cần gắn, ảnh do pack tạo trước, chữ trên ảnh; mỗi placeholder trong prompt có đúng một mục tham
  chiếu tương ứng và ngược lại.
- **FR-024**: Hệ thống MUST tính thứ tự tạo ảnh từ quan hệ "ảnh này cần ảnh kia trước" và hiển thị thẻ
  theo thứ tự đó.
- **FR-025**: Mỗi ảnh MUST có alt text tiếng Anh mô tả nội dung ảnh và tên file dạng slug tiếng Anh.
- **FR-026**: Hệ thống MUST kiểm Pack và báo lỗi/cảnh báo trên từng thẻ: placeholder thiếu hoặc thừa, thiếu
  tham chiếu bắt buộc, phụ thuộc vòng hoặc trỏ tới chỗ ảnh đã xoá, prompt dài hơn 2.000 ký tự, chứa từ cấm,
  và — chỉ khi Brandkit có bảng màu và Style Set không tắt kiểm này — không có màu HEX nào của brand.
- **FR-027**: Khi Pack còn lỗi, Creator MUST có hai cách xử lý: (a) quay lại sửa cấu hình chỗ ảnh rồi tạo
  lại cả pack; (b) sửa tay nội dung prompt ngay trên thẻ, sau đó hệ thống kiểm tra lại thẻ đó ngay lập tức
  với cùng bộ quy tắc của FR-026.
- **FR-027a**: Prompt đã sửa tay MUST được đánh dấu "đã sửa tay" và giữ lại nội dung gốc do hệ thống sinh;
  dấu này đi kèm khi copy/export. Tạo lại cả pack sẽ thay prompt sửa tay bằng prompt mới, và hệ thống MUST
  cảnh báo trước khi ghi đè.
- **FR-028**: Mỗi prompt MUST ghi lại model AI đã dùng và phiên bản của các preset/template đã ghép.
- **FR-029**: Thẻ có placeholder chữ trên ảnh MUST kèm cảnh báo "kiểm chữ trước khi dùng".
- **FR-030**: Outline và ghi chú của Creator MUST được xử lý như dữ liệu; lệnh nằm trong đó không làm thay
  đổi quy tắc brand hay định dạng đầu ra.

**Copy và export (C6)**

- **FR-031**: Creator MUST copy được từng prompt và copy được cả pack.
- **FR-032**: Creator MUST export được pack ra JSON, CSV và Markdown với cùng tập trường: mã ảnh, thứ tự
  tạo, mục đích, tỉ lệ, prompt, negative, tham chiếu, ảnh phụ thuộc, alt text, tên file, cảnh báo.
- **FR-033**: Hệ thống MUST hiển thị danh sách "File cần chuẩn bị" gồm mọi tham chiếu cần gắn file.

**Nhật ký và chi phí**

- **FR-034**: Hệ thống MUST ghi một dòng nhật ký cho mỗi lần tạo phiên, phân tích, tạo prompt và
  copy/export cả pack, gồm người, thời điểm, dự án, phiên và chi tiết hành động.
- **FR-035**: Mỗi lượt gọi AI MUST ghi lượng sử dụng vào/ra, model, thời gian xử lý và chi phí ước tính.
- **FR-036**: Nhật ký MUST chỉ được ghi bởi hệ thống, không ai sửa được; Creator không đọc được.
- **FR-037**: Hệ thống MUST NOT giới hạn số lượt gọi AI của từng Creator. Hệ thống vẫn MUST giữ giới hạn tổng
  số lượt mỗi ngày của nhà cung cấp AI do Admin cấu hình (chặn chi phí bất thường); vượt giới hạn tổng thì
  chặn trước khi gọi.

**Cấu hình AI**

- **FR-038**: Admin MUST đổi được nhà cung cấp AI (tương thích chuẩn OpenAI) và model cho từng bước mà
  không sửa code; khoá truy cập AI không bao giờ hiển thị trên giao diện hay lưu cùng dữ liệu ứng dụng.

**Giao diện**

- **FR-039**: Giao diện MUST bằng tiếng Việt; prompt, alt text và tên file đầu ra bằng tiếng Anh.
- **FR-040**: Mọi màn hình MUST dùng được từ chiều rộng 360px tới desktop, không cuộn ngang trang; có trạng
  thái đang xử lý, trống và lỗi rõ ràng cho phân tích và tạo prompt.

### Key Entities *(include if feature involves data)*

- **Tài khoản (Profile)**: người dùng; họ tên, vai trò (Admin/Creator), trạng thái (hoạt động/khoá).
- **Dự án (Project)**: một store/blog; tên, domain, mô tả, khách hàng mục tiêu, trạng thái.
- **Thành viên dự án**: liên kết Creator ↔ dự án, kèm Style Set mặc định và các Style Set được phép.
- **Brandkit**: thuộc một dự án, có phiên bản; giọng brand, danh sách nên, danh sách không nên, từ cấm;
  tuỳ chọn: bảng màu (tên, HEX, vai trò primary/accent/nền), typography, logo; trạng thái nháp/xuất bản/lưu trữ; mỗi dự
  án đúng một bản đã xuất bản.
- **Style Set**: thuộc một dự án, có phiên bản; chất liệu, ánh sáng, bố cục, mood, camera, xử lý màu,
  negative, ảnh ví dụ, cờ bật/tắt kiểm màu HEX brand.
- **Danh mục (Category)**: cây cha/con trong dự án; tên, slug, URL, ghi chú use case, mục đích ảnh mặc định,
  Style Set mặc định, Style Guide đang dùng.
- **Style Guide danh mục**: tài liệu Markdown thuộc một danh mục của một dự án, có phiên bản và trạng thái;
  phần thông số do code ghép vào prompt, phần hướng dẫn làm dữ liệu tham chiếu cho AI.
- **Tham chiếu (Reference)**: thứ cần gắn khi tạo ảnh, thuộc dự án (logo, ảnh nền sản phẩm, design, ảnh
  persona, ảnh ví dụ); loại, khoá dùng trong placeholder, tên, mô tả, tag, đường dẫn tuỳ chọn tới thư mục của
  team. Trong L1 app không lưu file.
- **Loại sản phẩm**: sản phẩm store bán (hoodie, mug…); khoá, tên, ghi chú mockup, tham chiếu ảnh nền.
- **Persona**: người mẫu dùng trong ảnh; độ tuổi, giới tính, ngoại hình, phong cách, vibe, tham chiếu ảnh.
  (Khác với persona độc giả trong hồ sơ brand.)
- **Mục đích ảnh (Purpose template)**: dùng chung mọi dự án, có phiên bản; khoá, tên, tỉ lệ mặc định,
  hướng dẫn viết prompt cho AI, mẫu hướng dẫn tham chiếu, tham chiếu bắt buộc, tham chiếu tuỳ chọn.
- **Phiên làm việc (Session)**: một bài viết; dự án, danh mục, Creator, keyword, outline, ghi chú, bản chụp
  preset (chụp một lần ở lần phân tích đầu, không đổi suốt đời phiên), trạng thái (nháp → đã phân tích → đã
  tạo prompt; phân tích lại đưa phiên về "đã phân tích").
- **Chỗ ảnh (Image Slot)**: thuộc một phiên; mã, vị trí, heading, mục đích, tỉ lệ, số ảnh, Style Set, tuỳ
  chọn sản phẩm/persona/design/bối cảnh, lý do AI đề xuất.
- **Prompt**: một ảnh; mã (S2-1), prompt, negative, tham chiếu, ảnh phụ thuộc, thứ tự tạo, alt text, tên
  file, model, phiên bản preset/template, cảnh báo, cờ "đã sửa tay" và nội dung gốc do hệ thống sinh.
- **Nhà cung cấp AI**: tên, địa chỉ dịch vụ, tên biến môi trường chứa khoá, model từng bước, hỗ trợ đầu ra
  có schema hay không, giới hạn lượt/ngày, đơn giá để ước tính chi phí.
- **Nhật ký hoạt động**: người, dự án, hành động, đối tượng, chi tiết, lượng sử dụng AI, chi phí ước tính,
  thời điểm; chỉ thêm, không sửa.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: 100% truy vấn của Creator vào dữ liệu dự án không được cấp trả về rỗng hoặc bị từ chối, kiểm
  bằng 2 tài khoản Creator trên 2 dự án khác nhau.
- **SC-002**: 100% thao tác ghi của Creator vào dữ liệu cấu hình (brand, style, danh mục, tham chiếu) bị từ chối.
- **SC-003**: Prompt Pack của một bài (outline 6 H2, khoảng 7–10 ảnh) sẵn sàng trong ≤ 60 giây kể từ khi
  bấm "Tạo prompt".
- **SC-004**: 3 bài thật ở 3 danh mục khác nhau của dự án thử nghiệm cho Pack không còn lỗi kiểm tra.
- **SC-005**: Khi Admin chấm ảnh tạo từ 3 bài trên, ≥ 70% prompt cho ảnh dùng được ngay lần đầu.
- **SC-006**: 0 prompt chứa từ cấm của Brandkit; với dự án có bảng màu, 100% prompt chứa ít nhất một màu
  HEX của brand (trừ Style Set tắt kiểm).
- **SC-007**: Outline 6 H2 cho ra 7–10 ảnh được đề xuất và không có phụ thuộc vòng.
- **SC-008**: Một Creator mới hoàn thành Pack đầu tiên mà không cần hỏi Admin; thời gian từ tạo phiên tới
  export được ghi nhận trong nhật ký để theo dõi mục tiêu ≤ 10 phút/bài.
- **SC-009**: 100% phiên hoàn chỉnh có đủ 4 loại dòng nhật ký (tạo phiên, phân tích, tạo prompt, export).
- **SC-010**: File export CSV mở đúng trong bảng tính thông dụng với một dòng mỗi ảnh và không vỡ cột.

## Assumptions

- Dự án thử nghiệm cho L1 là Print Your Wear (hồ sơ brand trong `docs/brand.json`: 6 danh mục, chủ đề nhạy
  cảm, từ nên tránh). Bảng màu, font và logo chưa có và không bắt buộc; chất lượng ảnh dựa chủ yếu vào Style
  Guide của từng danh mục, Admin viết dựa trên hồ sơ brand và ảnh hiện có của blog. Khung 6 file đã tạo sẵn
  tại `docs/style-guides/print-your-wear/` (vai trò danh mục và chủ đề nhạy cảm lấy từ `brand.json`; phần
  phong cách hình ảnh Admin cần điền và xuất bản trước khi nghiệm thu L1).
- Trong L1, Admin nhập dự án, Brandkit, Style Guide của các danh mục, 1 Style Set mặc định, 3–6 danh mục, loại sản phẩm, 1–2 persona và cấp quyền
  trực tiếp bằng công cụ quản trị dữ liệu; màn hình Admin thuộc spec L2.
- Bộ 9 mục đích ảnh được nạp sẵn vào hệ thống và có phiên bản ngay từ L0; màn hình sửa mục đích ảnh thuộc L3.
- Công cụ tạo ảnh của team: ChatGPT/GPT Image, prompt ngôn ngữ tự nhiên tiếng Anh; cú pháp riêng từng tool
  để sau.
- Outline được dán dạng markdown; đọc từ Google Docs để sau.
- Giới hạn mặc định: không giới hạn lượt theo Creator; 300 lượt mỗi ngày cho nhà cung cấp AI (tổng toàn hệ
  thống), outline 15.000 ký tự, prompt 2.000 ký tự; Admin chỉnh được.
- L1 không lưu, upload hay hiển thị file ảnh; lưu file và thư viện asset thuộc L2 (A4) và L4.
- Quy mô: ≤ 20 dự án, ≤ 30 tài khoản; không đa tổ chức, không billing.
- Desktop là bối cảnh làm việc chính; màn hình hẹp vẫn dùng được.
- Ngoài phạm vi spec này: màn hình Admin (A1–A4, A6), sinh lại một phần (C7), lịch sử và nhân bản phiên
  (C8), chấm prompt và trang Chất lượng (C9, A8), màn hình Log (A7), sửa mục đích ảnh (A5), ghim ví dụ mẫu,
  caption ảnh, tạo ảnh trong app (L4).
- Phụ thuộc: dịch vụ AI tương thích chuẩn OpenAI; dịch vụ gửi email cho lời mời và đăng nhập.
