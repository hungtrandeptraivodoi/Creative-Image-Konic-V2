# Spec phạm vi: W1 Nền (skeleton, dữ liệu, phân quyền, đăng nhập)

**Feature Branch**: `001-w1-foundation` · **Created**: 2026-11-08 · **Status**: Draft

**Spec cha**: [../../spec.md](../../spec.md) · **Plan tổng**: [../../plan.md](../../plan.md)

File này chỉ khoanh phạm vi W1 trong spec cha. Hành vi chi tiết, edge case và định nghĩa nằm ở spec cha;
nếu hai nơi lệch nhau, spec cha thắng.

## User Scenarios & Testing

### User Story 1 - Nền dự án chạy được cho mọi worktree (Priority: P1) — cổng G0

Đội phát triển có một bộ khung ứng dụng thống nhất, có đủ thư viện đã chốt, cấu trúc thư mục theo plan tổng
và các lệnh chuẩn để chạy, kiểm kiểu, lint và test, để W2–W6 bắt đầu trên cùng một nền mà không xung đột.

**Independent Test**: clone nhánh, cài dependency, chạy đủ các lệnh chuẩn đều thành công; trang chủ mở được.

**Acceptance Scenarios**:

1. **Given** máy có Node 20, **When** cài dependency và chạy lệnh build, typecheck, lint, test, **Then**
   tất cả thành công (test mẫu xanh).
2. **Given** repo, **When** tìm file chứa giá trị secret, **Then** không có; chỉ có `.env.example` liệt kê tên biến.

### User Story 2 - Creator chỉ thấy và dùng dự án được cấp (Priority: P1) — cổng G1

Tương ứng User Story 2 của spec cha (FR-001…FR-009). Kiểm bằng truy vấn trực tiếp dữ liệu, không qua giao diện.

**Independent Test**: tạo 2 Creator cấp 2 dự án khác nhau và 1 Admin trên môi trường local; bộ test quyền
chạy tự động và xanh.

**Acceptance Scenarios**: như spec cha US2 (1–5), cộng thêm:

6. **Given** Admin không là thành viên dự án X, **When** Admin tạo phiên của chính mình ở X, **Then** thành
   công; **When** Admin sửa hoặc xoá phiên của Creator A, **Then** bị từ chối (FR-007).
7. **Given** Creator, **When** đọc preset (Brandkit, Style Set, Style Guide, mục đích ảnh), **Then** chỉ
   thấy bản đã xuất bản của dự án được cấp.
8. **Given** Creator, **When** đọc hoặc ghi nhật ký hoạt động, **Then** bị từ chối (FR-036).

### User Story 3 - Đăng nhập bằng lời mời (Priority: P2) — cổng G1

Người được Admin mời đăng nhập bằng email + mật khẩu hoặc liên kết qua email; không có đường tự đăng ký;
tài khoản bị khoá bị chặn (FR-001…FR-003).

**Independent Test**: mời 1 email trên local, đăng nhập bằng cả hai cách; thử truy cập trang trong khi chưa
đăng nhập → bị chuyển về trang đăng nhập.

### User Story 4 - Bộ 9 mục đích ảnh có sẵn (Priority: P2) — cổng G1

Sau khi dựng dữ liệu, hệ thống có đúng 9 mục đích ảnh đã xuất bản với tỉ lệ mặc định, nhóm, tham chiếu bắt
buộc/tuỳ chọn như spec cha FR-019, kèm hướng dẫn viết cho AI và mẫu câu tham chiếu.

**Independent Test**: dựng lại dữ liệu local, đếm và đối chiếu 9 mục với bảng FR-019.

## Requirements

### Phụ trách (từ spec cha)

- FR-001, FR-002, FR-003 (tài khoản, đăng nhập, khoá).
- FR-004, FR-005, FR-006, FR-007, FR-008 (quyền ở tầng dữ liệu).
- FR-009: W1 cung cấp hàm kiểm quyền thành viên ở DB; W5 gọi trước mỗi lượt AI.
- FR-019: dữ liệu 9 mục đích ảnh.
- FR-036: chỉ hệ thống ghi nhật ký; Creator không đọc/sửa.
- SC-001, SC-002.

### Thêm cho W1

- **W1-R1**: Khung ứng dụng và lệnh chuẩn (dev, build, lint, typecheck, test, test:rls, db:reset) MUST có
  sẵn và được ghi vào `CLAUDE.md`.
- **W1-R2**: Toàn bộ dependency trong Technical Context của plan tổng MUST được cài ở G0.
- **W1-R3**: Cấu trúc dữ liệu MUST đúng [data-model.md](../../data-model.md); mọi bảng dự án có quy tắc quyền
  trong cùng migration.
- **W1-R4**: Dữ liệu test local gồm 1 Admin, 2 Creator, 2 dự án (mỗi Creator được cấp 1 dự án), đủ để chạy
  bộ test quyền; không dùng dữ liệu thật.

### Ngoài phạm vi W1

Logic Prompt Engine (W2), gọi AI (W3), dữ liệu Print Your Wear (W4), API phiên và ghi log (W5), giao diện
Creator và export (W6), màn hình Admin (L2), bảng `feedback` (L3), Storage file (L2+). Không tạo project
Supabase/Vercel thật.

## Phụ thuộc và giả định

- Không phụ thuộc worktree khác. W2–W6 phụ thuộc G0; W4, W5 phụ thuộc G1.
- Máy dev có Docker Desktop chạy được Supabase local trên Windows.
- Gửi email lời mời/magic link trên local dùng hộp thư giả của Supabase (Inbucket/Mailpit).

## Success Criteria (cổng)

- **G0**: build, typecheck, lint, test đều thành công trên máy sạch; không có secret trong repo; cấu trúc thư
  mục khớp plan tổng; nhánh merge vào `main` sau khi Hung duyệt.
- **G1**: dựng lại DB local thành công; bộ test quyền xanh 100% (SC-001, SC-002, FR-007, FR-036); 9 mục đích
  ảnh đúng FR-019; đăng nhập 2 cách hoạt động, không tự đăng ký được; ADR-0001 và ADR-0002 có mặt.
