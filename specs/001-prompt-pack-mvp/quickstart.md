# Quickstart: kiểm chứng Prompt Pack MVP (L0 + L1)

Hướng dẫn chạy và kiểm end-to-end. Lệnh cụ thể (script npm) do W1 chốt trong `package.json` và cập nhật
vào `CLAUDE.md`; các tên dưới đây là tên dự kiến.

## Điều kiện

- Node.js 20 LTS, npm; Docker Desktop (cho Supabase local); Supabase CLI.
- `.env.local` tạo từ `.env.example` (chỉ tên biến, không commit giá trị).
- Không chạy migration hay deploy lên Supabase/Vercel thật khi chưa được Hung duyệt (constitution X).

## Kịch bản 1 — Nền và phân quyền (US2, SC-001, SC-002)

1. `npx supabase start` → `npm run db:reset` (áp migration + seed 9 mục đích ảnh + dữ liệu test).
2. `npm run test:rls` — tạo 2 Creator (A cấp dự án X, B cấp dự án Y) và 1 Admin, rồi kiểm:
   - A đọc `categories` của Y → rỗng; A ghi `brandkits` → bị từ chối.
   - A chỉ thấy phiên của mình; Admin thấy mọi phiên nhưng không sửa/xoá phiên của A.
   - Tài khoản `locked` không đọc được gì.
3. **Kỳ vọng**: mọi test xanh.

## Kịch bản 2 — Engine (SC-006, SC-007)

1. `npm test -- lib/engine` → outline mẫu 6 H2 cho 7–10 ảnh với provider giả; không vòng; palette rỗng
   không có dòng màu; từ cấm bị bắt; prompt sửa tay được kiểm lại.
2. **Kỳ vọng**: mọi test xanh.

## Kịch bản 3 — Dữ liệu Print Your Wear

1. Hung điền phần `style` và chuyển `status: published` cho ít nhất 3 file trong
   `docs/style-guides/print-your-wear/`.
2. `npm run seed:pyw` → kiểm: 1 dự án, 6 danh mục, Brandkit published (không bảng màu), Style Guide
   published cho các file đã điền, ≥ 1 Style Set, loại sản phẩm, persona, `project_refs`.

## Kịch bản 4 — Luồng Creator end-to-end (US1, US3, US4, US5, SC-003…SC-010)

1. `npm run dev`, đăng nhập Creator được cấp Print Your Wear.
2. Chọn danh mục Gift Guides → thấy ghi chú use case. Dán outline 6 H2 → thấy preview heading.
3. Bấm **Phân tích** → bảng chỗ ảnh có lý do. Đổi 1 chỗ thành gallery 3 ảnh, xoá 1 chỗ.
4. Bấm **Tạo prompt** → đo thời gian ≤ 60 giây; thẻ xếp theo thứ tự tạo; placeholder tô màu; "File cần
   chuẩn bị" đúng; không có dòng màu brand (PYW chưa có bảng màu).
5. Sửa tay 1 thẻ cho dài quá 2.000 ký tự → lỗi hiện ngay; khôi phục bản gốc.
6. Export CSV, JSON, Markdown → mở CSV bằng Excel/Google Sheets, đủ cột, tiếng Việt không lỗi.
7. Admin xem `activity_logs` (Dashboard) → có `session.create`, `slots.plan`, `prompts.generate`,
   `pack.export` kèm tokens và chi phí.
8. Thu hẹp cửa sổ còn 360px → không cuộn ngang, thao tác chính vẫn dùng được.

## Kịch bản 5 — Nghiệm thu L1 (SC-004, SC-005)

3 bài thật ở 3 danh mục khác nhau → Pack không lỗi Validator → dán prompt vào công cụ tạo ảnh → Hung chấm
≥ 70% ảnh dùng được lần đầu. Chỉ báo "đạt" khi có kết quả chấm thật.
