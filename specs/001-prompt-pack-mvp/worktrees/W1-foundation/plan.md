# Implementation Plan: W1 Nền

> **Ghi chú người điều phối (2026-11-08)**: hợp đồng đã được điều chỉnh sau khi plan này được viết — xem
> mục "Nhật ký điều chỉnh hợp đồng" trong [plan tổng](../../plan.md). Chỗ nào plan này lệch hợp đồng thì
> **hợp đồng thắng**; các mục "Cần xác nhận" bên dưới đã được trả lời ở đó. Cập nhật plan này trước khi chạy
> `/speckit-tasks`.

**Branch**: `001-w1-foundation` | **Date**: 2026-11-08 | **Spec**: [spec.md](./spec.md)

**Bối cảnh chung**: [plan tổng](../../plan.md) · [research](../../research.md) ·
[data-model](../../data-model.md) · [contracts](../../contracts/) · [quickstart](../../quickstart.md)

## Summary

Dựng khung Next.js dùng chung (G0) rồi nền dữ liệu Supabase local với RLS, đăng nhập bằng lời mời, seed 9
mục đích ảnh và bộ test quyền (G1). W1 là worktree duy nhất được thêm migration ở L1.

## Technical Context

**Language/Version**: TypeScript 5 strict, Node.js 20 LTS, npm

**Primary Dependencies (cài hết ở G0)**: next@15, react@19, react-dom@19, `@supabase/ssr`,
`@supabase/supabase-js`, zod@4, openai, yaml, tailwindcss@4 + shadcn/ui (init), `@dnd-kit/core`,
`@dnd-kit/sortable`, lucide-react; dev: typescript, vitest, eslint (config Next), `supabase` CLI, tsx

**Storage**: Supabase Postgres local (Docker); không Storage

**Testing**: Vitest — `tests/rls/**` chạy với Supabase local (anon key đăng nhập từng user, service role để
tạo user test)

**Constraints**: không migration/deploy lên remote; không secret trong repo

## Constitution Check — PASS

| Mục | W1 tuân thủ |
| --- | --- |
| I MVP | Chỉ bảng L1 theo data-model (không `feedback`, không Storage bucket) |
| II Module | `lib/supabase/*` là lớp duy nhất tạo client; `admin.ts` có `import 'server-only'` |
| V Test | `tests/rls` là cổng G1 |
| VI Breaking | Migration chỉ theo data-model.md; lệch → báo người điều phối, không tự đổi |
| VIII ADR | ADR-0001 stack, ADR-0002 Supabase/RLS |
| IX Secret | `.gitignore` sẵn có chặn `.env*`; `.env.example` chỉ tên biến |
| X Duyệt | Chỉ commit trên `001-w1-foundation`; không push/merge; không `supabase link`/`db push` |
| B RLS | Mọi bảng dự án + bảng con có policy trong cùng migration; `is_active_user()` AND mọi policy |
| C Phiên bản | Unique/partial unique cho `published` theo data-model §1 |

## Project Structure (chỉ đường dẫn W1 sở hữu)

```text
package.json, package-lock.json, tsconfig.json, next.config.ts, eslint.config.mjs,
postcss.config.mjs, components.json, vitest.config.ts, .env.example
middleware.ts
app/layout.tsx (khung), app/page.tsx (redirect), app/(auth)/login/page.tsx, app/auth/callback/route.ts
lib/supabase/server.ts, client.ts, admin.ts, middleware.ts
supabase/config.toml, supabase/migrations/*.sql, supabase/seed.sql
tests/rls/*.test.ts, tests/rls/helpers.ts
docs/adr/0001-stack.md, docs/adr/0002-supabase-rls.md
CLAUDE.md (chỉ phần Commands)
```

Thư mục rỗng tạo sẵn (giữ bằng `.gitkeep`) cho chủ sở hữu khác: `lib/engine`, `lib/ai`, `lib/export`,
`lib/api`, `app/(creator)`, `app/api`, `components/ui`, `components/creator`, `scripts`.

## Pha thực thi

### Pha G0 — Skeleton (làm trước, merge sớm)

1. Khởi tạo Next.js 15 App Router + TS strict (`noUncheckedIndexedAccess`), alias `@/*`.
2. Tailwind 4 + `shadcn init` (chưa thêm component; W6 thêm). ESLint config Next. Vitest + 1 test mẫu.
3. Cài toàn bộ dependency ở Technical Context (Technical Context của plan tổng, cộng `tsx`; tránh xung đột `package.json`).
4. Scripts npm: `dev`, `build`, `lint`, `typecheck` (`tsc --noEmit`), `test` (`vitest run`),
   `test:rls` (`vitest run tests/rls`), `db:start` (`supabase start`), `db:reset` (`supabase db reset`).
5. `.env.example`: `NEXT_PUBLIC_SUPABASE_URL`, `NEXT_PUBLIC_SUPABASE_ANON_KEY`, `SUPABASE_SERVICE_ROLE_KEY`,
   `OPENAI_API_KEY`, `CUSTOM_AI_KEY` (chỉ tên, giá trị rỗng).
