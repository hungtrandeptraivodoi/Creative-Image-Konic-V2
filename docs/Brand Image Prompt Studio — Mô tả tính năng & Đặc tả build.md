# Brand Image Prompt Studio — Mô tả tính năng & Đặc tả build

Nov 8, 2026 · @Hung

## 1. Goal Card

App biến "keyword + outline của một bài blog" thành một **Prompt Pack**: đủ prompt cho mọi ảnh của bài, đúng brand của dự án, có placeholder đánh dấu ảnh tham chiếu. Admin giữ brand và quyền, Creator chỉ chọn và bấm.

| Dòng | Nội dung |
| --- | --- |
| Bài toán SEO/business | Ảnh blog của nhiều store POD đang lệch brand, mỗi người viết prompt một kiểu, tốn thời gian và chất lượng phụ thuộc người làm. Mỗi danh mục blog lại cần loại ảnh khác nhau. |
| Người dùng & tần suất | **Admin** (Hung/lead): setup dự án, brandkit, preset, tài khoản, vài lần mỗi tuần. **Creator** (content/design): mỗi bài viết một lần, giả định 5–20 bài/ngày cho cả team. |
| Đầu vào → Đầu ra | Dự án + danh mục + keyword + outline (+ lựa chọn từng chỗ ảnh) → Prompt Pack: danh sách chỗ ảnh, prompt hoàn chỉnh mỗi ảnh, placeholder tham chiếu, thứ tự tạo ảnh, alt text và tên file gợi ý; copy hoặc export. |
| Thước đo "xong" | Creator làm xong Prompt Pack cho 1 bài trong ≤ 10 phút, không phải hỏi Admin. Hung chấm 10 bài thật: ≥ 80% prompt cho ảnh dùng được ngay lần đầu. Creator chỉ thấy dự án được cấp. Admin xem được log ai làm gì. |

**Giả định đã tự chốt** (sửa thẳng ở đây nếu sai):

- MVP chỉ **tạo prompt**, chưa gọi API tạo ảnh trong app. Creator dán prompt vào công cụ tạo ảnh. Tạo ảnh trong app nằm ở lát cắt sau.
- Prompt viết bằng tiếng Anh, ngôn ngữ tự nhiên (hợp GPT Image và đa số model). Cú pháp riêng của từng tool (ví dụ tham số của Midjourney) để sau.
- Outline được dán dạng markdown H2/H3. Đọc từ Google Docs để sau.
- Quy mô: ≤ 20 dự án, ≤ 30 tài khoản. Không cần đa tổ chức, không billing.
- Số liệu trong Goal Card (5–20 bài/ngày, 80%) là giả định để đo, chưa phải số thật.

## 2. Tổng quan

App là bộ khung nối **context của dự án** (brand, danh mục, sản phẩm) với **context của bài viết** (keyword, outline) để sinh prompt ảnh nhất quán. Phần brand do code ghép cố định; AI chỉ viết phần cần phán đoán: chia bài thành chỗ ảnh và mô tả bối cảnh từng ảnh.

**Ví dụ đời thường.** Giống một studio chụp ảnh có sẵn "sổ tay thương hiệu" cho từng khách. Thợ ảnh (Creator) chỉ cần biết hôm nay chụp cho bài nào, các cảnh cần gì. Đèn, màu, logo, phong cách đã có trong sổ do trưởng studio (Admin) giữ.

**Ví dụ thực tế.** Dự án *TartanVibes* → danh mục *Gift Guides* → keyword "tartan gifts for dad" → outline 6 H2. App đề xuất 8 chỗ ảnh: 1 ảnh hero, 5 ảnh mockup sản phẩm (mỗi H2 một món quà, có 1 H2 là gallery 3 ảnh), 1 ảnh lifestyle, 1 pin Pinterest. Creator chỉnh vài chỗ, bấm tạo, nhận 10 prompt có sẵn màu tartan của brand, logo đúng vị trí và placeholder `[[REF:DESIGN#1]]` chỗ cần gắn file design thật.

### Khái niệm cốt lõi

| Khái niệm | Là gì | Ai tạo |
| --- | --- | --- |
| Dự án (Project) | Một site/brand, ví dụ một store POD và blog của nó | Admin |
| Brandkit | Logo, bảng màu có vai trò (primary, accent, nền), font, quy tắc nên/không nên. Có phiên bản | Admin |
| Style Set (bộ phong cách) | Một "công thức" hình ảnh: chất liệu (ảnh chụp, 3D, flat illustration), ánh sáng, bố cục, mood, negative prompt. Một dự án có nhiều bộ | Admin |
| Danh mục (Category) | Một nhánh taxonomy của blog, kèm ghi chú use case ảnh và loại ảnh mặc định | Admin |
| Thư viện asset | Logo, ảnh nền sản phẩm (blank hoodie, mug…), design mẫu, persona người mẫu, ảnh ví dụ đẹp | Admin |
| Mục đích ảnh (Purpose) | Loại ảnh: hero, minh hoạ bối cảnh, mockup sản phẩm, on-model, gallery, Pinterest pin… Mỗi loại có template prompt và tham chiếu bắt buộc | Admin |
| Phiên làm việc (Session) | Một bài viết Creator đang làm: keyword, outline, preset đã đóng băng | Creator |
| Chỗ ảnh (Image Slot) | Một vị trí cần ảnh trong bài, có 1 hoặc nhiều ảnh (gallery) | AI đề xuất, Creator chỉnh |
| Prompt Pack | Toàn bộ prompt của một phiên, kèm placeholder, thứ tự tạo, alt text | App sinh |
| Placeholder | Chỗ đánh dấu ảnh tham chiếu phải gắn khi tạo ảnh, ví dụ `[[REF:LOGO]]`, `[[GEN:S3-1]]` | App sinh |

