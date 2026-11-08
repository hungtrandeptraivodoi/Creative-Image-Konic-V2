# Brand Image Prompt Studio Constitution

## Core Principles

### I. MVP-first, tránh overengineering

- Mỗi tính năng MUST thuộc một lát đã định (L0 → L4) và chỉ làm phần cần cho tiêu chí nghiệm
  thu của lát đó. Lát sau chỉ bắt đầu khi lát trước qua nghiệm thu bằng dữ liệu thật.
- KHÔNG thêm abstraction, cấu hình, tuỳ chọn hay hạ tầng "để dành cho tương lai". Thêm khi có
  nhu cầu đo được hoặc spec đã duyệt yêu cầu.
- Bị loại cho tới khi có lý do đo được: framework multi-agent, vector DB, đa tổ chức, billing,
  tạo ảnh trong app (L4), cú pháp riêng từng tool tạo ảnh.
- Quy mô thiết kế: ≤ 20 dự án, ≤ 30 tài khoản; không tối ưu cho quy mô lớn hơn.

**Lý do:** đo giá trị luồng Creator sớm nhất với ít code nhất.

### II. Kiến trúc module hoá, dễ bảo trì

- Tách lớp rõ ràng, phụ thuộc một chiều: UI/route (`app/`) → service/data access (`lib/`) →
  logic thuần (`lib/engine`). `lib/engine` KHÔNG import DB, AI SDK hay framework UI.
- AI provider đi qua interface `AIProvider`; đổi provider không sửa engine hay UI.
- Mỗi module có một trách nhiệm; schema dùng chung (Zod + JSON Schema) định nghĩa đúng một nơi.
- Schema DB chỉ thay đổi qua `supabase/migrations/*.sql`.

**Lý do:** engine và quyền là phần sống lâu nhất; phải sửa được mà không kéo theo UI.

### III. Tái sử dụng code và component sẵn có

- Trước khi viết mới MUST tìm trong repo xem đã có hàm, component, hook, type hay schema tương
  đương chưa; có thì dùng hoặc mở rộng, không nhân bản.
- Ưu tiên component của thư viện UI đã chọn và tính năng có sẵn của Supabase/Next.js hơn tự viết.
- Thêm dependency mới MUST có lý do ghi trong plan (cái gì sẵn có không đáp ứng được).

**Lý do:** ít code trùng = ít chỗ lệch hành vi và ít chỗ phải sửa.

### IV. Ưu tiên usability và UI responsive

- Luồng Creator MUST hoàn thành được một Prompt Pack ≤ 10 phút mà không cần hỏi Admin: mặc định
  lấy từ danh mục/preset, trường phụ chỉ hiện khi cần, lỗi nói rõ cách sửa.
- Mọi màn hình MUST dùng được từ chiều rộng 360px tới desktop, không cuộn ngang trang; bảng phức
  tạp được phép chuyển thành dạng thẻ trên màn hẹp. Desktop là bối cảnh làm việc chính.
- Thao tác chính MUST dùng được bằng bàn phím, có trạng thái loading/empty/error rõ ràng.

**Lý do:** Goal Card đo bằng thời gian và mức tự phục vụ của Creator.

### V. Bắt buộc test tự động cho business logic

- Business logic MUST có test tự động trước khi nối vào UI, gồm: Prompt Engine (resolver,
  assembler, validator, graph), thứ tự ưu tiên context, định dạng export, quy tắc giới hạn lượt gọi.
- Phân quyền MUST có test RLS bằng tài khoản Creator thật (đọc dự án không được cấp → rỗng; ghi
  bảng preset → bị từ chối).
- Test không gọi mạng tới AI: dùng provider giả qua interface `AIProvider`.
- Sửa bug business logic MUST kèm test tái hiện bug đó.

**Lý do:** chất lượng prompt và bảo mật nằm ở code tất định, nên phải chứng minh bằng test.

### VI. Không đưa breaking change một cách âm thầm

- Breaking change gồm: đổi/xoá cột hay bảng, đổi policy RLS, đổi cú pháp placeholder, đổi schema
  đầu ra AI hoặc định dạng export, đổi hành vi preset/phiên bản, đổi API route.
- Mỗi breaking change MUST được nêu rõ trong spec/plan và mô tả thay đổi, kèm đường chuyển đổi
  (migration dữ liệu, tương thích ngược hoặc lý do không cần) và được Hung duyệt trước khi làm.
- Phiên làm việc và prompt đã sinh MUST vẫn đọc được sau thay đổi.

**Lý do:** dữ liệu phiên cũ dùng để so sánh chất lượng giữa các phiên bản preset.

### VII. Bám sát spec đã duyệt

