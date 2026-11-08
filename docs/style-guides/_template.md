---
project: <project-slug>
category: <category-slug>
category_name: <Tên danh mục>
version: 1
status: draft            # draft | published | archived
updated: YYYY-MM-DD
owner: <người chịu trách nhiệm>

# Mục đích ảnh mặc định khi AI đề xuất chỗ ảnh (key trong bộ 9 mục đích)
default_purposes: []
# Quy tắc mật độ ảnh cho AI đề xuất chỗ ảnh
images:
  hero: true             # bài luôn có 1 ảnh hero
  per_h2: 1              # số ảnh gợi ý cho mỗi H2 (0 = chỉ khi cần)
  max_total: 12          # trần số ảnh mỗi bài
  pinterest_pin: true    # có thêm 1 pin dọc

# Thông số phong cách, AI bắt buộc áp dụng khi viết prompt (tiếng Anh)
style:
  medium: ""             # vd: editorial lifestyle photography
  lighting: ""           # vd: soft natural window light
  composition: ""        # vd: clean negative space on the left for text overlay
  mood: ""               # vd: warm, friendly, practical
  camera: ""             # vd: eye level, 35mm, shallow depth of field
  color_treatment: ""    # vd: true-to-life colors, gentle warmth
# Phần dưới đây code tự gắn vào cuối prompt và kiểm, không qua AI
negative: ""             # vd: no watermark, no extra text, no distorted hands
check_brand_hex: false   # true = bắt buộc prompt có màu HEX của brand (cần Brandkit có bảng màu)

# Bổ sung riêng cho danh mục; quy tắc của Brandkit vẫn luôn thắng
dont: []                 # vd: "no stereotypes", "no partisan symbols"
banned_terms: []         # từ/tên không được xuất hiện trong prompt
---

# <Tên danh mục>: Style Guide

## 1. Vai trò của danh mục

<!-- CẦN ĐIỀN: danh mục giúp người đọc làm gì; giai đoạn phễu; mức độ bán hàng; giọng điệu -->

## 2. Khi nào bài cần ảnh

<!-- CẦN ĐIỀN: heading nào luôn có ảnh, heading nào không cần; khi nào dùng gallery; ví dụ cấu trúc bài
điển hình và số ảnh mong đợi -->

## 3. Phong cách hình ảnh

<!-- CẦN ĐIỀN: mô tả bằng lời cảm giác chung của ảnh; bối cảnh hay dùng; điều làm ảnh "đúng chất" danh mục -->

### Ảnh tham chiếu

<!-- CẦN ĐIỀN: link hoặc tên asset của 3–5 ảnh đẹp đang có trên blog, mỗi ảnh một dòng ghi vì sao đẹp -->

## 4. Hướng dẫn theo mục đích ảnh

| Mục đích | Chủ thể thường dùng | Bối cảnh | Ghi chú bố cục |
| --- | --- | --- | --- |
| `hero` | | | |

## 5. Con người trong ảnh

<!-- CẦN ĐIỀN: ai xuất hiện (độ tuổi, nhóm, gia đình…), mức đa dạng, khi nào không có người -->

## 6. Sản phẩm trong ảnh

<!-- CẦN ĐIỀN: loại sản phẩm hay xuất hiện; cách thể hiện (on-model, flatlay, lifestyle); giữ nguyên artwork -->

## 7. Nên / Không nên riêng cho danh mục

- Nên:
- Không nên:

## 8. Chủ đề nhạy cảm

<!-- CẦN ĐIỀN: chủ đề cần cẩn trọng và cách thể hiện bằng hình ảnh -->

## 9. Alt text và tên file

- Alt text: mô tả nội dung ảnh bằng tiếng Anh, có keyword khi tự nhiên, không nhồi từ khoá.
- Tên file: slug tiếng Anh, chữ thường, nối bằng gạch ngang.

## 10. Lịch sử phiên bản

| Version | Ngày | Thay đổi | Người sửa |
| --- | --- | --- | --- |
| 1 | YYYY-MM-DD | Tạo mới | |