6. Cấu trúc thư mục + `.gitkeep`; cập nhật `CLAUDE.md` mục Commands bằng lệnh thật.

**Bàn giao G0**: build/typecheck/lint/test xanh → báo người điều phối → Hung duyệt merge vào `main` →
W2, W3, W6 bắt đầu (rebase lên `main`).

### Pha G1 — Dữ liệu, quyền, đăng nhập

1. `supabase init` → `config.toml`: tắt `enable_signup`, bật email + magic link, site_url `localhost:3000`.
2. Migration theo [data-model.md](../../data-model.md) (chia file theo nhóm: auth/quyền, preset, dự án,
   phiên, hệ thống): kiểu enum bằng CHECK, unique/partial unique, CHECK `outline_md` ≤ 15.000.
3. Hàm `is_admin()`, `is_member(uuid)`, `is_active_user()` (security definer, `search_path = public`);
   trigger tạo `profiles` khi có `auth.users` mới (role `creator`, status `active`).
4. RLS từng bảng theo [data-model.md §2](../../data-model.md#2-rls-ràng-buộc-b); bảng con (`image_slots`,
   `prompts`) đi qua `sessions`; `activity_logs` bật RLS, không policy ghi (chỉ service role).
5. `seed.sql`: 9 `purpose_templates` published (key, tỉ lệ, `group`, `required_refs`/`optional_refs` theo tài
   liệu gốc §7.2; `writer_guidance` 2–4 câu tiếng Anh mỗi mục; `ref_instructions` theo loại PRODUCT, DESIGN,
   MODEL, LOGO, GEN dựa trên ví dụ §7.5); 1 dòng `ai_providers` mẫu `is_active` dùng `key_env =
   OPENAI_API_KEY`, đơn giá 0. Không seed user (test tự tạo).
6. `lib/supabase/server.ts` (cookie, RLS người gọi), `client.ts` (browser), `admin.ts` (service role,
   server-only); `middleware.ts` làm mới session, chặn route `(creator)` và `api` khi chưa đăng nhập.
7. `(auth)/login`: email + mật khẩu và gửi magic link; `auth/callback` đổi code lấy session; tài khoản
   `locked` → đăng xuất kèm thông báo. Giao diện tối giản (W6 có thể làm đẹp sau).
8. `tests/rls`: helper tạo user bằng service role, đăng nhập bằng anon key, dọn dữ liệu sau test.
9. ADR-0001, ADR-0002; cập nhật `CLAUDE.md` nếu lệnh thay đổi.

### Danh sách test RLS (`npm run test:rls`)

| # | Kiểm | Map |
| --- | --- | --- |
| 1 | Creator A đọc `categories`, `brandkits`, `style_sets`, `category_style_guides`, `project_refs`, `product_types`, `personas` của dự án Y → rỗng | SC-001, FR-004 |
| 2 | Creator A đọc preset `draft` của chính dự án X → rỗng | FR-004 |
| 3 | Creator A insert/update/delete vào mọi bảng cấu hình và `purpose_templates` → bị từ chối | SC-002, FR-005 |
| 4 | Creator A tạo phiên ở Y → bị từ chối; ở X → được | FR-006 |
| 5 | Creator A không thấy phiên, slot, prompt của Creator B | FR-006 |
| 6 | Admin (không là thành viên X) tạo phiên của mình ở X → được | FR-007 |
| 7 | Admin đọc phiên/slot/prompt của A → được; update/delete → 0 dòng bị ảnh hưởng | FR-007 |
| 8 | Creator đọc/insert `activity_logs`, đọc `ai_providers` → rỗng/bị từ chối | FR-036 |
| 9 | User `locked` đọc mọi bảng → rỗng | FR-003 |
| 10 | `member_style_sets` của A chỉ thấy dòng của chính A | FR-008 |
| 11 | Tạo user qua anon `signUp` → bị từ chối | FR-001 |

**Bàn giao G1**: `db:reset` + `test:rls` xanh → Hung duyệt merge → W4, W5 rebase và bắt đầu.

## Rủi ro

| Rủi ro | Xử lý |
| --- | --- |
| Docker Desktop/WSL2 trên Windows chậm hoặc không chạy | Kiểm `npx supabase start` ngay đầu G1; nếu không chạy, báo người điều phối (phương án: Supabase project dev riêng — cần Hung duyệt) |
| Gói Vercel không cho `maxDuration = 60` | Không chặn W1; ghi vào ADR-0001 là điểm cần Hung kiểm |
| Phiên bản Next/Tailwind/shadcn mới đổi cách init | Ghim phiên bản cụ thể trong `package.json`, ghi vào ADR-0001 |
| RLS sai làm lộ dữ liệu | Test 1–11 là cổng bắt buộc; không merge khi còn đỏ |

## Việc cần Hung làm / duyệt

- Duyệt merge G0 và G1 vào `main` (từng lần).
- Không tạo project Supabase/Vercel thật, không `supabase link`/`db push`, không đặt biến môi trường trên Vercel
  cho tới khi Hung duyệt riêng.
- Xác nhận gói Vercel (ảnh hưởng `maxDuration`).

## Complexity Tracking

Không có vi phạm.
