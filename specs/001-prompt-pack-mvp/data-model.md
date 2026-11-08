# Data Model: Prompt Pack MVP (L0 + L1)

**Owner**: W1 (migrations + RLS). Mọi worktree khác đọc file này; thay đổi data model = breaking change
(constitution VI), phải sửa file này trên `main` trước rồi mới sửa migration.

Quy ước: mọi bảng có `id uuid pk default gen_random_uuid()` (trừ khi ghi khác), `created_at timestamptz
default now()`, `updated_at` nếu có sửa. Kiểu `text[]` cho danh sách đơn giản, `jsonb` cho cấu trúc.

## 1. Bảng

### Tài khoản và quyền

| Bảng | Cột | Ràng buộc / ghi chú |
| --- | --- | --- |
| `profiles` | `id` (= `auth.users.id`, pk), `full_name`, `role` (`admin`\|`creator`), `status` (`active`\|`locked`) | Tạo bằng trigger khi user được mời; mặc định `creator`, `active` |
| `projects` | `slug` (unique), `name`, `domain`, `description`, `audience`, `status` (`active`\|`archived`) | |
| `project_members` | `project_id`, `user_id`, `default_style_set_key` (nullable) | pk (`project_id`, `user_id`) |
| `member_style_sets` | `project_id`, `user_id`, `style_set_key` | pk (`project_id`, `user_id`, `style_set_key`). Không có dòng nào cho (project, user) = dùng mọi Style Set `published` của dự án (FR-008) |

**Quy ước tham chiếu Style Set**: mọi nơi trỏ tới Style Set dùng `key` (text), không dùng `id`, vì một key có
nhiều phiên bản; luôn lấy bản `published` mới nhất của key đó tại thời điểm chụp snapshot.

### Preset (có phiên bản, bản `published` không sửa đè)

| Bảng | Cột | Ràng buộc / ghi chú |
| --- | --- | --- |
| `brandkits` | `project_id`, `version`, `palette` jsonb `[{name, hex, role}]` (nullable/rỗng), `typography` jsonb (nullable), `voice`, `do` text[], `dont` text[], `banned_terms` text[], `logo_ref_id` (nullable), `status` (`draft`\|`published`\|`archived`) | unique (`project_id`, `version`); partial unique: một `published` mỗi dự án |
| `style_sets` | `project_id`, `key`, `name`, `version`, `tokens` jsonb `{medium, lighting, composition, mood, camera, color_treatment}`, `negative`, `check_brand_hex` bool default false, `status` | unique (`project_id`, `key`, `version`) |
| `category_style_guides` | `project_id`, `category_id`, `version`, `status`, `markdown` text, `params` jsonb (frontmatter đã kiểm, xem [contracts/style-guide.md](./contracts/style-guide.md)), `source_path` | unique (`category_id`, `version`); partial unique: một `published` mỗi danh mục |
| `purpose_templates` | `key`, `name`, `version`, `default_aspect`, `group` (`general`\|`product`), `writer_guidance` text (hướng dẫn cho AI bước D), `ref_instructions` jsonb (mẫu câu hướng dẫn tham chiếu theo loại, code dùng ở bước E), `required_refs` text[], `optional_refs` text[], `status` | Dùng chung mọi dự án; unique (`key`, `version`); seed 9 mục |

### Dữ liệu dự án

| Bảng | Cột | Ràng buộc / ghi chú |
| --- | --- | --- |
| `categories` | `project_id`, `parent_id` (nullable), `name`, `slug`, `url`, `use_case_notes`, `default_purposes` text[], `default_style_set_key` (nullable) | unique (`project_id`, `slug`). Khi danh mục có Style Guide `published`, `params.default_purposes` của Style Guide được ưu tiên hơn cột này (resolver) |
| `project_refs` | `project_id`, `type` (`logo`\|`product_base`\|`design`\|`persona_photo`\|`example`), `key` (dùng trong placeholder, vd `hoodie`, `dad-mark`), `name`, `description`, `tags` text[], `link_url` (nullable) | unique (`project_id`, `type`, `key`). Không lưu file (L1) |
| `product_types` | `project_id`, `key`, `name`, `mockup_notes`, `base_ref_id` (nullable → `project_refs`) | unique (`project_id`, `key`) |
| `personas` | `project_id`, `key`, `name`, `description` jsonb `{age_range, gender, look, style, vibe}`, `photo_ref_id` (nullable) | unique (`project_id`, `key`) |

### Phiên làm việc

