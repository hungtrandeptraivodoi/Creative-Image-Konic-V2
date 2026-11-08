# Spec phạm vi: W5 — Phiên làm việc & API

**Worktree**: `001-w5-session-api` · **Spec cha**: [../../spec.md](../../spec.md) · **Plan tổng**:
[../../plan.md](../../plan.md) · **Date**: 2026-11-08

Spec này chỉ cắt phạm vi W5 từ spec cha; không thêm yêu cầu mới. Chỗ nào khác spec cha thì spec cha thắng.

## Phạm vi W5 phụ trách

**User stories** (phần phía server): US1 (luồng Phân tích → Tạo prompt), US3 (lưu chỉnh sửa chỗ ảnh),
US4 (route export), US5 (nhật ký và chi phí).

| FR | Nội dung W5 đảm nhận |
| --- | --- |
| FR-009 | Kiểm quyền thành viên (hoặc admin) ngay trước mỗi lượt gọi AI |
| FR-014, FR-014a | Chụp snapshot ở lần phân tích đầu; phân tích lại dùng lại snapshot, cần xác nhận khi đã có slot/prompt |
| FR-016 | Lưu thêm/xoá/đổi thứ tự/đổi số ảnh của chỗ ảnh |
| FR-020 | Điều phối bước D: chia lô tuần tự, truyền `composition_tag` đã dùng |
| FR-027 (a), FR-027a | Tạo lại cả pack (409 khi có prompt sửa tay); lưu prompt sửa tay, khôi phục bản gốc |
| FR-028 | Ghi `model` và `template_versions` cho từng prompt |
| FR-033 | Trả dữ liệu "File cần chuẩn bị" (refs) trong response |
| FR-034, FR-035, FR-036 | Ghi `activity_logs` (4 action + `slots.edit`) qua service role, kèm tokens/cost/duration |
| FR-037 | Kiểm giới hạn tổng/ngày của provider trước khi gọi AI; không giới hạn theo Creator |

**Success criteria**: SC-003 (≤ 60 giây/bài 7–10 ảnh — đo thủ công với provider thật), SC-009 (đủ 4 loại
dòng log mỗi phiên hoàn chỉnh). Góp phần cho SC-004 (Pack không lỗi khi lưu và trả về đúng `issues`).

## Ngoài phạm vi W5

- Logic thuần (parse outline, snapshot, ghép prompt, validator, gen order) — W2.
- Gọi AI, prompt hệ thống, provider giả — W3.
- Migration, RLS, Supabase client — W1. Formatter export — W6. Giao diện — W6. Seed — W4.
- Đọc danh sách dự án/danh mục/phiên (W6 đọc thẳng bằng Supabase có RLS).

## Phụ thuộc

- **G1** (W1): bảng + RLS theo [data-model.md](../../data-model.md), `lib/supabase/{server,admin}.ts`.
- **W2 merge**: `lib/engine/*` theo [contracts/engine.md](../../contracts/engine.md).
- **W3 merge**: `lib/ai/factory.ts`, `fake.ts` theo [contracts/ai.md](../../contracts/ai.md).
- **W6**: `lib/export/*` theo [contracts/export.md](../../contracts/export.md) — route export chỉ gọi hàm.

## Giả định

- Một provider `is_active` (R6); ngày tính theo UTC khi đếm giới hạn.
- `maxDuration = 60` được gói Vercel cho phép (R5, Hung cần kiểm).
- Ghi nhiều bảng một lần cần cơ chế nguyên tử (xem plan, mục Rủi ro R-W5-1).

## Tiêu chí nghiệm thu W5

1. Test route (Vitest + Supabase local + provider giả) xanh cho mọi route trong
   [contracts/api.md](../../contracts/api.md), gồm 401/403/404/409/429/502/503.
2. Phân tích lại giữ nguyên `context_snapshot`; 409 khi chưa xác nhận; prompt sửa tay không bị ghi đè khi
   chưa xác nhận.
3. Mỗi phiên chạy hết luồng có đủ `session.create`, `slots.plan`, `prompts.generate`, `pack.export` với
   tokens/cost (SC-009); Creator không đọc được log.
4. Quickstart kịch bản 4 bước 1–7 chạy được với W6 (sau khi merge).
5. Đo thủ công SC-003 với provider thật (key do Hung duyệt), ghi kết quả thật vào báo cáo.
