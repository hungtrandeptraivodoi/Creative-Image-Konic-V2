# Spec phạm vi: W6 — Giao diện Creator

**Worktree / nhánh**: `001-w6-creator-ui` · **Spec cha**: [../../spec.md](../../spec.md) ·
**Plan tổng**: [../../plan.md](../../plan.md) · **Date**: 2026-11-08

Tài liệu này chỉ khoanh phạm vi W6 trong spec cha; mọi yêu cầu, định nghĩa và Clarifications lấy nguyên từ
spec cha. Chỗ nào khác nhau thì spec cha thắng.

## Phạm vi

W6 làm toàn bộ giao diện của Creator cho luồng C1–C6 và bộ formatter export thuần.

### User Story phụ trách (phần giao diện)

| US | Nội dung | Phần W6 |
| --- | --- | --- |
| US1 (P1) | Sinh Prompt Pack từ keyword + outline | Chọn dự án/danh mục, nhập bài, preview heading, nút "Phân tích"/"Tạo prompt", thẻ prompt, "File cần chuẩn bị", nhãn phụ thuộc, sửa tay |
| US3 (P2) | Rà và cấu hình chỗ ảnh | Bảng/thẻ chỗ ảnh: thêm, xoá, đổi thứ tự, đổi số ảnh, trường phụ theo mục đích |
| US4 (P2) | Copy và export | Copy 1 prompt, copy cả pack, export JSON/CSV/Markdown; formatter `lib/export/` |

### Yêu cầu chức năng phụ trách

FR-010, FR-011, FR-012 (hiển thị), FR-016, FR-017, FR-018 (giao diện), FR-027 (giao diện sửa tay +
tạo lại), FR-027a (nhãn "đã sửa tay", khôi phục, cảnh báo ghi đè), FR-031, FR-032 (formatter), FR-033
(hiển thị), FR-039, FR-040. Hiển thị cảnh báo FR-029 ("kiểm chữ trước khi dùng") và lỗi của FR-026.

### Tiêu chí thành công phụ trách

- **SC-008**: Creator mới hoàn thành Pack đầu tiên không cần hỏi Admin (giao diện tự giải thích được).
- **SC-010**: CSV mở đúng trong Excel/Google Sheets, một dòng mỗi ảnh, không vỡ cột, tiếng Việt không lỗi.
- Góp phần vào SC-003 (hiển thị tiến trình rõ trong ≤ 60 giây chờ).

## Ngoài phạm vi

- Mọi màn hình Admin (A1–A9), màn hình Log, trang Chất lượng.
- Lịch sử phiên, nhân bản phiên (C8), sinh lại một phần (C7), chấm prompt (C9).
- Đăng nhập (W1 làm `app/(auth)/login`), Route Handlers và ghi log (W5), engine/Validator (W2), gọi AI (W3).
- Lưu, upload, xem trước file ảnh (L1 không lưu file).

## Phụ thuộc

| Phụ thuộc | Từ | Trước khi có thì |
| --- | --- | --- |
| G0 skeleton (Next.js, Tailwind, shadcn base, dependency) | W1 | Không bắt đầu code |
| Types `lib/engine/types.ts` | W2 | Dùng bản sao types từ [contracts/engine.md](../../contracts/engine.md) trong mock, đổi sang import thật khi W2 merge |
| Route Handlers theo [contracts/api.md](../../contracts/api.md) | W5 | Dùng mock API (xem plan) |
| Đăng nhập + Supabase client (`lib/supabase/*`) | W1 | Trang đọc dự án/danh mục chạy trên mock |

## Giả định

- Desktop là bối cảnh chính; màn hình 360px vẫn làm được mọi thao tác chính.
- Chữ giao diện tiếng Việt; prompt, alt text, tên file hiển thị nguyên văn tiếng Anh.
- Copy 1 prompt làm ở client, không gọi API, không ghi log (contracts/api.md).
- Một phiên chỉ có một Creator thao tác; không xử lý sửa đồng thời nhiều tab.

## Tiêu chí nghiệm thu W6

1. Unit test `lib/export/` xanh: đủ 12 trường theo [contracts/export.md](../../contracts/export.md), thứ tự
   theo `genOrder`, CSV escape đúng dấu phẩy/ngoặc kép/xuống dòng, có BOM UTF-8 và CRLF.
2. Với mock API: đi hết quickstart kịch bản 4 bước 2–6 không lỗi console.
3. Với API thật của W5: đi hết quickstart kịch bản 4 bước 1–6 và 8.
4. Ở chiều rộng 360px: không cuộn ngang trang trên cả 4 màn hình; thao tác chính làm được.
5. Toàn bộ luồng làm được chỉ bằng bàn phím (Tab/Shift+Tab/Enter/Space/Esc và phím đổi thứ tự).
6. Type-check và lint sạch.