| Bảng | Cột | Ràng buộc / ghi chú |
| --- | --- | --- |
| `sessions` | `project_id`, `category_id`, `creator_id`, `keyword`, `outline_md` (≤ 15.000 ký tự, CHECK), `notes`, `context_snapshot` jsonb (nullable đến lần phân tích đầu; xem [contracts/engine.md](./contracts/engine.md) `ContextSnapshot`), `snapshot_at`, `status` (`draft`\|`planned`\|`prompted`) | |
| `image_slots` | `session_id`, `code` (`S1`…), `position` int, `section_heading` (nullable), `purpose_key`, `aspect`, `count` int (1–6), `style_set_key` (nullable, theo quy ước key), `options` jsonb `{product_type_key, persona_key \| "none", design_ref: {mode: "library"\|"later", key?}, setting, notes, depends_on_slot?}`, `ai_reason` (nullable) | unique (`session_id`, `code`) |
| `prompts` | `slot_id`, `session_id` (denormalize cho RLS/truy vấn), `index` (`S2-1`), `gen_order` int, `body` (phần AI viết), `prompt_text` (bản cuối), `original_text` (bản hệ thống sinh, giữ khi sửa tay), `negative`, `refs` jsonb `[RefEntry]`, `depends_on` text[], `alt_text`, `filename`, `warnings` jsonb `[Issue]`, `is_manual_edit` bool, `model`, `template_versions` jsonb | unique (`session_id`, `index`) |

### Hệ thống

| Bảng | Cột | Ràng buộc / ghi chú |
| --- | --- | --- |
| `ai_providers` | `name` (unique), `base_url`, `key_env`, `model_planner`, `model_writer`, `supports_json_schema` bool, `daily_call_limit` int, `price_in_per_mtok` numeric, `price_out_per_mtok` numeric, `is_active` bool | Partial unique: một `is_active`. Server đọc bằng service role (`lib/api`) khi Creator gọi AI; client không bao giờ đọc |
| `activity_logs` | `id` bigint identity, `user_id`, `project_id`, `session_id` (nullable), `action`, `ok` bool, `entity_type`, `entity_id`, `meta` jsonb, `tokens_in`, `tokens_out`, `cost_usd_est` numeric, `duration_ms`, `created_at` | Chỉ insert bằng service role. `ok = false` cho lượt AI thất bại (vẫn ghi tokens đã tiêu). Giới hạn tổng ngày đếm dòng `ok = true` của `slots.plan`/`prompts.generate` |

Bảng `feedback` (C9) **không** thuộc L1 (research R13).

### Hàm SQL (owner W1, trong migration)

| Hàm | Mục đích |
| --- | --- |
| `replace_session_slots(p_session uuid, p_slots jsonb)` | Xoá slot + prompt cũ của phiên và chèn slot mới trong một transaction; `security invoker` (RLS vẫn áp dụng) |
| `replace_session_prompts(p_session uuid, p_prompts jsonb)` | Xoá prompt cũ và chèn prompt mới trong một transaction; `security invoker` |

### Dữ liệu test (owner W1, `supabase/seed.sql`, chỉ local)

User `admin@test.local`, `creator-a@test.local`, `creator-b@test.local` (mật khẩu chỉ dùng local), 2 dự án test.
W4 gán quyền Print Your Wear cho các user này theo email.

## 2. RLS (ràng buộc B)

Hàm `is_admin()`, `is_member(project uuid)` như tài liệu gốc §6; thêm `is_active_user()` (status =
`active`) — mọi policy đều AND với `is_active_user()`.

| Bảng | SELECT | INSERT / UPDATE / DELETE |
| --- | --- | --- |
| `profiles` | chính mình, hoặc admin | admin sửa mọi dòng; user chỉ sửa `full_name` của chính mình (trigger chặn đổi `role`, `status`) |
| `projects`, `categories`, `brandkits`, `style_sets`, `category_style_guides`, `project_refs`, `product_types`, `personas` | admin, hoặc `is_member(project_id)`; với preset chỉ dòng `published` cho creator | admin |
| `project_members`, `member_style_sets` | admin, hoặc dòng của chính mình | admin |
| `purpose_templates` | mọi user active (chỉ `published` cho creator) | admin |
| `sessions` | admin, hoặc `creator_id = auth.uid()` | INSERT: `creator_id = auth.uid()` AND (`is_admin()` OR `is_member(project_id)`). UPDATE/DELETE: `creator_id = auth.uid()` AND (`is_admin()` OR `is_member(project_id)`) — admin không sửa/xoá phiên người khác (FR-007) |
| `image_slots`, `prompts` | qua phiên cha (`exists` trên `sessions` với cùng điều kiện SELECT) | qua phiên cha với điều kiện UPDATE của `sessions` |
| `ai_providers` | admin | admin |
| `activity_logs` | admin | không ai (chỉ service role, bỏ qua RLS) |

## 3. Trạng thái phiên

```text
draft --Phân tích (lần đầu: chụp snapshot)--> planned --Tạo prompt--> prompted
planned/prompted --Phân tích lại (giữ snapshot, xoá slot+prompt sau xác nhận)--> planned
prompted --sửa slot--> planned (prompt cũ giữ đến lần Tạo prompt kế tiếp, UI cảnh báo "pack đã cũ")
```

## 4. Quy tắc kiểm (đối chiếu FR)

- `outline_md` có ≥ 1 H2 và ≤ 15.000 ký tự (FR-013) — kiểm ở engine + CHECK độ dài ở DB.
- `image_slots.purpose_key` ∈ khoá `purpose_templates` trong snapshot (FR-015, FR-019).
- `prompts.prompt_text` ≤ 2.000 ký tự là **cảnh báo/lỗi Validator**, không phải CHECK DB (sửa tay vẫn lưu được).
- Placeholder trong `prompt_text` ↔ `refs` một-một (FR-023) — Validator.
