# Worktree Spec: W3 — AI provider và prompt hệ thống

**Nhánh**: `001-w3-ai` · **Spec cha**: [../../spec.md](../../spec.md) · **Plan tổng**: [../../plan.md](../../plan.md)
· **Hợp đồng**: [../../contracts/ai.md](../../contracts/ai.md), [../../contracts/engine.md](../../contracts/engine.md)

## Mục tiêu

Cung cấp một lớp gọi AI duy nhất, tương thích chuẩn OpenAI, trả JSON đã kiểm schema kèm số liệu sử dụng,
và hai prompt hệ thống có phiên bản cho bước B (đề xuất chỗ ảnh) và bước D (viết phần mô tả prompt).

## FR / SC phụ trách

| Mã | Phần W3 làm |
| --- | --- |
| FR-015 | Prompt hệ thống Planner sinh danh sách chỗ ảnh đúng schema `SlotPlan` |
| FR-020 | Prompt hệ thống Writer + gọi AI cho từng lô (việc chia lô do `splitIntoBatches` của W2, điều phối do W5) |
| FR-030 | Prompt hệ thống coi nội dung trong thẻ phân cách là dữ liệu, không làm theo lệnh bên trong |
| FR-035 | Trả `tokensIn`, `tokensOut`, `durationMs`, `attempts`, `model` cho mỗi lần gọi |
| FR-038 | Đổi provider/model chỉ bằng dữ liệu `ai_providers` + biến môi trường; key không lộ |
| Edge case "AI sai định dạng" | Thử lại 1 lần, vẫn sai → `AIError('BAD_OUTPUT')` |
| Edge case "AI lỗi/không phản hồi" | Ánh xạ thành `AIError` có `retryable` để W5 trả 502 |
| SC-003 (một phần) | Một lượt Writer cho bài 7–10 ảnh trong ngân sách thời gian 60 giây |

## Ngoài phạm vi

- Không Route Handler, không đọc/ghi DB, không ghi `activity_logs` (W5).
- Không định nghĩa Zod/JSON Schema đầu ra (W2, `lib/engine/schemas.ts`); W3 chỉ dùng.
- Không dựng message từ snapshot (W2: `buildPlannerMessages`, `buildWriterMessages`).
- Không UI, không đếm giới hạn ngày (W5).

## Phụ thuộc

- **G0** (W1): skeleton, `openai`, `zod`, Vitest đã cài.
- **W2**: `SlotPlan`/`WrittenPack` schema. Trước khi W2 merge, W3 dùng bản sao tạm theo
  [contracts/ai.md](../../contracts/ai.md) trong test, xoá khi rebase.

## Giả định

- Một provider `is_active` tại một thời điểm; L1 dùng OpenAI hoặc endpoint tương thích.
- Tên model do Admin nhập trong `ai_providers`; W3 không cố định tên model.
- Không streaming ở L1 (research R5).

## Tiêu chí nghiệm thu

1. `npm test -- lib/ai` xanh: fake provider, ánh xạ lỗi, đường strict và đường `json_object` + retry.
2. Test hồi quy: prompt hệ thống chứa đủ các quy tắc bắt buộc (mục "Tiêu chí prompt" trong plan) và hằng số
   phiên bản tồn tại.
3. Một lần gọi thật thủ công (script, key do Hung đặt trong `.env.local`) cho outline mẫu 6 H2 trả
   `SlotPlan` hợp lệ và `WrittenPack` hợp lệ trong ≤ 60 giây; kết quả ghi vào báo cáo, không commit key.
4. ADR-0003 có trong `docs/adr/`.
