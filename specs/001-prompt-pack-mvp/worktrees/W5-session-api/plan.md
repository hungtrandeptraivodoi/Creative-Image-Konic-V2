# Implementation Plan: W5 — Phiên làm việc & API

> **Ghi chú người điều phối (2026-11-08)**: hợp đồng đã được điều chỉnh sau khi plan này được viết — xem
> mục "Nhật ký điều chỉnh hợp đồng" trong [plan tổng](../../plan.md). Chỗ nào plan này lệch hợp đồng thì
> **hợp đồng thắng**; các mục "Cần xác nhận" bên dưới đã được trả lời ở đó. Cập nhật plan này trước khi chạy
> `/speckit-tasks`.

**Branch**: `001-w5-session-api` | **Date**: 2026-11-08 | **Spec**: [spec.md](./spec.md)

**Input**: [spec.md](./spec.md) (phạm vi W5), bối cảnh chung [../../plan.md](../../plan.md)

## Summary

Hiện thực các Route Handler trong [contracts/api.md](../../contracts/api.md): nối DB (RLS của người gọi) với
engine thuần (W2) và AI (W3), lưu phiên/slot/prompt, kiểm quyền và giới hạn tổng của provider, ghi
`activity_logs` bằng service role. Không chứa logic ghép prompt; mọi quyết định nội dung nằm ở `lib/engine`.

## Technical Context

**Language/Version**: TypeScript 5 strict, Node 20 · **Primary Dependencies**: Next.js 15 Route Handlers,
`@supabase/ssr`, `@supabase/supabase-js`, Zod 4 (schema từ `lib/engine/schemas.ts`)

**Storage**: Supabase Postgres (RLS) · **Testing**: Vitest + Supabase local + provider giả (`lib/ai/fake.ts`)

**Performance Goals**: `POST …/prompts` ≤ 60 giây cho 7–10 ảnh · **Constraints**: `runtime = 'nodejs'`,
`maxDuration = 60` cho `plan` và `prompts`; service role chỉ trong `lib/log.ts` và đọc `ai_providers`

## Constitution Check

**PASS** — I: không streaming, không queue; II: route mỏng, logic ở `lib/engine`, I/O ở `lib/api`; III: dùng
lại schema/engine/AI factory, không viết lại; V: test route cho mọi nhánh lỗi; VI/VII: bám đúng
`contracts/api.md`, lệch thì báo người điều phối; IX: không đọc key ngoài factory W3; X: chỉ commit trên
nhánh W5. Ràng buộc B (kiểm thành viên trước AI), C (snapshot một lần), D (`issues` từ Validator), E (log +
giới hạn tổng) là trọng tâm của W5.

## Project Structure (sở hữu của W5)

```text
app/api/sessions/route.ts                       # POST tạo phiên
app/api/sessions/[sid]/route.ts                 # GET, PATCH intake
app/api/sessions/[sid]/plan/route.ts            # POST bước A+B
app/api/sessions/[sid]/slots/route.ts           # PUT
app/api/sessions/[sid]/prompts/route.ts         # POST bước D+E+F
app/api/sessions/[sid]/export/route.ts          # POST
app/api/prompts/[pid]/route.ts                  # PATCH sửa tay / khôi phục
lib/api/guard.ts      # requireUser, requireSessionAccess(sid, mode), requireProjectAccess(pid)
lib/api/errors.ts     # ApiError(code,status,message,issues?) + toResponse()
lib/api/load.ts       # loadSnapshotSource(projectId, categoryId, userId), loadSession(sid)
lib/api/persist.ts    # replaceSlots, replacePrompts, updatePrompt (xem Rủi ro R-W5-1)
lib/api/quota.ts      # assertProviderQuota(provider)
lib/log.ts            # logActivity(...), estimateCost(...)
tests/api/**          # test route
```

## Thiết kế

### `lib/api/guard.ts`

- `requireUser()`: Supabase server client (cookie) → `auth.getUser()`; không có → 401
  `UNAUTHENTICATED`. Đọc `profiles` → `status = locked` → 403 `FORBIDDEN`.
- `requireProjectAccess(projectId)`: admin, hoặc có dòng `project_members` → ngược lại 403.
- `requireSessionAccess(sid, 'read' | 'write')`: đọc phiên qua client RLS (không thấy → 404). `write` yêu
  cầu `creator_id = user.id` và (`is_admin` hoặc thành viên) — admin không sửa phiên người khác (FR-007) → 403.
