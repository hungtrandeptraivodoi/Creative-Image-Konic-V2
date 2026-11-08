---
project: print-your-wear
category: lifestyle
category_name: Lifestyle
version: 1
status: draft            # draft | published | archived
updated: 2026-11-08
owner: Hung

# Mục đích ảnh mặc định khi AI đề xuất chỗ ảnh (key trong bộ 9 mục đích)
default_purposes: [hero, scene, product_lifestyle, pinterest_pin]   # đề xuất ban đầu, Admin duyệt
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

# Lifestyle: Style Guide

## 1. Vai trò của danh mục

Lấy từ `docs/brand.json`:

- **Nhiệm vụ:** Real-life living: everyday family activities, travel, camping, book clubs
- **Phễu:** TOFU-MOFU · **Mức bán hàng:** low · **Product card:** 0-1 soft block at the end
- **Giọng điệu:** độ vui 3/5; Calm, safety-minded, reassuring (Easy, Beginner-Friendly)
- **Định dạng đặc trưng:** Checklists, printable tables, step-by-step, 'Complete' guides
- **Tag thường gặp:** Activities, Outdoor & Adventure
- **Độc giả chính:** Parent / party host (Kids' birthdays, Friendsgiving, Halloween, picnics)
- **URL:** https://magazine.printyourwear.com/blogs/lifestyle

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

Từ hồ sơ brand (áp dụng khi bài chạm chủ đề này):

- **Memorial / sympathy, family loss:** nên gentle, short, practical; no humor; không use grief as a sales hook.
- **Military remembrance:** nên respectful; check facts and dates; không joke slogans, 'sale' framing.
- **Race, heritage, identity:** nên nuanced, sourced, people-first language; không stereotypes, flattening a culture into one image.
- **Religious holidays:** nên inclusive, descriptive; không assume every reader celebrates.
- **Health, relationship status, age:** nên neutral, opt-in humor; không jokes that shame or exclude.
- **Patriot / political symbols:** nên celebrate shared symbols and history; không partisan positions.

## 9. Alt text và tên file

- Alt text: mô tả nội dung ảnh bằng tiếng Anh, có keyword khi tự nhiên, không nhồi từ khoá.
- Tên file: slug tiếng Anh, chữ thường, nối bằng gạch ngang.

## 10. Lịch sử phiên bản

| Version | Ngày | Thay đổi | Người sửa |
| --- | --- | --- | --- |
| 1 | 2026-11-08 | Tạo khung từ brand.json, phần hình ảnh chờ điền | Claude |