## 3. Vai trò & quyền

Hai vai trò, quyền được chặn ở tầng database (Row Level Security của Supabase), không chỉ ẩn trên giao diện. Creator chỉ đọc được dữ liệu của dự án và preset Admin đã cấp.

| Hành động | Admin | Creator |
| --- | --- | --- |
| Tạo/sửa/lưu trữ dự án, danh mục | Có | Không |
| Tạo/sửa Brandkit, Style Set, Purpose template | Có | Không (chỉ xem preset được cấp) |
| Upload asset vào thư viện dự án | Có | Không (upload file riêng cho phiên của mình: để sau) |
| Tạo tài khoản, khoá tài khoản, cấp dự án và preset | Có | Không |
| Cấu hình AI provider, model | Có | Không |
| Xem danh sách dự án | Tất cả | Chỉ dự án được cấp |
| Tạo phiên, sinh chỗ ảnh, sinh Prompt Pack | Có (để test) | Có, trong dự án được cấp |
| Xem/sửa phiên | Tất cả (chỉ xem) | Phiên của mình |
| Chấm prompt (đạt/không đạt + lý do) | Có | Có |
| Ghim prompt thành "ví dụ mẫu" | Có | Không |
| Xem log làm việc, thống kê | Tất cả | Không |

**Quy tắc cấp quyền.** Một Creator gắn với nhiều dự án. Trong mỗi dự án, Admin chọn Style Set nào Creator được dùng và Style Set mặc định. Không cấp gì = Creator dùng được mọi Style Set đang hoạt động của dự án (giả định, dễ vận hành hơn).

## 4. Danh sách tính năng

Có 18 tính năng chia 2 nhóm. Cột "Lát" cho biết tính năng nằm ở lát cắt nào trong lộ trình (mục 11); L1 là lát mỏng chạy end-to-end đầu tiên.

### Creator

| # | Tính năng | Làm gì | Để làm gì | Xài thế nào | Lát |
| --- | --- | --- | --- | --- | --- |
| C1 | Chọn dự án & danh mục | Hiện dự án được cấp, rồi cây danh mục của dự án kèm ghi chú use case | Nạp đúng brand và loại ảnh của danh mục, không phải nhớ | Bấm thẻ dự án → chọn danh mục từ dropdown | L1 |
| C2 | Nhập bài | Ô keyword, ô outline (dán markdown H2/H3), ghi chú tuỳ chọn | Xác định chủ đề và các bối cảnh/use case nhỏ trong bài | Dán outline → app hiện preview các heading đã nhận | L1 |
| C3 | Đề xuất chỗ ảnh | AI đọc outline, trả danh sách chỗ ảnh: gắn với heading nào, mục đích gợi ý, số ảnh, lý do | Không bỏ sót chỗ cần ảnh, không nhồi ảnh vô ích | Bấm "Phân tích" → xem bảng chỗ ảnh, thêm/xoá/đổi thứ tự, đổi số ảnh (gallery) | L1 |
| C4 | Cấu hình từng chỗ ảnh | Chọn mục đích, format (tỉ lệ), Style Set, ghi chú. Nếu là ảnh sản phẩm: loại sản phẩm, persona người mẫu, design tham chiếu, bối cảnh | Creator quyết định đúng chỗ cần quyết, phần còn lại ăn theo preset | Mở từng thẻ chỗ ảnh; trường phụ hiện theo mục đích đã chọn | L1 |
| C5 | Sinh Prompt Pack | Ghép prompt cho mọi ảnh, đánh dấu placeholder, xếp thứ tự tạo, gợi ý alt text và tên file | Có đủ prompt dùng ngay, đúng brand | Bấm "Tạo prompt" → xem thẻ prompt, placeholder tô màu, nhãn "cần ảnh S2-1 trước" | L1 |
| C6 | Copy & export | Copy từng prompt, copy cả pack, export JSON/CSV/Markdown | Đưa sang công cụ tạo ảnh hoặc Sheet của team | Nút copy trên mỗi thẻ; nút export ở đầu pack | L1 |
| C7 | Tạo lại một phần | Sinh lại 1 chỗ ảnh hoặc 1 prompt, giữ phần khác | Không tốn token chạy lại cả bài | Nút "Sinh lại" trên thẻ, có ô gõ yêu cầu chỉnh | L2 |
| C8 | Lịch sử phiên | Danh sách bài đã làm, mở lại, nhân bản phiên | Làm bài tương tự nhanh hơn | Trang "Phiên của tôi" | L2 |
| C9 | Chấm prompt | Đạt/không đạt + tag lý do (sai màu, sai sản phẩm, bố cục xấu, chữ lỗi…) + link ảnh kết quả | Dữ liệu để Admin sửa preset | 2 nút trên mỗi thẻ sau khi tạo ảnh xong | L3 |

### Admin

