# Implementation Plan: Prompt Pack MVP (L0 + L1) — Plan tổng

**Branch**: `001-prompt-pack-mvp` (plan tổng trên `main`) | **Date**: 2026-11-08 | **Spec**: [spec.md](./spec.md)

**Input**: Feature specification from `specs/001-prompt-pack-mvp/spec.md`

**Vai trò của file này**: bối cảnh chung cho 6 worktree W1–W6. File này chốt kiến trúc, hợp đồng và ranh
giới sở hữu; **mỗi worktree có plan thực thi riêng** trong `worktrees/<Wn>-<tên>/plan.md`. Worktree không
được tự đổi hợp đồng ở đây; muốn đổi thì báo người điều phối để sửa trên `main` (constitution VI, VII).

## Summary

Creator chọn dự án + danh mục, dán keyword/outline, hệ thống (B) dùng AI đề xuất chỗ ảnh, Creator chỉnh,
rồi (D) AI viết phần mô tả của mọi prompt dựa trên Style Guide danh mục; code (E) gắn phần bắt buộc
(không nên, negative, placeholder, tỉ lệ, màu brand nếu có) và (F) kiểm lỗi; Creator sửa tay hoặc tạo lại,
rồi copy/export. Nền: Supabase Auth mời qua email + Postgres RLS theo dự án; L1 không lưu file ảnh.

## Technical Context

**Language/Version**: TypeScript 5 (strict), Node.js 20 LTS

**Primary Dependencies**: Next.js 15 App Router, React 19, `@supabase/ssr`, `@supabase/supabase-js`, Zod 4,
`openai` (Node SDK), `yaml`, Tailwind CSS 4 + shadcn/ui, `@dnd-kit/sortable`, `lucide-react` (research R1–R9)

**Storage**: Supabase Postgres với RLS; không dùng Storage ở L1

**Testing**: Vitest (unit cho `lib/**`, integration RLS với Supabase local) (R3)

**Target Platform**: Vercel (Node runtime) + Supabase; trình duyệt desktop, dùng được từ 360px

**Project Type**: web application (một project Next.js)

**Performance Goals**: Pack cho bài 7–10 ảnh ≤ 60 giây (SC-003)

**Constraints**: `maxDuration = 60` cho route AI (cần Hung kiểm gói Vercel, R5); outline ≤ 15.000 ký tự;
prompt ≤ 2.000 ký tự (Validator); key AI và service role chỉ ở server

**Scale/Scope**: ≤ 20 dự án, ≤ 30 tài khoản; L1 thử trên Print Your Wear (6 danh mục)

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.* — **Kết quả: PASS (cả hai lần)**

| # | Nguyên tắc / ràng buộc | Plan tuân thủ thế nào |
| --- | --- | --- |
| I | MVP-first | Chỉ bảng cho L1 (không `feedback`, không Storage); không streaming; không E2E browser |
| II | Module hoá | `app/` → `lib/{api,ai,export,log}` → `lib/engine` (thuần); AI qua `AIProvider` |
| III | Tái sử dụng | shadcn/ui, Supabase Auth, Zod `toJSONSchema`; một nguồn schema |
| IV | Usability, responsive | Mặc định từ danh mục/Style Guide; trạng thái loading/empty/error; 360px (quickstart 4.8) |
| V | Test business logic | Unit engine + export + validator; integration RLS; provider giả |
| VI | Không breaking âm thầm | Hợp đồng ở `contracts/` + `data-model.md`, chỉ sửa trên `main` qua người điều phối |
| VII | Bám spec | Mỗi plan worktree liệt kê FR/SC phụ trách; chỗ mơ hồ → hỏi người điều phối |
| VIII | ADR | R1, R2, R5 cần ADR; W1 viết ADR-0001 (stack), ADR-0002 (Supabase/RLS); W3 viết ADR-0003 (gọi AI, chia lô) |
| IX | Không secret | `.gitignore` chặn `.env*`; `.env.example` chỉ tên biến; `key_env` chỉ lưu tên |
| X | Duyệt trước khi push/merge/deploy | Worktree chỉ commit trên nhánh của mình; merge vào `main` do người điều phối làm sau khi Hung duyệt; không chạy migration remote |
| A | AI viết mô tả, code giữ phần bắt buộc | `finishPrompt` (W2) là nơi duy nhất sinh placeholder/negative/dont/aspect/màu |
| B | RLS | Bảng RLS trong `data-model.md` §2; test RLS là cổng nghiệm thu W1 |
| C | Phiên bản, đóng băng | `context_snapshot` chụp ở lần plan đầu; version trong mọi preset |
| D | Schema + Validator | `contracts/ai.md` strict schema; `validatePack`/`validatePrompt` |
| E | Quan sát, chi phí | `lib/log.ts` (W5) ghi 4 action + tokens/cost; giới hạn tổng của provider |

