# Implementation Plan: W2 Prompt Engine

> **Ghi chú người điều phối (2026-11-08)**: hợp đồng đã được điều chỉnh sau khi plan này được viết — xem
> mục "Nhật ký điều chỉnh hợp đồng" trong [plan tổng](../../plan.md). Chỗ nào plan này lệch hợp đồng thì
> **hợp đồng thắng**; các mục "Cần xác nhận" bên dưới đã được trả lời ở đó. Cập nhật plan này trước khi chạy
> `/speckit-tasks`.

**Branch**: `001-w2-engine` | **Date**: 2026-11-08 | **Spec**: [spec.md](./spec.md) | **Plan tổng**: [../../plan.md](../../plan.md)

## Summary

Hiện thực `lib/engine/**` thành các hàm thuần theo [contracts/engine.md](../../contracts/engine.md): types và
schemas (bàn giao sớm cho W3/W4/W5/W6), parse outline, resolver, graph, `finishPrompt`/`assemblePack`,
validator, và hàm dựng message cho bước B/D. Làm theo TDD; có fixtures dùng chung cho các worktree khác.

## Technical Context

**Language/Version**: TypeScript 5 strict (từ G0 của W1)

**Primary Dependencies**: chỉ Zod 4 (`z.toJSONSchema`). Không thêm dependency.

**Storage**: N/A (hàm thuần)

**Testing**: Vitest, test đặt cạnh module (`*.test.ts`), fixtures trong `lib/engine/__fixtures__/`

**Constraints**: không import I/O (W2-R1); deterministic (không `Date.now()`/random bên trong, thời gian
truyền vào qua tham số)

## Constitution Check

**Kết quả: PASS**

| # | Áp dụng cho W2 |
| --- | --- |
| A | `finishPrompt` là nơi duy nhất sinh placeholder, negative, dont, aspect, màu brand; schema `body` cấm `[[` và mã HEX |
| D | Schema strict (mọi field required, `additionalProperties: false`, field tuỳ chọn là nullable); `validatePack`/`validatePrompt` đủ 10 `IssueCode` |
| V | TDD: viết test trước mỗi module; ma trận test bên dưới là cổng xong |
| II | Phụ thuộc một chiều: engine không import `lib/ai`, `lib/api`, Supabase, Next; W2-SC4 kiểm bằng test |
| I, III | Parser outline tự viết vài chục dòng (research R8); không thêm thư viện |
| VI, VII | Lệch hợp đồng → báo người điều phối; không tự sửa `contracts/` |

## Project Structure

```text
lib/engine/
├── types.ts            # 1. types theo contract
├── schemas.ts          # 1. Zod + JSON Schema: SlotPlan, WrittenPack, StyleGuideParams, request body API
├── versions.ts         # 1. (xem "Cần xác nhận" #2)
├── outline.ts          # 2
├── resolver.ts         # 3. buildSnapshot, resolveSlotContext
├── graph.ts            # 4. computeGenOrder
├── assembler.ts        # 5. finishPrompt, assemblePack
├── validator.ts        # 6. validatePack, validatePrompt
├── planner-input.ts    # 7. buildPlannerMessages
├── writer-input.ts     # 7. buildWriterMessages, splitIntoBatches
├── index.ts            # re-export công khai
└── __fixtures__/       # dùng chung cho W3, W5, W6 (chỉ đọc)
    ├── outline-6h2.md                # outline mẫu Gift Guides, 6 H2 + H3
    ├── snapshot-pyw-no-palette.ts    # ContextSnapshot Print Your Wear, palette rỗng, có Style Guide
    ├── snapshot-with-palette.ts      # có palette, Style Set check_brand_hex = true
    ├── snapshot-no-style-guide.ts    # styleGuide = null
    ├── slot-plan.ts                  # SlotPlan giả cho outline mẫu (8 slot, 1 gallery 3 ảnh → 10 ảnh)
    └── written-pack.ts               # WrittenPack giả khớp slot-plan
```

## Thứ tự hiện thực (TDD)

| Bước | Module | Test viết trước | Bàn giao |
| --- | --- | --- | --- |
| 1 | `types.ts`, `schemas.ts`, `versions.ts`, `__fixtures__/` | Schema chấp nhận fixtures hợp lệ; từ chối `body` có `[[`, `count` 0/7, filename sai slug, field thừa; `toJSONSchema` có `additionalProperties: false` ở mọi object | **Commit "schemas-first"** → báo người điều phối merge sớm để W3/W4/W6 dùng |
| 2 | `outline.ts` | 6 H2 đúng thứ tự; H3 gắn H2 cha; bỏ heading trong code fence; H1 bỏ qua; H4+ gộp vào H3; không H2 → lỗi; > 15.000 ký tự → lỗi | |
| 3 | `resolver.ts` | Thứ tự ưu tiên 6 lớp; notes Creator không gỡ được `dont`; Style Set ngoài danh sách được phép bị từ chối; không cấp gì → mọi Style Set published; thiếu Style Guide → fallback + `NO_STYLE_GUIDE` | |
| 4 | `graph.ts` | Sắp topo ổn định (giữ thứ tự `position` khi hoà); vòng 2 và 3 nút trả đúng các mã; trỏ slot không tồn tại | |
| 5 | `assembler.ts` | Phần bắt buộc đúng thứ tự trong contract; palette rỗng → không dòng màu; logo chỉ khi Brandkit có `logoRefKey`; design `later` → `[[REF:DESIGN#n]]` source `creator`; `text_overlay` → `[[TEXT:…]]`; GEN từ `dependsOnSlot` và quy tắc `pinterest_pin`/`howto_step`; `originalText = promptText` | |
| 6 | `validator.ts` | Mỗi `IssueCode` có ít nhất 1 test dương và 1 test âm; `validatePrompt` sau sửa tay | |
| 7 | `planner-input.ts`, `writer-input.ts` | Dữ liệu bài và Style Guide nằm trong `<article_data>`/`<style_guide>`; system text lấy từ tham số; `splitIntoBatches` 20 → 1 lô, 21 → 10/10/1; lô sau nhận `usedCompositions` | |