| # | Tính năng | Làm gì | Để làm gì | Xài thế nào | Lát |
| --- | --- | --- | --- | --- | --- |
| A1 | Quản lý dự án & danh mục | CRUD dự án; cây danh mục (cha/con), mỗi danh mục có ghi chú use case, mục đích ảnh mặc định, Style Set mặc định. Dán danh sách danh mục hàng loạt | Phản ánh taxonomy thật của blog | Trang Dự án → tab Danh mục | L2 (L1 nhập qua Supabase) |
| A2 | Brandkit | Logo (nhiều biến thể), bảng màu có vai trò và mã HEX, font, giọng thương hiệu, danh sách nên/không nên. Lưu thành phiên bản | Giữ nhận diện nhất quán giữa mọi Creator | Tab Brandkit → sửa → "Xuất bản phiên bản mới" | L2 |
| A3 | Style Set | Bộ token phong cách (chất liệu, ánh sáng, bố cục, mood, camera, xử lý màu), negative prompt, ảnh ví dụ. Có phiên bản, nhân bản được | Tạo nhiều "chất ảnh" khác nhau cho cùng brand | Tab Style Sets; nút "Xem prompt mẫu" để thử trước khi xuất bản | L2 |
| A4 | Thư viện asset | Upload logo, ảnh nền sản phẩm, design mẫu, persona, ảnh ví dụ; gắn tag; khai báo loại sản phẩm của store | Placeholder trỏ được tới file thật | Tab Assets, kéo thả file | L2 |
| A5 | Purpose template | Sửa template prompt của từng mục đích ảnh, format mặc định, tham chiếu bắt buộc | Điều chỉnh chất lượng theo thời gian | Trang Cài đặt → Mục đích ảnh | L3 (L1 dùng bộ mặc định trong code) |
| A6 | Tài khoản & cấp quyền | Mời Creator qua email, khoá/mở, gán dự án và Style Set được dùng | Kiểm soát ai làm dự án nào | Trang Người dùng → "Mời" → tick dự án | L2 (L1 qua Supabase) |
| A7 | Log làm việc | Bảng hành động: ai, lúc nào, dự án, bài, số prompt, token và chi phí ước tính. Lọc theo người/dự án/ngày | Theo dõi năng suất và chi phí AI | Trang Log, export CSV | L3 |
| A8 | Chất lượng preset | Tỉ lệ đạt theo Style Set / mục đích / phiên bản, top lý do không đạt; ghim prompt tốt thành ví dụ mẫu | Biết sửa preset nào trước | Trang Chất lượng | L3 |
| A9 | AI provider | Khai báo provider (OpenAI hoặc endpoint tương thích OpenAI), model cho từng bước, giới hạn lượt/ngày | Đổi model/provider không sửa code | Trang Cài đặt → AI. Key nằm ở biến môi trường, UI chỉ chọn tên | L1 (cấu hình bằng env) |

## 5. Luồng người dùng

Creator đi 6 bước và chỉ quyết định ở bước 4–5. Admin setup một lần cho mỗi dự án rồi chỉ tinh chỉnh theo số liệu chấm.

### Creator: từ outline tới Prompt Pack

1. **Đăng nhập** (email + mật khẩu hoặc magic link). Vào Dashboard thấy các dự án được cấp.
2. **Chọn dự án → danh mục.** App hiện ghi chú use case của danh mục (ví dụ "Gift Guides: mỗi H2 là 1 sản phẩm, ưu tiên mockup on-model").
3. **Nhập bài:** keyword, dán outline, ghi chú tuỳ chọn. Bấm **Phân tích**.
   - Code đóng băng preset hiện hành (Brandkit và Style Set đúng phiên bản) vào phiên.
   - AI bước 1 trả danh sách chỗ ảnh.
4. **Rà chỗ ảnh:** thêm, xoá, kéo đổi thứ tự, đổi số ảnh của gallery. Mỗi chỗ hiện lý do AI đề xuất.
5. **Cấu hình từng chỗ:** mục đích, format, Style Set (trong danh sách được cấp).
   - Chọn mục đích thuộc nhóm sản phẩm → hiện thêm: loại sản phẩm (từ danh sách của store), persona người mẫu hoặc "không người", design tham chiếu (chọn từ thư viện hoặc "gắn sau"), bối cảnh.
   - Không chỉnh gì = dùng mặc định của danh mục.
6. **Bấm Tạo prompt** → nhận Prompt Pack: thẻ từng ảnh theo thứ tự tạo, placeholder tô màu, danh sách file cần gắn. Copy hoặc export. Sau khi tạo ảnh, quay lại chấm từng prompt.

### Admin: setup một dự án mới

1. Tạo dự án (tên, domain, mô tả ngắn về store và khách hàng).
2. Tạo Brandkit: upload logo, nhập bảng màu HEX kèm vai trò, quy tắc nên/không nên. Xuất bản v1.
3. Tạo 2–4 Style Set (ví dụ "Lifestyle ấm", "Flat lay studio", "Illustration phẳng"). Bấm "Xem prompt mẫu" để thử.
4. Khai báo loại sản phẩm của store, upload ảnh nền sản phẩm, design mẫu, persona.
5. Dán danh sách danh mục blog; với mỗi danh mục ghi use case, mục đích ảnh mặc định, Style Set mặc định.
6. Mời Creator, tick dự án và Style Set được dùng.
7. Hằng tuần: xem Log và trang Chất lượng, sửa preset có tỉ lệ đạt thấp, xuất bản phiên bản mới.

## 6. Mô hình dữ liệu (Supabase Postgres)

Có 16 bảng. Preset (Brandkit, Style Set, Purpose template) lưu theo phiên bản không sửa đè; mỗi phiên làm việc chụp lại preset lúc tạo. Nhờ vậy Admin sửa preset không làm hỏng bài cũ và đo được phiên bản nào tốt hơn.

