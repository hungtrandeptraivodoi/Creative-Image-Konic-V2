# Worktree Spec: W2 Prompt Engine

**Nhánh**: `001-w2-engine` · **Spec cha**: [../../spec.md](../../spec.md) · **Plan tổng**: [../../plan.md](../../plan.md)
· **Plan thực thi**: [plan.md](./plan.md) · **Created**: 2026-11-08 · **Status**: Draft

W2 xây phần lõi tất định của Prompt Engine: đọc outline, dựng bản chụp context, dựng message cho AI, hoàn
thiện prompt từ phần mô tả AI viết, tính thứ tự tạo ảnh và kiểm lỗi. Mọi thứ là hàm thuần, chạy được không
cần DB, AI hay giao diện. Hợp đồng: [contracts/engine.md](../../contracts/engine.md),
[contracts/ai.md](../../contracts/ai.md) (phần schema), [contracts/style-guide.md](../../contracts/style-guide.md).

## User Scenarios & Testing

### Story 1 — Hoàn thiện prompt đúng phần bắt buộc (P1)

Khi nhận phần mô tả AI viết cho một ảnh, engine trả về prompt hoàn chỉnh có hướng dẫn tham chiếu kèm
placeholder, quy tắc "không nên", tỉ lệ khung, negative, và dòng màu brand chỉ khi Brandkit có bảng màu.

**Independent Test**: với snapshot mẫu và WrittenPack giả, `assemblePack` cho đủ prompt; kiểm bằng unit test.

1. **Given** snapshot Print Your Wear không có bảng màu, **When** hoàn thiện prompt, **Then** không có dòng
   màu brand và không có cảnh báo `BRAND_HEX_MISSING`.
2. **Given** snapshot có bảng màu và Style Set bật kiểm HEX, **When** hoàn thiện prompt, **Then** prompt có
   ít nhất một mã HEX của brand.
3. **Given** ghi chú của Creator yêu cầu bỏ quy tắc "không nên", **When** hoàn thiện, **Then** quy tắc
   "không nên" của Brandkit vẫn có mặt.
4. **Given** phần mô tả của AI có chứa `[[`, **When** kiểm schema, **Then** bị từ chối.

### Story 2 — Phát hiện lỗi của Pack và của prompt sửa tay (P1)

Engine trả danh sách lỗi/cảnh báo theo từng thẻ, và kiểm lại một prompt sau khi Creator sửa tay.

**Independent Test**: bộ test validator với các ca lỗi dựng sẵn.

1. **Given** hai ảnh phụ thuộc vòng, **Then** có lỗi `GEN_CYCLE` chỉ ra các mã ảnh trong vòng.
2. **Given** prompt sửa tay xoá mất placeholder bắt buộc, **When** `validatePrompt`, **Then** có
   `PLACEHOLDER_MISSING_REF` hoặc `REQUIRED_REF_MISSING`.
3. **Given** prompt dài hơn 2.000 ký tự, **Then** có `PROMPT_TOO_LONG`.

### Story 3 — Đọc outline và dựng message cho AI (P2)

**Independent Test**: unit test với outline mẫu 6 H2.

1. **Given** outline 6 H2 kèm H3, **Then** `parseOutline` trả đúng 6 H2 theo thứ tự, bỏ heading trong code fence.
2. **Given** outline không có H2 hoặc dài hơn 15.000 ký tự, **Then** engine báo lỗi trước khi dựng message.
3. **Given** outline chứa câu lệnh nhắm vào AI, **Then** nội dung nằm trong thẻ dữ liệu, không nằm trong phần hệ thống.

### Edge Cases