- Gọi `requireProjectAccess` lại ngay trước mọi lượt gọi AI (FR-009), không dùng kết quả cache.

### `lib/api/errors.ts`

Bảng mã lỗi đúng [contracts/api.md](../../contracts/api.md); `message` tiếng Việt. Map `AIError` của W3:
`BAD_OUTPUT` → 502 `AI_BAD_OUTPUT`; `TIMEOUT`/`UPSTREAM` → 502 `AI_UPSTREAM`; `CONFIG` → 503
`NO_ACTIVE_PROVIDER`. Lỗi Zod body → 400 `VALIDATION`.

### `lib/api/load.ts`

`loadSnapshotSource` đọc bằng client RLS của người gọi (Creator chỉ thấy preset `published` và Style Set
được phép): project, brandkit `published`, category + Style Guide `published` (có thể null), style sets
(lọc theo `member_style_sets`; rỗng = mọi bản published), `default_style_set_id`, purposes `published`,
product types, personas, `project_refs`. Trả `SnapshotSource` cho `buildSnapshot` (W2). Admin tạo phiên ở dự
án không phải thành viên → mọi Style Set published.

### `lib/log.ts` (server-only)

`logActivity({ userId, projectId, sessionId, action, entityType, entityId, meta, usage? })` dùng client
service role (`lib/supabase/admin.ts`); `estimateCost(usage, provider)` theo research R11. Lỗi ghi log không
làm hỏng request chính nhưng phải `console.error` (không nuốt im lặng).

### `lib/api/quota.ts`

Đọc provider `is_active` (service role; không có → 503). Đếm `activity_logs` hôm nay (UTC) có
`action in ('slots.plan','prompts.generate')` và `meta->>'ok' = 'true'`; ≥ `daily_call_limit` → 429
`PROVIDER_DAILY_LIMIT`. Một lần bấm = một lượt (chia lô, thử lại không cộng thêm).

### Luồng từng route

| Route | Luồng |
| --- | --- |
| `POST /api/sessions` | Zod body → `requireProjectAccess` → kiểm category thuộc project → insert `sessions` (`draft`, `creator_id = user`) → log `session.create {keyword: null, categoryId}` |
| `GET /api/sessions/:sid` | `requireSessionAccess(read)` → phiên + slots + prompts (theo `gen_order`) → `issues` = `validatePack` nếu có snapshot + prompts; `snapshotSummary` (version brandkit/style guide, có/không palette) |
| `PATCH /api/sessions/:sid` | `write` → Zod → `parseOutline` (W2); ≥ 15.000 ký tự hoặc 0 H2 vẫn lưu nhưng trả `outline` để UI chặn nút Phân tích → trả `{session, outline}` |
| `POST …/plan` | `write` → kiểm outline hợp lệ (FR-013, 400 nếu không) → đã có slot/prompt và `confirmReplace ≠ true` → 409 `{slots, prompts}` → `requireProjectAccess` (FR-009) → quota → snapshot: có rồi thì dùng lại, chưa có thì `loadSnapshotSource` + `buildSnapshot` + lưu `snapshot_at` (C, FR-014) → `buildPlannerMessages` → `provider.generateJSON` (schema SlotPlan) → kiểm sau Zod (contracts/ai.md) → `replaceSlots` (xoá prompts cũ) → status `planned` → log `slots.plan {ok, slotCount, model, attempts}` + usage. AI lỗi → log `ok:false`, trả 502, dữ liệu cũ giữ nguyên |
| `PUT …/slots` | `write` → Zod `SlotDraft[]` → kiểm `purpose_key`/style set thuộc snapshot → `replaceSlots`; nếu phiên `prompted` → status `planned` (prompt cũ giữ, UI báo "pack đã cũ") → trả `issues` cấp slot (vd GEN tới slot đã xoá) → log `slots.edit {added, removed, changed}` |
| `POST …/prompts` | `write` → có snapshot + slot (400 nếu không) → có prompt `is_manual_edit` và `confirmReplace ≠ true` → 409 → FR-009 → quota → `splitIntoBatches` → **tuần tự** từng lô: `buildWriterMessages(…, usedCompositions)` → `generateJSON` (WrittenPack) → cộng dồn usage, nối `composition_tag` → `assemblePack(snap, slots, written, {model})` → `replacePrompts` → status `prompted` → log `prompts.generate {ok, promptCount, errorCount, warningCount, batches}` + usage tổng → trả `{session, prompts, issues}` (có lỗi vẫn 200) |
| `PATCH /api/prompts/:pid` | Đọc prompt → `requireSessionAccess(write)` qua session → `{promptText}`: `is_manual_edit = true`, giữ `original_text`; `{restoreOriginal}`: về `original_text`, `is_manual_edit = false` → `validatePrompt` → lưu `warnings` → trả `{prompt}` |
| `POST …/export` | `read` (chủ phiên hoặc admin) → có prompts (400 nếu không) → `lib/export` (W6) theo `format` → file với `Content-Disposition` hoặc `{text}` cho clipboard → log `pack.export {format}` |