| Bảng | Cột chính | Ghi chú |
| --- | --- | --- |
| `profiles` | `id` (= auth.users), `full_name`, `role` (admin/creator), `status` (active/locked) | Tạo tự động khi user được mời |
| `projects` | `id`, `name`, `domain`, `description`, `audience`, `status` |  |
| `project_members` | `project_id`, `user_id`, `default_style_set_id` | Khoá của RLS |
| `member_style_sets` | `user_id`, `style_set_id` | Trống = dùng mọi Style Set active của dự án |
| `brandkits` | `id`, `project_id`, `version`, `palette` jsonb `[{name, hex, role}]`, `typography`, `voice`, `do` text\[\], `dont` text\[\], `logo_asset_ids` uuid\[\], `status` (draft/published/archived) | Mỗi dự án 1 bản published |
| `style_sets` | `id`, `project_id`, `name`, `version`, `tokens` jsonb `{medium, lighting, composition, mood, camera, color_treatment}`, `negative`, `example_asset_ids`, `status` |  |
| `categories` | `id`, `project_id`, `parent_id`, `name`, `slug`, `url`, `use_case_notes`, `default_purposes` text\[\], `default_style_set_id` | Taxonomy blog |
| `assets` | `id`, `project_id`, `type` (logo/product\_base/design/persona\_photo/example/reference), `name`, `storage_path`, `tags` text\[\], `meta` jsonb | File nằm trong Storage bucket private |
| `product_types` | `id`, `project_id`, `key` (hoodie, mug…), `name`, `mockup_notes`, `base_asset_id` | Danh sách sản phẩm của store |
| `personas` | `id`, `project_id`, `name`, `description` jsonb `{age_range, gender, look, style, vibe}`, `photo_asset_id` | Người mẫu/avatar |
| `purpose_templates` | `id`, `key`, `name`, `version`, `default_aspect`, `template` text, `required_refs` text\[\], `optional_refs` text\[\], `status` | Dùng chung mọi dự án |
| `sessions` | `id`, `project_id`, `category_id`, `creator_id`, `keyword`, `outline_md`, `notes`, `context_snapshot` jsonb, `status` (draft/planned/prompted) | 1 bài viết |
| `image_slots` | `id`, `session_id`, `code` (S1, S2…), `position`, `section_heading`, `purpose_key`, `aspect`, `count`, `style_set_id`, `options` jsonb, `ai_reason` |  |
| `prompts` | `id`, `slot_id`, `index` (S2-1, S2-2), `prompt_text`, `negative`, `references` jsonb, `depends_on` text\[\], `alt_text`, `filename`, `gen_order`, `model`, `template_versions` jsonb | 1 dòng = 1 ảnh |
| `feedback` | `prompt_id`, `user_id`, `verdict` (pass/fail), `reasons` text\[\], `note`, `result_url` | Dữ liệu chất lượng |
| `activity_logs` | `id`, `user_id`, `project_id`, `action`, `entity_type`, `entity_id`, `meta` jsonb, `tokens_in`, `tokens_out`, `cost_usd_est`, `created_at` | Chỉ ghi, không sửa |

**Chính sách RLS cốt lõi** (mẫu, áp cho mọi bảng có `project_id`):

```sql
-- Ham kiem tra quyen, dung lai o moi policy
create function public.is_admin() returns boolean
  language sql stable security definer set search_path = public as $$
  select exists (select 1 from profiles where id = auth.uid() and role = 'admin' and status = 'active')
$$;

create function public.is_member(p uuid) returns boolean
  language sql stable security definer set search_path = public as $$
  select exists (select 1 from project_members where project_id = p and user_id = auth.uid())
$$;

alter table categories enable row level security;
create policy "read: admin or member" on categories for select
  using (is_admin() or is_member(project_id));
create policy "write: admin only" on categories for all
  using (is_admin()) with check (is_admin());

-- sessions: creator chi thay va sua phien cua minh, trong du an duoc cap
create policy "own sessions" on sessions for all
  using (is_admin() or (creator_id = auth.uid() and is_member(project_id)))
  with check (creator_id = auth.uid() and is_member(project_id));
```

`activity_logs` chỉ được ghi từ server (service role), Creator không có quyền đọc hay sửa.

## 7. Prompt Engine

Pipeline có 6 bước. AI chỉ chạy ở bước B và D, nơi cần phán đoán; brand, màu, logo, format và placeholder do code ghép. Nhờ vậy prompt luôn đúng brand dù model thay đổi, và Admin chỉnh chất lượng bằng preset chứ không phải bằng code.

| Bước | Ai làm | Đầu vào | Đầu ra |
| --- | --- | --- | --- |
| A. Context Resolver | Code | Dự án, danh mục, user | `context_snapshot`: brandkit, Style Set được phép, purpose template, sản phẩm, persona, ví dụ mẫu (ghim bởi Admin) |
| B. Slot Planner | AI (lượt 1) | Keyword, outline, use case danh mục, danh mục mục đích ảnh | Danh sách chỗ ảnh JSON (schema bên dưới) |
| C. Creator chỉnh | Người | Danh sách chỗ ảnh | Chỗ ảnh đã chốt + tuỳ chọn từng chỗ |
| D. Scene Writer | AI (lượt 2, một lần cho cả bài) | Chỗ ảnh đã chốt, outline, keyword, persona/sản phẩm đã chọn | Mỗi ảnh: subject, bối cảnh, bố cục, góc máy, alt text, tên file |
| E. Prompt Assembler | Code | Scene + template + Style Set + Brandkit + format + tham chiếu | `prompt_text`, `negative`, `references`, `depends_on` |
| F. Validator | Code | Prompt Pack | Lỗi/cảnh báo: placeholder thiếu, vòng phụ thuộc, prompt quá dài, thiếu màu brand |

