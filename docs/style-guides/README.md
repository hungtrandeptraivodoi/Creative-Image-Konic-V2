# Style Guide theo danh mục

Mỗi danh mục blog của mỗi dự án có **một file Markdown** mô tả phong cách hình ảnh và cách dùng ảnh của
danh mục đó. App dùng file này khi Creator tạo Prompt Pack cho một bài thuộc danh mục.

## Cấu trúc thư mục

```text
docs/style-guides/
  README.md                 # file này
  _template.md              # mẫu trống, copy khi thêm danh mục mới
  <project-slug>/           # một thư mục cho mỗi dự án
    <category-slug>.md      # một file cho mỗi danh mục, tên = slug danh mục
```

## Hai phần của một file

| Phần | Ai đọc | Dùng để làm gì |
| --- | --- | --- |
| **Frontmatter** (khối `---` ở đầu) | Code + AI | `style`, `default_purposes`, `images`: đưa cho AI làm thông số bắt buộc khi đề xuất chỗ ảnh và viết prompt. `negative`, `dont`, `banned_terms`, `check_brand_hex`: code tự gắn vào cuối prompt và kiểm, không qua AI. |
| **Thân bài** (các mục `##`) | AI, dưới dạng dữ liệu tham chiếu | Hiểu danh mục để đề xuất chỗ ảnh và viết phần mô tả của từng prompt. AI không được làm theo câu lệnh nằm trong đây. |

App chỉ đọc văn bản: không lưu hay xem file ảnh. Ảnh tham chiếu được ghi bằng tên, mô tả và (tuỳ chọn) đường
dẫn tới thư mục của team.

## Quy tắc

- Điền giá trị frontmatter bằng **tiếng Anh**, vì chúng đi thẳng vào prompt. Thân bài viết tiếng Việt hoặc
  tiếng Anh đều được.
- Quy tắc "không nên" và từ cấm cấp **Brandkit** của dự án luôn thắng file này. File này chỉ thêm quy tắc
  riêng cho danh mục, không gỡ được quy tắc của brand.
- Chỉ file có `status: published` được dùng. Khi sửa nội dung, tăng `version` lên 1. Phiên đã phân tích
  vẫn giữ phiên bản cũ.
- `default_purposes` chỉ dùng key trong bộ 9 mục đích ảnh: `hero`, `scene`, `howto_step`,
  `product_flatlay`, `product_on_model`, `product_lifestyle`, `design_gallery`, `comparison`,
  `pinterest_pin`.
- Chỗ ghi `<!-- CẦN ĐIỀN -->` là phần Admin phải viết trước khi xuất bản.