Complexity Tracking: không có vi phạm.

## Project Structure

### Documentation (this feature)

```text
specs/001-prompt-pack-mvp/
├── spec.md               # Spec đã duyệt (có Clarifications)
├── plan.md               # File này — plan tổng
├── research.md           # Quyết định kỹ thuật R1–R13
├── data-model.md         # Bảng, RLS, trạng thái (owner W1)
├── quickstart.md         # Kịch bản kiểm end-to-end
├── contracts/
│   ├── engine.md         # lib/engine types + hàm (owner W2)
│   ├── ai.md             # AIProvider + schema đầu ra AI (owner W3, schema ở W2)
│   ├── api.md            # Route Handlers (owner W5)
│   ├── export.md         # JSON/CSV/MD (owner W6)
│   └── style-guide.md    # Frontmatter Style Guide (schema W2, seed W4)
└── worktrees/
    ├── W1-foundation/    # spec.md (phạm vi) + plan.md (thực thi) + tasks.md (sau)
    ├── W2-engine/
    ├── W3-ai/
    ├── W4-pyw-data/
    ├── W5-session-api/
    └── W6-creator-ui/
```

### Source Code (repository root)

```text
app/
├── (auth)/login/                         # W1
├── (creator)/                            # W6
│   ├── projects/                         #   C1 chọn dự án
│   └── projects/[pid]/sessions/[sid]/    #   C2–C6
├── api/sessions/…, api/prompts/[pid]/    # W5
└── layout.tsx, globals.css               # W1 (khung), W6 (giao diện)
components/
├── ui/                                   # W6 (shadcn)
└── creator/                              # W6
lib/
├── engine/                               # W2 (thuần)
├── ai/                                   # W3
├── export/                               # W6 (thuần)
├── api/                                  # W5 (guard, lỗi, đọc DB → snapshot source)
├── supabase/{server,client,admin}.ts     # W1
└── log.ts                                # W5
supabase/
├── config.toml, migrations/*.sql         # W1 (duy nhất được thêm migration ở L1)
└── seed.sql                              # W1 (purpose templates + dữ liệu test)
scripts/seed-pyw.ts, seed/print-your-wear/ # W4
tests/rls/                                # W1
docs/adr/                                 # W1, W3
```

**Structure Decision**: một project Next.js; logic thuần trong `lib/engine` và `lib/export` để test không cần
DB/AI; mỗi thư mục có đúng một worktree sở hữu.

## Bản đồ worktree