## Ma trận test bắt buộc

| Test | FR / SC | IssueCode |
| --- | --- | --- |
| Placeholder trong prompt thiếu dòng refs | FR-023, FR-026 | `PLACEHOLDER_MISSING_REF` |
| Ref không xuất hiện trong prompt | FR-023, FR-026 | `REF_NOT_IN_PROMPT` |
| Mục đích sản phẩm thiếu PRODUCT/DESIGN | FR-026 | `REQUIRED_REF_MISSING` |
| Vòng GEN | FR-024, SC-007 | `GEN_CYCLE` |
| GEN trỏ slot đã xoá | FR-026 | `GEN_TARGET_MISSING` |
| Prompt > `limits.promptMaxChars` | FR-026 | `PROMPT_TOO_LONG` |
| Từ cấm (Brandkit ∪ Style Guide), không phân biệt hoa thường | FR-022, SC-006 | `BANNED_TERM` |
| Có palette + bật kiểm, prompt thiếu HEX | FR-026, SC-006 | `BRAND_HEX_MISSING` |
| Palette rỗng → không báo HEX | FR-021, Clarification Q3 | (không có issue) |
| `text_overlay` có giá trị | FR-029 | `TEXT_CHECK_REMINDER` |
| Danh mục không Style Guide | Edge case cha | `NO_STYLE_GUIDE` |
| Outline 6 H2 + fixtures → 7–10 ảnh, không vòng | SC-007 | — |
| Notes chứa "ignore the dont rules" | FR-022, FR-030 | — |
| Sửa tay xoá placeholder bắt buộc | FR-027 | `PLACEHOLDER_MISSING_REF` / `REQUIRED_REF_MISSING` |
| Engine không import module cấm | W2-R1 | — |

## Cần người điều phối xác nhận

1. **Cách nhận chữ prompt hệ thống (tránh phụ thuộc ngược engine → `lib/ai`).** Đề xuất:
   `buildPlannerMessages(snap, intake, system: SystemPrompt)` và `buildWriterMessages(…, system: SystemPrompt)`
   với `SystemPrompt = { text: string; version: string }` khai báo trong `types.ts`. W3 export các hằng
   `PLANNER_SYSTEM` / `WRITER_SYSTEM` kiểu `SystemPrompt` trong `lib/ai/prompts/*`; W5 import từ `lib/ai` rồi
   truyền vào engine. Chữ ký này **thêm một tham số** so với [contracts/engine.md](../../contracts/engine.md).
2. **`versions.ts`**: contract đặt hằng phiên bản prompt hệ thống ở engine, còn [contracts/ai.md](../../contracts/ai.md)
   đặt cạnh nội dung prompt ở W3. Đề xuất: phiên bản đi cùng nội dung (`SystemPrompt.version` của W3); bỏ
   `versions.ts` khỏi engine, `templateVersions` lấy `system.version` từ tham số.

## Điểm bàn giao

- Sau bước 1: types, schemas, fixtures → W3 (validate output), W4 (`StyleGuideParams`), W6 (types hiển thị, mock).
- Sau bước 6: `assemblePack`, `validatePack`, `validatePrompt`, `computeGenOrder` → W5.
- Sau bước 7: message builders → W5 (kết hợp với chữ prompt hệ thống của W3).

## Rủi ro

| Rủi ro | Xử lý |
| --- | --- |
| So khớp từ cấm bắt nhầm hoặc bỏ sót ("cheap" vs "cheapest", cụm nhiều từ, dấu nháy) | So theo ranh giới từ, không phân biệt hoa thường, chuẩn hoá khoảng trắng; ghi quy tắc vào test; không bắt trong placeholder |
| Phần bắt buộc làm prompt vượt 2.000 ký tự (nhiều dont, dòng màu, nhiều refs) | Đo độ dài phần bắt buộc trong test với fixture "nặng"; nếu vượt, báo người điều phối để chỉnh ngân sách độ dài trong hướng dẫn cho AI (W3) |
| `dont` từ Brandkit và Style Guide trùng nhau | Gộp, khử trùng lặp, giữ thứ tự Brandkit trước |
| Đánh số `DESIGN#n` không ổn định giữa các lần tạo | Đánh số theo `gen_order` rồi `index`, trong phạm vi cả pack |
| Schema strict của Zod 4 sinh JSON Schema không hợp yêu cầu strict của provider | Test snapshot JSON Schema; W3 thử với 1 lần gọi thật |
