# Research: Prompt Pack MVP (L0 + L1)

**Date**: 2026-11-08 · **Spec**: [spec.md](./spec.md) · **Plan**: [plan.md](./plan.md)

Mỗi mục: Decision / Rationale / Alternatives considered. Các quyết định đánh dấu **(ADR)** cần một ADR
ngắn trong `docs/adr/` khi worktree sở hữu bắt đầu hiện thực (constitution VIII).

## R1. Framework và runtime (ADR)

- **Decision**: Next.js 15 (App Router) + React 19 + TypeScript 5 (strict), Node.js 20 LTS, npm.
  Gọi AI trong Route Handlers (`runtime = 'nodejs'`), không dùng Edge.
- **Rationale**: stack đã chốt trong tài liệu gốc §9 và constitution; Route Handler cho hợp đồng HTTP rõ
  ràng để W6 làm UI song song với W5 bằng mock; SDK `openai` và service role chỉ chạy ở Node.
- **Alternatives**: Server Actions cho toàn bộ (gọn hơn nhưng khó mock và khó test hợp đồng khi chia
  worktree); pnpm (nhanh hơn nhưng thêm một công cụ trên máy Windows của team).

## R2. Supabase (ADR)

- **Decision**: Supabase Auth (email + mật khẩu và magic link, tắt sign-up công khai, mời bằng
  `auth.admin.inviteUserByEmail` hoặc Dashboard ở L1), Postgres + RLS, `@supabase/ssr` cho cookie session,
  `@supabase/supabase-js` cho service role phía server. Phát triển local bằng Supabase CLI (Docker).
  **Không dùng Storage ở L1** (spec Clarification: không lưu file).
- **Rationale**: RLS là ràng buộc B; local stack cho phép test RLS tự động không đụng project thật
  (constitution X cấm chạy migration lên remote khi chưa duyệt).
- **Alternatives**: Prisma + Postgres tự quản (mất RLS gắn với Auth); NextAuth (thêm hệ thống auth thứ hai).

## R3. Kiểm thử

- **Decision**: Vitest cho unit test (`lib/**`) và integration test RLS (chạy với Supabase local, tạo user
  test bằng service role, đăng nhập bằng anon key). Không có E2E browser ở L1; luồng UI kiểm bằng
  `quickstart.md` thủ công.
- **Rationale**: constitution V yêu cầu test tự động cho business logic và RLS; E2E đắt so với giá trị ở
  MVP (constitution I).
- **Alternatives**: Jest (chậm hơn với ESM/TS); Playwright E2E (để L2 nếu cần).

## R4. Zod và JSON Schema

- **Decision**: Zod 4, sinh JSON Schema bằng `z.toJSONSchema()`; một nguồn duy nhất
  `lib/engine/schemas.ts`. Schema đầu ra AI tuân theo ràng buộc strict structured output: mọi field
  `required`, `additionalProperties: false`, field tuỳ chọn dùng `nullable` thay vì bỏ trống.
- **Rationale**: tránh định nghĩa trùng (constitution II); strict mode giúp model không trả sai định dạng.
- **Alternatives**: Zod 3 + `zod-to-json-schema` (thêm dependency).

## R5. Gọi AI và chia lô (ADR)

- **Decision**: SDK `openai` với `baseURL` tuỳ chỉnh, Chat Completions + `response_format: json_schema`
  (strict). Provider không hỗ trợ → `json_object` + Zod + thử lại 1 lần. Bước D: ≤ 20 ảnh một lượt; > 20
  ảnh chia lô 10, chạy **tuần tự** và truyền danh sách `composition_tag` đã dùng sang lô sau. Route Handler
  đặt `maxDuration = 60`.
- **Rationale**: SC-003 ≤ 60 giây cho bài 7–10 ảnh = một lượt gọi; chạy tuần tự giữ được tính đa dạng giữa
  các lô.