- Code MUST hiện thực đúng spec/plan đã duyệt; không tự thêm, bớt hay đổi yêu cầu sản phẩm.
- Gặp chỗ spec mơ hồ, mâu thuẫn hoặc sai: dừng phần đó, ghi lại và hỏi (hoặc chạy
  `/speckit-clarify`), không tự đoán rồi làm tiếp.
- Thay đổi phạm vi MUST đi qua cập nhật spec trước, rồi mới tới code.
- Tài liệu nguồn: `docs/Brand Image Prompt Studio — Mô tả tính năng & Đặc tả build.md` và các
  spec trong `specs/`.

**Lý do:** spec là thoả thuận với người dùng; lệch spec âm thầm làm hỏng nghiệm thu.

### VIII. Ghi lại quyết định kiến trúc quan trọng

- Quyết định khó đảo ngược hoặc ảnh hưởng nhiều module MUST có ADR ngắn trong `docs/adr/`
  (`NNNN-tieu-de.md`: bối cảnh, quyết định, phương án đã loại, hệ quả).
- Thuộc diện bắt buộc: chọn thư viện/dịch vụ, thay đổi data model hoặc RLS, cách gọi AI, cách
  chia lô/timeout, cách lưu phiên bản preset, mọi ngoại lệ với constitution.
- Plan MUST tham chiếu ADR liên quan; ADR đã chấp nhận không sửa đè, chỉ được thay bằng ADR mới.

**Lý do:** người (và agent) đến sau cần biết vì sao, không chỉ cái gì.

### IX. Không có secret/credential trong source control

- API key AI, `SUPABASE_SERVICE_ROLE_KEY`, mật khẩu, token MUST chỉ nằm trong biến môi trường
  (`.env.local` bị gitignore, biến môi trường Vercel). Repo chỉ có `.env.example` chứa tên biến.
- Secret KHÔNG vào code, DB, log, response hay bundle trình duyệt. Bảng `ai_providers` chỉ lưu
  tên biến môi trường (`key_env`). Mọi lượt gọi AI chạy phía server.
- Lộ secret = thu hồi và thay key ngay, rồi báo Hung.

**Lý do:** Creator dùng nội bộ nhưng không được giữ khoá chi phí và dữ liệu toàn hệ thống.

### X. Không deploy, push, merge khi chưa được duyệt

- Push lên remote, merge, tạo/đóng PR, deploy Vercel, chạy migration lên Supabase remote, thay
  đổi biến môi trường hay dữ liệu production MUST có duyệt rõ ràng của Hung cho đúng lần đó.
- Duyệt cho một lần không áp dụng cho lần sau.
- Làm việc trên nhánh riêng; không commit thẳng lên nhánh chính.

**Lý do:** hành động ra bên ngoài khó đảo ngược và ảnh hưởng người dùng thật.

## Ràng buộc kiến trúc sản phẩm

Các ràng buộc dưới đây lấy từ spec đã duyệt (mục 6–10) và có hiệu lực như nguyên tắc.

**A. AI viết mô tả, code giữ phần bắt buộc.** AI chỉ chạy ở bước B (Slot Planner) và D (Prompt
Writer); bước A, E, F là code tất định. Ở bước D, AI viết phần mô tả của prompt dựa trên Style Guide
danh mục, Style Set, mục đích ảnh và nội dung bài. Ở bước E, code gắn thêm phần bắt buộc từ preset:
quy tắc "không nên", negative, hướng dẫn tham chiếu kèm placeholder, tỉ lệ khung và — chỉ khi
Brandkit có khai báo — màu brand và logo; AI không viết các phần này. Thứ tự ưu tiên context: mặc
định hệ thống → Brandkit → Danh mục (Style Guide) → Style Set → Purpose template → tuỳ chọn Creator;
danh sách "không nên" và từ cấm của Brandkit luôn thắng. Không hard-code brand dự án nào vào code;
`docs/brand.json` và `docs/style-guides/` chỉ là dữ liệu seed.

**B. Phân quyền chặn ở database.** Mọi bảng dữ liệu dự án bật RLS dùng `is_admin()` và
`is_member(project_id)`; bảng con không có `project_id` dùng policy đi qua bảng cha. Server kiểm lại
quyền thành viên trước mọi lượt gọi AI. `activity_logs` chỉ ghi bằng service role. Tắt đăng ký
công khai, tài khoản chỉ tạo qua lời mời. Khi có lưu file (từ L2): Storage private, signed URL ≤ 1
giờ. Migration chạm bảng
dự án MUST kèm policy trong cùng migration.

**C. Preset có phiên bản, phiên đóng băng.** Brandkit, Style Guide danh mục, Style Set, Purpose
template đã xuất bản không sửa đè. Ở lần "Phân tích" đầu tiên, code chụp preset vào
`sessions.context_snapshot`; phân tích lại trong cùng phiên dùng lại bản chụp đó. Mỗi prompt ghi `model` và
`template_versions`; đổi prompt hệ thống bước B/D phải tăng số phiên bản.

