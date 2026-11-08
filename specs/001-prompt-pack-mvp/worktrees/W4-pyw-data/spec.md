# Worktree Spec: W4 — Dữ liệu Print Your Wear

**Nhánh**: `001-w4-pyw-data` · **Spec cha**: [../../spec.md](../../spec.md) · **Plan tổng**: [../../plan.md](../../plan.md)
**Plan thực thi**: [plan.md](./plan.md)

## Phạm vi W4 phụ trách

Đưa dự án thử nghiệm **Print Your Wear** vào cơ sở dữ liệu local để luồng Creator L1 chạy được trên dữ liệu
thật (spec cha, mục Assumptions).

| Hạng mục | Yêu cầu cha | W4 làm gì |
| --- | --- | --- |
| Dự án thử | Assumptions | 1 dòng `projects` cho Print Your Wear |
| Brandkit | FR-021, FR-022, FR-026 | 1 Brandkit `published`: giọng brand, nên/không nên, từ cấm; **không có bảng màu, font, logo** |
| Danh mục | FR-010 | 6 danh mục (culture, how-to, lifestyle, occasions, fashion, gift-guides) với ghi chú use case và mục đích ảnh mặc định |
| Style Guide | FR-018a, FR-018b | Nạp `docs/style-guides/print-your-wear/*.md` vào `category_style_guides` có phiên bản |
| Style Set | FR-008, FR-017 | Ít nhất 1 Style Set mặc định `published` của dự án |
| Sản phẩm, persona, tham chiếu | FR-018, FR-023 | `product_types`, `personas` (người mẫu), `project_refs` (chỉ tên/mô tả, không file) |
| Thành viên test | FR-008 | Cấp quyền dự án cho tài khoản Creator test trên Supabase local |

## Ngoài phạm vi

- Migration, RLS, bảng mới (W1). Schema `StyleGuideParams` (W2). UI Admin (L2).
- Viết nội dung phong cách hình ảnh: do Hung soạn trong các file Style Guide; W4 chỉ nạp và kiểm.
- Chạy seed lên Supabase thật hoặc Vercel (constitution X).
- Lưu file ảnh (L1 không lưu file).

## Phụ thuộc

- **G1 của W1**: bảng và RLS theo [data-model.md](../../data-model.md) đã merge vào `main`.
- **W2**: Zod `StyleGuideParams` trong `lib/engine/schemas.ts` ([contracts/style-guide.md](../../contracts/style-guide.md)).
  Trước khi W2 merge, W4 dùng bản sao tạm theo contract và thay bằng import khi W2 merge.
- **Hung**: dữ liệu ở [plan.md § Dữ liệu chờ Hung](./plan.md#dữ-liệu-chờ-hung).

## Giả định

- Brandkit PYW không có bảng màu nên bộ kiểm không đòi màu HEX (`check_brand_hex: false`).
- `audience.personas` trong `brand.json` là **độc giả**, không nạp vào bảng `personas` (người mẫu).
- Chỉ phần quy tắc áp dụng được cho **hình ảnh** trong `brand.json` mới vào Brandkit; quy tắc viết bài bị bỏ.
- Seed chỉ chạy trên Supabase local; sau khi Hung duyệt mới chạy lên môi trường thật.

## Tiêu chí nghiệm thu (khớp [quickstart.md](../../quickstart.md) kịch bản 3)

1. `npm run seed:pyw` chạy trên Supabase local không lỗi; chạy lần 2 không tạo dòng trùng, không tăng phiên bản.
2. Có đúng 1 dự án, 6 danh mục, 1 Brandkit `published` không có bảng màu.
3. Mỗi file Style Guide `published` có đủ `style.*` thì thành 1 dòng `published`; file `published` mà `style` rỗng thì
   bị từ chối kèm thông báo tên file; file `draft` được nạp ở trạng thái `draft`.
4. Sửa nội dung một Style Guide rồi seed lại → phiên bản mới, bản `published` cũ chuyển `archived`.
5. Có ≥ 1 Style Set `published`, loại sản phẩm, persona và `project_refs` theo dữ liệu Hung duyệt.
6. Creator test đăng nhập thấy Print Your Wear và 6 danh mục; Creator khác không thấy (đối chiếu test RLS của W1).
7. Unit test parse + validate của script xanh.