### Tránh vượt `maxDuration = 60`

- Bài ≤ 20 ảnh: đúng 1 lượt gọi bước D (R5). Đặt timeout SDK ≈ 50 giây qua factory W3; quá hạn → 502.
- Bài > 20 ảnh: trước khi chạy lô kế tiếp, nếu thời gian đã dùng + thời lượng trung bình một lô > 55 giây
  → dừng, trả 502 `AI_UPSTREAM` với thông điệp đề nghị giảm số ảnh; ghi `meta.partial`. (Streaming/chạy nền
  để sau — R5.)
- Không đọc DB thừa trong vòng lô; snapshot đọc một lần.

## Chiến lược test

- `tests/api/*.test.ts`: gọi trực tiếp hàm handler (`POST(request, {params})`) với cookie phiên của user test
  tạo bằng service role trên Supabase local (dùng lại helper của `tests/rls` từ W1). Provider: trỏ
  `ai_providers` test tới factory trả `lib/ai/fake.ts` (biến env `AI_FAKE=1` do W3 cung cấp — cần xác nhận).
- Ca bắt buộc: mỗi mã lỗi trong contract; snapshot không đổi sau phân tích lại; 409 phân tích lại và tạo lại
  khi có sửa tay; admin tạo phiên ở dự án không phải thành viên được, sửa phiên người khác bị 403; quota chặn
  ở lượt thứ `limit+1`; chia lô 25 ảnh tạo 3 lô và `usedCompositions` được truyền; AI lỗi không làm mất slot
  cũ; đủ dòng log và Creator không đọc được log.
- Thủ công: đo SC-003 với provider thật (Hung duyệt key), ghi số giây thật vào báo cáo.

## Bàn giao cho W6 (trước khi W5 merge)

Đề xuất đơn giản nhất: W5 commit sớm `tests/fixtures/api/*.json` (response mẫu cho mọi route, đúng
contract) và W6 dùng một mock client `lib/api-client` chuyển sang đọc fixture khi
`NEXT_PUBLIC_API_MOCK=1`. Không dựng mock server riêng. Vì fixture và client nằm ở thư mục khác nhau,
người điều phối cần chốt ai sở hữu `tests/fixtures/api/` (đề xuất: W5 sở hữu, W6 chỉ đọc) và đưa lên `main`
sớm để W6 rebase.

## Rủi ro

| ID | Rủi ro | Đề xuất | Cần ai |
| --- | --- | --- | --- |
| R-W5-1 | supabase-js không có transaction; "xoá slot/prompt cũ rồi chèn mới" có thể để phiên dở dang nếu lỗi giữa chừng | Hàm SQL `replace_session_slots(sid, slots jsonb)` và `replace_session_prompts(sid, prompts jsonb)` (`security invoker` để RLS vẫn áp) chạy trong một transaction, gọi qua `rpc` | **Người điều phối xác nhận**: đây là migration, thuộc W1 (`supabase/**`), phải thêm vào `data-model.md`. Phương án dự phòng không cần migration: chèn bản mới gắn `batch_id` rồi mới xoá bản cũ |
| R-W5-2 | Gói Vercel không cho 60 giây | Hạ ngưỡng 1 lượt xuống ≤ 10 ảnh | Hung kiểm gói |
| R-W5-3 | Đếm quota bằng `activity_logs` phụ thuộc log ghi thành công | Ghi log trước khi trả response; log lỗi thì vẫn tính lượt qua `meta.ok` | — |
| R-W5-4 | `requireSessionAccess` cho admin xem phiên người khác cần RLS SELECT của admin trên `image_slots`/`prompts` | Đã có trong data-model §2; test xác nhận | W1 |

## Complexity Tracking

Không có vi phạm.