- **Alternatives**: chạy lô song song (nhanh hơn nhưng lặp bố cục); streaming tiến độ (để sau, chỉ cần khi
  bài > 20 ảnh hay vượt thời gian).
- **Cần Hung kiểm**: gói Vercel đang dùng cho phép `maxDuration = 60` không; nếu không, hạ ngưỡng lô.

## R6. Cấu hình provider ở L1

- **Decision**: bảng `ai_providers` tồn tại từ L0 và là nguồn cấu hình (base_url, `key_env`, model từng
  bước, `supports_json_schema`, `daily_call_limit`, đơn giá). L1 nhập bằng seed/Dashboard; key nằm trong env
  theo tên ở `key_env`. Chỉ một provider `is_active = true`.
- **Rationale**: FR-038 (đổi provider không sửa code) và FR-037 (giới hạn tổng) cần chỗ lưu; không làm UI
  ở L1.
- **Alternatives**: chỉ dùng env (phải sửa env + redeploy để đổi model).

## R7. Style Guide danh mục

- **Decision**: lưu trong bảng `category_style_guides` (markdown đầy đủ + `params` jsonb parse từ
  frontmatter + version + status). Nguồn soạn thảo ở L1 là file `docs/style-guides/<project>/<category>.md`;
  script seed (W4) parse frontmatter bằng thư viện `yaml`, kiểm bằng Zod rồi upsert thành phiên bản mới.
- **Rationale**: app cần chụp phiên bản vào phiên (FR-018b); repo giữ lịch sử soạn thảo; Admin chưa có UI.
- **Alternatives**: đọc file trực tiếp lúc chạy (không có phiên bản trong DB, không hợp với Vercel).

## R8. Parse outline

- **Decision**: tự viết parser nhỏ cho heading ATX (`##`, `###`) trong `lib/engine/outline.ts`; bỏ qua
  heading trong code fence; H1 bị bỏ qua, H4+ gộp vào H3 gần nhất.
- **Rationale**: chỉ cần H2/H3 (FR-012); không cần cả bộ markdown parser (constitution I, III).
- **Alternatives**: `remark`/`mdast` (nặng, thừa).

## R9. UI

- **Decision**: Tailwind CSS 4 + shadcn/ui (component copy vào `components/ui`), `@dnd-kit/sortable` cho kéo
  đổi thứ tự chỗ ảnh, `lucide-react` cho icon. Không dùng thư viện state riêng: state phiên lấy từ API.
- **Rationale**: có sẵn component truy cập được bằng bàn phím (constitution IV), không phải tự viết
  (constitution III).
- **Alternatives**: MUI/Chakra (nặng, khó tuỳ biến); tự viết drag-and-drop.

## R10. Export

- **Decision**: formatter thuần trong `lib/export/` (JSON, CSV RFC 4180 có BOM UTF-8 để Excel đọc đúng tiếng
  Việt, Markdown). Không dùng thư viện CSV.
- **Rationale**: SC-010 (mở đúng trong bảng tính); vài chục dòng code, test được.

## R11. Ước tính chi phí

- **Decision**: `cost_usd_est = tokens_in × price_in_per_mtok / 1e6 + tokens_out × price_out_per_mtok / 1e6`,
  đơn giá lấy từ `ai_providers`. Đơn giá do Admin nhập, có thể để 0.
- **Rationale**: FR-035; không phụ thuộc bảng giá bên ngoài.

## R12. Tên bảng và cột tránh từ khoá SQL

- **Decision**: thực thể "Tham chiếu" lưu ở bảng `project_refs`; cột danh sách tham chiếu của prompt tên là
  `refs` (không dùng `references`, là từ khoá Postgres).
- **Rationale**: tránh phải quote ở mọi truy vấn.

## R13. Phạm vi bảng của L0

- **Decision**: chỉ tạo bảng cần cho L1 (xem `data-model.md`). Bảng `feedback` (C9) để migration của L3.
- **Rationale**: constitution I (MVP-first). Tài liệu gốc ghi "16+1 bảng" là cho toàn lộ trình.