**Vì sao bước D chạy một lần cho cả bài:** model nhìn thấy mọi ảnh cùng lúc nên tránh lặp bố cục (yêu cầu "đa dạng"), rẻ hơn gọi từng ảnh. Bài > 20 ảnh thì chia lô 10 ảnh, truyền danh sách bố cục đã dùng sang lô sau.

### 7.1 Thứ tự ưu tiên của context

Lớp sau ghi đè lớp trước, trừ **Brandkit "không nên"** luôn thắng (không lớp nào gỡ được).

1. Mặc định hệ thống (format, negative chung)
2. Brandkit dự án
3. Danh mục (use case, mục đích mặc định)
4. Style Set được chọn
5. Purpose template của chỗ ảnh
6. Tuỳ chọn và ghi chú của Creator

### 7.2 Danh mục mục đích ảnh (bộ khởi đầu)

| Key | Dùng khi | Tỉ lệ mặc định | Tham chiếu bắt buộc | Tham chiếu tuỳ chọn |
| --- | --- | --- | --- | --- |
| `hero` | Ảnh đại diện bài | 16:9 | — | LOGO |
| `scene` | Minh hoạ bối cảnh/use case của một đoạn | 3:2 | — | — |
| `howto_step` | Một bước trong hướng dẫn | 4:3 | — | GEN bước trước |
| `product_flatlay` | Mockup sản phẩm trải phẳng/studio | 1:1 | PRODUCT, DESIGN | LOGO |
| `product_on_model` | Sản phẩm trên người mẫu | 4:5 | PRODUCT, DESIGN | MODEL |
| `product_lifestyle` | Sản phẩm trong bối cảnh dùng thật | 3:2 | PRODUCT, DESIGN | MODEL |
| `design_gallery` | Nhiều ý tưởng design cùng chủ đề | 1:1 | — | PRODUCT |
| `comparison` | So sánh 2–3 lựa chọn | 3:2 | — | PRODUCT |
| `pinterest_pin` | Pin dọc cho Pinterest | 2:3 | — | LOGO, GEN hero |

Ảnh cần chữ dài (infographic, bảng giá) không nằm trong bộ khởi đầu: model ảnh vẫn hay sai chữ. Prompt của ảnh có chữ luôn kèm cảnh báo "kiểm chữ trước khi dùng".

### 7.3 Placeholder

Cú pháp `[[LOẠI:KHOÁ]]`, ngoặc vuông kép để không đụng cú pháp của tool khác. Mỗi placeholder trong prompt phải có một dòng trong `references`.

| Placeholder | Nguồn | Khi tạo ảnh |
| --- | --- | --- |
| `[[REF:LOGO]]` | Thư viện dự án (tự biết file) | Gắn file logo |
| `[[REF:PRODUCT:hoodie]]` | Ảnh nền của loại sản phẩm | Gắn ảnh blank sản phẩm |
| `[[REF:MODEL:anna]]` | Ảnh persona | Gắn ảnh người mẫu (giữ khuôn mặt nhất quán giữa các ảnh) |
| `[[REF:DESIGN#1]]` | Creator cung cấp (chưa có file) | Gắn design thật; app liệt kê trong "File cần chuẩn bị" |
| `[[GEN:S1-1]]` | Ảnh do chính pack tạo trước | Tạo S1-1 trước, rồi gắn làm tham chiếu |
| `[[TEXT:headline]]` | Chữ cần hiện trên ảnh | Creator thay bằng chữ thật |

Code dựng đồ thị từ các `[[GEN:…]]`, sắp thứ tự topo để ra `gen_order`. Có vòng phụ thuộc thì Validator chặn và báo chỗ ảnh nào gây vòng.

### 7.4 Schema đầu ra của AI

Dùng structured output (JSON Schema) để model không trả sai định dạng. Code kiểm lại bằng Zod trước khi lưu.

```json
{
  "slot_plan": {
    "slots": [{
      "code": "S2",
      "section_heading": "Tartan Hoodie for Weekend Dads",
      "purpose_key": "product_on_model",
      "count": 1,
      "reason": "H2 giới thiệu một sản phẩm cụ thể, danh mục ưu tiên on-model"
    }]
  },
  "scenes": [{
    "prompt_index": "S2-1",
    "subject": "a man in his 50s wearing the hoodie",
    "setting": "backyard deck, autumn morning, coffee mug in hand",
    "composition": "three-quarter body, subject left third, shallow depth of field",
    "camera": "eye level, 50mm",
    "alt_text": "Dad wearing a red tartan hoodie on a backyard deck",
    "filename": "tartan-hoodie-gift-for-dad.webp"
  }]
}
```

### 7.5 Template ghép prompt (bước E)

Template do Admin sửa được (A5); biến trong `{}` do code điền, không qua AI.