- Danh mục không có Style Guide → snapshot `styleGuide = null`, cảnh báo `NO_STYLE_GUIDE`, dùng Style Set mặc định.
- Creator không được cấp Style Set nào trong dự án → được dùng mọi Style Set đã xuất bản (FR-008).
- `dependsOnSlot` trỏ tới chỗ ảnh đã xoá → `GEN_TARGET_MISSING`.
- Từ cấm viết hoa/thường khác nhau, hoặc nằm trong từ dài hơn (vd "cheap" trong "cheapest").
- Prompt có `text_overlay` → có `[[TEXT:…]]` và cảnh báo `TEXT_CHECK_REMINDER`.
- Bài > 20 ảnh → chia lô 10.

## Requirements

FR W2 phụ trách (theo bản đồ worktree, nội dung đầy đủ ở spec cha):

| FR cha | Phần W2 làm |
| --- | --- |
| FR-012, FR-013 | `parseOutline`: heading H2/H3, đếm H2, đếm ký tự; báo lỗi không H2 / quá dài |
| FR-015 | Kiểm SlotPlan sau Zod: mã liên tiếp, `purpose_key` hợp lệ, `count` 1–6, trần `max_total` |
| FR-017 | `resolveSlotContext`: thứ tự ưu tiên context; Style Set chỉ trong danh sách được phép |
| FR-018a | `StyleGuideParams` schema; style + hướng dẫn vào message AI; negative/dont/banned do code gắn |
| FR-020 (một phần) | `buildWriterMessages`, `splitIntoBatches` (≤ 20 → 1 lô; > 20 → lô 10, truyền `composition_tag` đã dùng) |
| FR-021, FR-022 | `finishPrompt`: phần bắt buộc; "không nên" và từ cấm của Brandkit luôn thắng |
| FR-023 | Placeholder `[[LOẠI:KHOÁ]]` chỉ sinh trong `finishPrompt`; `refs` một-một |
| FR-024 | `computeGenOrder`: sắp topo hoặc trả vòng |
| FR-025 | Schema alt text (≤ 125 ký tự) và tên file (slug, `.webp`) |
| FR-026 | `validatePack`: đủ 10 `IssueCode` |
| FR-027 (b) | `validatePrompt` cho prompt sửa tay |
| FR-029 | Cảnh báo `TEXT_CHECK_REMINDER` |
| FR-030 | Dựng message: dữ liệu bài và Style Guide trong thẻ phân cách |

Yêu cầu riêng của worktree:

- **W2-R1**: `lib/engine/**` không import Supabase, `openai`, `next/*`, `lib/ai/**` hay bất kỳ module I/O nào.
- **W2-R2**: Chữ ký hàm và types khớp [contracts/engine.md](../../contracts/engine.md); lệch thì báo người điều phối, không tự đổi.
- **W2-R3**: Có bộ fixtures dùng chung cho W3, W5, W6 (xem plan).

## Success Criteria

- **W2-SC1**: 100% test trong ma trận test ở [plan.md](./plan.md) chạy xanh bằng `npm test -- lib/engine`.
- **W2-SC2** (SC-006 cha): với fixtures, 0 prompt chứa từ cấm; với snapshot có bảng màu, 100% prompt có HEX brand.
- **W2-SC3** (SC-007 cha): outline 6 H2 mẫu kèm SlotPlan giả cho 7–10 ảnh, không vòng phụ thuộc.
- **W2-SC4**: kiểm tra phụ thuộc (lint rule hoặc test) xác nhận W2-R1.
- **W2-SC5**: bộ test của engine chạy dưới 5 giây, không cần mạng.

## Ngoài phạm vi

Đọc/ghi DB, gọi AI thật, nội dung chữ của prompt hệ thống (W3), route API (W5), giao diện và export (W6),
seed dữ liệu (W4).

## Phụ thuộc và giả định

- Cần G0 của W1 (skeleton có TypeScript, Vitest, Zod 4) trước khi chạy test.
- Chữ của prompt hệ thống do W3 cung cấp; W2 nhận qua tham số (xem plan, phần cần xác nhận).
- Giới hạn mặc định: outline 15.000 ký tự, prompt 2.000 ký tự (lấy từ `snapshot.limits`).