**D. Đầu ra AI có schema và qua Validator.** Structured output (JSON Schema) và kiểm lại bằng Zod;
provider không hỗ trợ → JSON mode + Zod + thử lại 1 lần. Placeholder `[[LOẠI:KHOÁ]]`, mỗi
placeholder có đúng một dòng `references`. `gen_order` sắp topo; vòng phụ thuộc bị chặn. Validator
kiểm tham chiếu bắt buộc, độ dài prompt, từ cấm, và màu HEX brand chỉ khi Brandkit có bảng màu và
Style Guide/Style Set không tắt kiểm này. Prompt Creator sửa tay chạy lại cùng Validator và được đánh
dấu "đã sửa tay". Outline/ghi chú của Creator đưa
vào AI trong thẻ phân cách, model không làm theo lệnh bên trong.

**E. Quan sát được và chặn chi phí.** Mỗi hành động chính ghi một dòng `activity_logs` (tokens,
chi phí ước tính, model, thời gian cho lượt gọi AI). Không giới hạn lượt theo từng user; giới hạn
tổng mỗi ngày của nhà cung cấp AI (`daily_call_limit`) kiểm trên server trước khi gọi. Outline giới
hạn độ dài (mặc định 15.000 ký tự).

## Ràng buộc kỹ thuật & sản phẩm

- **Stack:** Next.js (App Router) + TypeScript trên Vercel; Supabase (Auth, Postgres + RLS,
  Storage); SDK `openai` cho Node với `baseURL` tuỳ chỉnh; Zod. Đổi stack cần ADR và sửa constitution.
- **Cấu trúc:** route Creator dưới `app/(creator)`, Admin dưới `app/(admin)/admin`, API pipeline
  dưới `app/api/sessions/[sid]/{plan,prompts}`; logic ở `lib/engine`, `lib/ai`, `lib/log.ts`.
- **Ngôn ngữ:** giao diện tiếng Việt; prompt, alt text và tên file ảnh đầu ra tiếng Anh.
- **Hiệu năng:** pack sinh xong ≤ 60 giây/bài; bước D chạy một lần cho cả bài, > 20 ảnh thì chia
  lô 10; tôn trọng giới hạn thời gian chạy function của gói Vercel.
- **Đầu ra Prompt Pack:** prompt, negative, references, `depends_on`, `gen_order`, alt text, tên
  file; placeholder tô nổi bật; export JSON/CSV/Markdown. Ảnh có `[[TEXT:…]]` kèm cảnh báo kiểm chữ.

## Quy trình phát triển & cổng chất lượng

- Chuỗi Spec Kit: `/speckit-specify` → `/speckit-clarify` → `/speckit-plan` → `/speckit-tasks` →
  `/speckit-implement`. Mỗi spec ghi rõ thuộc lát nào và tiêu chí nghiệm thu đo được.
- "Constitution Check" trong plan MUST đối chiếu đủ 10 nguyên tắc và 5 ràng buộc A–E; vi phạm
  ghi vào Complexity Tracking kèm lý do và ADR.
- Một thay đổi chỉ được coi là xong khi: test tự động liên quan chạy qua; type-check và lint
  sạch; không có secret trong diff; migration mới có policy RLS; breaking change (nếu có) đã được
  nêu và duyệt; ADR (nếu thuộc diện bắt buộc) đã viết.
- Báo cáo kết quả trung thực: test fail thì nói fail kèm output; bước bị bỏ qua thì nói rõ.
- Số liệu nghiệm thu (≥ 70% L1, ≥ 80% L3, ≤ 10 phút/bài) chỉ báo "đạt" khi có dữ liệu chấm thật.

## Governance

- Constitution có hiệu lực cao hơn mọi quy ước khác của dự án. Spec, plan, tasks và review MUST
  kiểm tra tuân thủ; xung đột được giải quyết theo constitution.
- Sửa đổi: đề xuất qua `/speckit-constitution`, kèm Sync Impact Report, được Hung duyệt; cập nhật
  `CLAUDE.md` và tài liệu phụ thuộc nếu bị ảnh hưởng.
- Phiên bản theo SemVer: MAJOR khi bỏ hoặc định nghĩa lại nguyên tắc; MINOR khi thêm nguyên tắc
  hoặc mở rộng đáng kể; PATCH khi làm rõ câu chữ.
- Rà soát tuân thủ ở cuối mỗi lát (L0–L4) trước khi bắt đầu lát kế tiếp.

**Version**: 2.0.1 | **Ratified**: 2026-11-08 | **Last Amended**: 2026-11-08