```text
{purpose.opening}: {scene.subject}, {scene.setting}.
Composition: {scene.composition}. Camera: {scene.camera}.
Style: {style.medium}, {style.lighting}, {style.mood}, {style.color_treatment}.
Brand colors: primary {brand.primary_hex}, accent {brand.accent_hex}; keep the palette dominant.
{refs.instructions}
Brand rules: {brand.do}. Avoid: {brand.dont}.
Aspect ratio {slot.aspect}. No added text unless specified. {style.negative}
```

Ví dụ `refs.instructions` cho S2-1: *"Use \[\[REF:PRODUCT:hoodie\]\] as the exact garment shape. Print the artwork from \[\[REF:DESIGN#1\]\] on the chest, keep its colors unchanged. Keep the face consistent with \[\[REF:MODEL:dad-mark\]\]."*

### 7.6 Validator (bước F)

- Mọi placeholder trong prompt có trong `references`, và ngược lại.
- Tham chiếu bắt buộc của mục đích ảnh có mặt.
- Không có vòng `[[GEN:…]]`; không trỏ tới chỗ ảnh đã xoá.
- Prompt ≤ giới hạn ký tự cấu hình (giả định 2.000; kiểm lại theo tool tạo ảnh team dùng).
- Có ít nhất một màu HEX của brand trong prompt, trừ khi Style Set tắt.
- Không chứa từ trong danh sách cấm của Brandkit (tên đối thủ, nhân vật có bản quyền…).

## 8. Tích hợp AI provider

Một lớp adapter duy nhất nói chuẩn API tương thích OpenAI. Đổi giữa OpenAI và endpoint riêng chỉ là đổi `base_url` + tên model + tên biến môi trường chứa key.

**Cấu hình provider** (bảng `ai_providers`, cộng vào 16 bảng ở mục 6):

| Cột | Ví dụ | Ghi chú |
| --- | --- | --- |
| `name` | openai-main, custom-proxy |  |
| `base_url` | `https://api.openai.com/v1` hoặc URL endpoint riêng |  |
| `key_env` | `OPENAI_API_KEY`, `CUSTOM_AI_KEY` | Chỉ lưu **tên** biến môi trường, không lưu key |
| `model_planner` | model text cho bước B | Admin chọn |
| `model_scene` | model text cho bước D | Có thể khác bước B |
| `supports_json_schema` | true/false | False → dùng JSON mode + Zod kiểm + thử lại 1 lần |
| `daily_call_limit` | 300 | Chặn vượt chi phí |

**Cách gọi.** Dùng SDK `openai` cho Node với `baseURL` tuỳ chỉnh, gọi Chat Completions có `response_format` JSON Schema. Endpoint riêng phải tương thích OpenAI; nếu không, viết thêm một adapter cùng interface:

```ts
// lib/ai/provider.ts — interface chung cho moi provider
export interface AIProvider {
  generateJSON<T>(args: {
    model: string;
    system: string;
    user: string;
    schema: object;        // JSON Schema cua dau ra
    schemaName: string;
  }): Promise<{ data: T; tokensIn: number; tokensOut: number }>;
}
```

**Prompt hệ thống** của bước B và D lưu trong code, có số phiên bản, ghi vào `prompts.template_versions`. Ghim ví dụ mẫu của Admin được chèn vào bước D làm few-shot (tối đa 3 ví dụ cùng mục đích ảnh).