| WT | Nhánh | Sở hữu (chỉ worktree này được sửa) | FR / SC phụ trách | Phụ thuộc vào | Cổng xong |
| --- | --- | --- | --- | --- | --- |
| W1 Nền | `001-w1-foundation` | `package.json` (khởi tạo, kèm script giữ chỗ `seed:pyw`, `ai:smoke`), cấu hình gốc, `app/(auth)`, `app/layout.tsx` khung, `lib/supabase/*`, `supabase/**` (gồm hàm RPC và user test), `tests/rls/**`, `.env.example`, `docs/adr/0001-0002` | FR-001…009, FR-019 (seed), FR-036 (quyền log), US2, SC-001, SC-002 | — | **G0** skeleton; **G1** migrations + RLS test xanh |
| W2 Engine | `001-w2-engine` | `lib/engine/**` (gồm `__fixtures__/` dùng chung) | FR-012, 013, 015 (kiểm), 017, 018a, 021–026, 027 (b), 029, 030 (dựng message), SC-006, SC-007 | G0 | Unit test xanh, hợp đồng engine đúng |
| W3 AI | `001-w3-ai` | `lib/ai/**`, `lib/prompts/**`, `scripts/ai-smoke.ts`, `docs/adr/0003` | FR-015, 020 (gọi AI, chia lô), 030 (prompt hệ thống), 035 (usage), 038 | G0; schema của W2 (có thể dùng bản trong contract trước) | Test với provider giả + 1 lần gọi thật thủ công (Hung duyệt key) |
| W4 Dữ liệu PYW | `001-w4-pyw-data` | `scripts/seed-pyw.ts`, `seed/print-your-wear/**`, `docs/style-guides/**` (chỉ khung, nội dung do Hung) | Assumptions (dự án thử), FR-018a/b (nạp Style Guide), FR-019 (dữ liệu) | G1 (bảng), schema `StyleGuideParams` của W2 | `npm run seed:pyw` trên Supabase local đúng quickstart 3 |
| W5 Phiên/API | `001-w5-session-api` | `app/api/**`, `lib/api/**`, `lib/log.ts`, `tests/fixtures/api/**` (bàn giao sớm cho W6) | FR-009, 014, 014a, 016 (lưu), 020 (điều phối), 027a, 028, 033 (dữ liệu), 034–037, US5, SC-003, SC-009 | G1, W2, W3 merge | Route test với provider giả + kịch bản quickstart 4 (bước 1–7) |
| W6 Giao diện | `001-w6-creator-ui` | `app/(creator)/**`, `components/**`, `lib/export/**`, `lib/api-client/**` (gọi API + chế độ mock đọc `tests/fixtures/api`), `app/globals.css` | FR-010–012 (UI), 016–018 (UI), 027 (UI), 031–033, 039, 040, US1/US3/US4 (UI), SC-008, SC-010 | G0; dùng mock theo `contracts/api.md` cho đến khi W5 merge | Export unit test xanh + quickstart 4 với API thật |

### Thứ tự và song song

```text
G0: W1 skeleton ──► merge main
        ├──► W2 engine ────────────┐
        ├──► W3 AI ────────────────┤
        ├──► W6 UI (mock API) ─────┤
        └──► W1 tiếp: G1 DB+RLS ───┼──► W4 seed PYW
                                   └──► W5 phiên/API (sau W2, W3) ──► W6 nối API thật ──► nghiệm thu L1
```

Thứ tự merge vào `main`: W1(G0) → W2 (bàn giao sớm `types`/`schemas`/`__fixtures__`) → W5 (bàn giao sớm
`tests/fixtures/api`) → W3 → W1(G1) → W4 → W5 (route) → W6.

### Nhật ký điều chỉnh hợp đồng (2026-11-08, sau khi 6 plan worktree báo cáo)

Tham chiếu Style Set dùng `key`; `lib/prompts/` (W3) chứa văn bản + phiên bản prompt hệ thống, engine import
được; `AIError` mang tokens; `AI_FAKE=1`; timeout SDK 50 s; RPC thay slot/prompt; cột `activity_logs.ok`;
409 có `details`; mã `NO_PUBLISHED_BRANDKIT`; `GET session` trả `editorOptions` + `filesToPrepare`; Admin không
cần là thành viên để gọi AI; placeholder `DESIGN#n` đánh số theo pack, `DESIGN:<key>` cho design trong danh
sách; user test do W1 seed. Plan worktree nào lệch các điểm này thì **hợp đồng thắng**; worktree tự cập nhật
plan của mình khi bắt đầu. Mỗi merge cần Hung duyệt
(constitution X); người điều phối rebase các nhánh còn lại sau mỗi merge.

### Quy tắc phối hợp

- Dependency mới: W1 cài sẵn toàn bộ danh sách ở Technical Context của file này trong G0, cộng `tsx`
  (chạy script TS) và `vitest`. Worktree khác cần thêm
  dependency → báo người điều phối; người điều phối thêm trên `main` để tránh xung đột `package.json`.
- Hợp đồng (`contracts/`, `data-model.md`) chỉ sửa trên `main` bởi người điều phối.
- Mỗi worktree: chạy `/speckit-tasks` → `/speckit-implement` trong thư mục `worktrees/<Wn>/` của mình;
  commit trên nhánh của mình; **không push, không merge** (constitution X).
- Báo cáo xong: liệt kê FR đã làm, test đã chạy (kèm kết quả thật), điểm lệch hợp đồng nếu có.

## Complexity Tracking

Không có vi phạm constitution cần biện minh.