**Tạo ảnh trong app (lát L4, để sau).** Images API của OpenAI có endpoint tạo ảnh từ prompt (`/images/generations`) và endpoint tạo ảnh từ ảnh đầu vào kèm prompt (`/images/edits`); `/images/variations` đã bị đánh dấu deprecated ([OpenAI API reference](https://developers.openai.com/api/reference/cli/resources/images)). Endpoint edits là chỗ placeholder `[[REF:…]]` được thay bằng file thật. Số ảnh tham chiếu tối đa, kích thước, giá mỗi ảnh: **cần kiểm lại trên tài liệu chính thức** trước khi làm L4.

## 9. Kiến trúc & stack

Chọn **Next.js trên Vercel + Supabase** (Auth, Postgres có RLS, Storage). Theo thang kiến trúc của Project, app cần leo tới bậc "app có DB" vì cần đăng nhập hai vai, phân quyền theo dự án, kho file ảnh và log. Sheet/Apps Script không đáp ứng ba điều này một cách an toàn.

|  | Phương án | Được | Mất |
| --- | --- | --- | --- |
| **Recommended** | Next.js (App Router) + Supabase + gọi AI từ Route Handler/Server Action trên Vercel | Auth + RLS có sẵn; Storage private cho logo/design; Hung đã quen Vercel, Supabase; key AI chỉ nằm trên server | Phải viết UI admin; cần hiểu RLS; thời gian chạy tối đa của function phụ thuộc gói Vercel (cần kiểm gói đang dùng) |
| **Alternative** | Giống trên nhưng lát L1 cho Admin nhập dữ liệu thẳng bằng Supabase Table Editor, chưa làm UI admin | Có luồng Creator chạy thật sau \~1–2 tuần; đo được giá trị trước khi đầu tư UI admin | Admin nhập JSON bằng tay, dễ sai; chỉ hợp với 1–2 dự án đầu. **Đây là cách đề xuất cho L1.** |
| **Alternative** | Google Sheet làm nơi cấu hình preset + Apps Script gọi AI, dùng nội bộ 1–2 người | Rẻ nhất, làm trong vài ngày | Không có phân quyền theo dự án thật; timeout 6 phút của GAS; log và file ảnh rời rạc. Chỉ nên dùng để thử template prompt trước khi build |
| **Avoid** | Cho LLM tự viết cả prompt kể cả màu, logo, style mỗi lần | Code ít hơn | Brand trôi theo model, không kiểm chứng được, khó cải thiện có hệ thống |
| **Avoid** | Gọi API AI từ trình duyệt hoặc lưu key trong bảng DB | Nhanh | Lộ key cho mọi Creator |
| **Avoid** | Framework multi-agent, vector DB, đa tổ chức/billing ngay từ đầu | "Sẵn cho tương lai" | Nợ kỹ thuật, lệch Goal Card; 2 lượt gọi model có schema là đủ |

### Cấu trúc code đề xuất

```text
app/
  (auth)/login/
  (creator)/projects/[id]/sessions/[sid]/   # nhap bai -> slot -> prompt pack
  (admin)/admin/{projects,users,logs,quality,settings}/
  api/sessions/[sid]/plan/route.ts          # buoc A + B
  api/sessions/[sid]/prompts/route.ts       # buoc D + E + F
lib/
  ai/provider.ts, ai/openai-compatible.ts
  engine/resolver.ts, assembler.ts, validator.ts, graph.ts
  engine/schemas.ts                         # Zod + JSON Schema dung chung
  log.ts                                    # ghi activity_logs bang service role
supabase/migrations/*.sql                   # bang + RLS + seed purpose_templates
```

Biến môi trường trên Vercel: `NEXT_PUBLIC_SUPABASE_URL`, `NEXT_PUBLIC_SUPABASE_ANON_KEY`, `SUPABASE_SERVICE_ROLE_KEY` (chỉ server), `OPENAI_API_KEY`, `CUSTOM_AI_KEY` (nếu dùng endpoint riêng).

## 10. Bảo mật, log và cải thiện chất lượng

### Bảo mật

- Quyền chặn bằng RLS ở database; server kiểm lại `is_member` trước mọi lượt gọi AI.
- Key AI và `SUPABASE_SERVICE_ROLE_KEY` chỉ trong biến môi trường Vercel, không vào code, không vào DB, không ra trình duyệt.
- Storage bucket `assets` để private; trình duyệt chỉ nhận signed URL hết hạn ngắn (giả định 1 giờ).
- Tắt đăng ký công khai; tài khoản chỉ tạo qua lời mời của Admin.
- Giới hạn lượt gọi AI mỗi user mỗi ngày và độ dài outline (giả định 15.000 ký tự) để chặn chi phí bất thường.
- Outline và ghi chú của Creator đưa vào AI dưới dạng dữ liệu trong thẻ riêng, prompt hệ thống dặn model không làm theo lệnh nằm trong đó. Brand rules luôn do code ghép nên Creator không gỡ được.

### Log làm việc

Ghi từ server vào `activity_logs`, mỗi hành động một dòng:

| Action | Khi nào | `meta` chứa |
| --- | --- | --- |
| `session.create` | Tạo phiên | keyword, category |
| `slots.plan` | Chạy bước B | số chỗ ảnh, model, thời gian |
| `slots.edit` | Creator sửa chỗ ảnh | thêm/xoá/đổi gì |
| `prompts.generate` | Chạy bước D–F | số prompt, số cảnh báo Validator |
| `prompts.regenerate` | Sinh lại một phần | slot/prompt, yêu cầu chỉnh |
| `pack.export` | Copy cả pack hoặc export | định dạng |
| `feedback.submit` | Chấm prompt | verdict, reasons |
| `preset.publish` | Admin xuất bản preset | loại, phiên bản |
| `user.invite` / `user.lock` / `member.assign` | Admin quản lý user | user, dự án |

### Cải thiện chất lượng theo thời gian

Vòng lặp hàng tuần, dựa trên dữ liệu thật chứ không dựa cảm nhận:

1. Creator chấm từng prompt sau khi tạo ảnh: đạt/không đạt + tag lý do cố định (sai màu brand, sai hình sản phẩm, design bị biến dạng, bố cục xấu, không hợp nội dung đoạn, chữ lỗi, lặp ý với ảnh khác).
2. Trang Chất lượng tính tỉ lệ đạt theo (Style Set × phiên bản), (mục đích ảnh × phiên bản template), (dự án), và top lý do không đạt.
3. Admin sửa preset có tỉ lệ thấp nhất, xuất bản phiên bản mới; phiên cũ giữ nguyên để so sánh.
4. Admin ghim prompt đạt tốt thành ví dụ mẫu; bước D dùng làm few-shot cho cùng mục đích ảnh.
5. Chỉ so sánh hai phiên bản khi mỗi bên có đủ mẫu (giả định ≥ 20 prompt đã chấm); ít hơn thì trang ghi "chưa đủ dữ liệu".

## 11. Lộ trình build

Làm 5 lát, lát sau chỉ bắt đầu khi lát trước qua nghiệm thu với dữ liệu thật. L1 là lát mỏng chạy end-to-end: Creator đi hết luồng trên 1 dự án thật, Admin nhập preset qua Supabase.

| Lát | Phạm vi | Cách test | Nghiệm thu |
| --- | --- | --- | --- |
| L0 — Nền | Migration 16+1 bảng, RLS, Auth mời qua email, seed 9 purpose template, 1 dự án mẫu | Tạo 2 Creator, cấp mỗi người 1 dự án khác nhau; đăng nhập từng người, gọi thẳng API Supabase đọc bảng `categories` | Mỗi Creator chỉ đọc được danh mục dự án của mình; ghi vào `brandkits` bị từ chối |
| L1 — Luồng Creator | C1–C6, bước A–F của Prompt Engine, A9 qua env, log cơ bản | 1 dự án thật (ví dụ TartanVibes), 3 bài thật ở 3 danh mục khác nhau; dán prompt vào công cụ tạo ảnh team đang dùng | Pack sinh xong ≤ 60 giây/bài; Validator không còn lỗi; Hung chấm ≥ 70% ảnh dùng được lần đầu (ngưỡng L1, thấp hơn Goal Card) |
| L2 — Admin UI | A1–A4, A6, C7, C8 | Admin setup dự án thứ hai hoàn toàn qua UI, không mở Supabase | Setup 1 dự án mới ≤ 60 phút; Creator mới tự làm pack đầu tiên không cần hỏi |
| L3 — Chất lượng | C9, A5, A7, A8, phiên bản preset, ghim ví dụ mẫu | 2 tuần dùng thật, ≥ 50 prompt được chấm | Goal Card đạt: ≤ 10 phút/bài, ≥ 80% đạt; log và chi phí hiện đúng |
| L4 — Tạo ảnh trong app (để sau) | Gọi Images API, tự thay `[[REF:…]]` bằng file thật, chạy theo `gen_order`, lưu ảnh vào Storage | Chỉ làm khi Hung duyệt sau L3 | Quyết định riêng, cần tính chi phí ảnh trước |

### Việc cụ thể của L1

- [ ] Tạo dự án Supabase + Next.js, nối Vercel, khai báo biến môi trường
- [ ] Migration bảng + RLS (mục 6), seed purpose template (mục 7.2)
- [ ] Nhập tay 1 dự án thật: brandkit, 2 Style Set, 3–5 danh mục, loại sản phẩm, 1–2 persona
- [ ] Màn hình Creator: chọn dự án/danh mục → nhập bài → bảng chỗ ảnh → thẻ cấu hình → Prompt Pack
- [ ] `lib/engine`: resolver, assembler, validator, graph + test đơn vị với dữ liệu mẫu (outline 6 H2 → kỳ vọng 7–10 ảnh, không vòng phụ thuộc)
- [ ] Adapter OpenAI-compatible + 2 prompt hệ thống (bước B, D) có số phiên bản
- [ ] Copy/export JSON, CSV, Markdown
- [ ] Ghi `activity_logs` cho 4 action chính

## 12. Rủi ro, câu hỏi mở, để sau

### Rủi ro

| Rủi ro | Ảnh hưởng | Xử lý |
| --- | --- | --- |
| Model ảnh làm biến dạng design thật khi mockup | Ảnh sản phẩm sai so với hàng bán, ảnh hưởng CRO | Prompt mockup luôn dặn "giữ nguyên artwork"; tag lý do "design bị biến dạng" để đo; ảnh sản phẩm quan trọng vẫn nên ghép mockup bằng tool mockup chuyên dụng |
| Preset viết kém thì mọi prompt kém theo | Chất lượng hệ thống phụ thuộc Admin | Nút "Xem prompt mẫu" trước khi xuất bản; trang Chất lượng chỉ ra preset yếu |
| Function Vercel hết thời gian với bài nhiều ảnh | Creator chờ lâu hoặc lỗi | Chia lô 10 ảnh; stream tiến độ; kiểm giới hạn thời gian chạy của gói Vercel |
| Chi phí AI tăng khi team dùng nhiều | Vượt ngân sách | Giới hạn lượt/ngày, log chi phí ước tính mỗi lượt, sinh lại từng phần thay vì cả bài |
| Endpoint riêng không hỗ trợ JSON Schema | AI trả sai định dạng | Fallback JSON mode + Zod + thử lại 1 lần; cờ `supports_json_schema` |

### Câu hỏi mở cho Hung (có mặc định, trả lời "ok" = giữ mặc định)

1. Team tạo ảnh bằng công cụ nào? *Mặc định: ChatGPT/GPT Image, prompt ngôn ngữ tự nhiên. Nếu dùng Midjourney hay Ideogram thì thêm "hồ sơ tool đích" để đổi cú pháp.*
2. MVP có cần tạo ảnh ngay trong app không? *Mặc định: không, để L4.*
3. Outline lấy từ đâu? *Mặc định: dán markdown. Đọc từ Google Docs để sau.*

### Để sau (ngoài Goal Card)

- Tạo ảnh trong app qua Images API (L4).
- Hồ sơ cú pháp cho từng tool tạo ảnh.
- Đọc outline từ Google Docs; nhập danh mục từ sitemap của blog.
- Creator upload file riêng cho phiên (design khách gửi).
- Đẩy Prompt Pack sang Google Sheet của team.
- Gợi ý chỗ ảnh dựa trên bài đối thủ đang rank.

### Trạng thái

| Trạng thái | Việc |
| --- | --- |
| Xong | Goal Card, mô tả tính năng, ma trận quyền, data model + RLS mẫu, thiết kế Prompt Engine, chọn stack, lộ trình L0–L4 |
| Đang làm | Chờ Hung trả lời 3 câu hỏi mở và duyệt Goal Card |
| Để sau | Danh sách ngay trên |

Chưa có dòng code nào được chạy; mọi số liệu nghiệm thu là mục tiêu, chưa phải kết quả.

### Nguồn

- [OpenAI API reference — Images (generations, edits, variations)](https://developers.openai.com/api/reference/cli/resources/images), đọc ngày 2026-11-08
