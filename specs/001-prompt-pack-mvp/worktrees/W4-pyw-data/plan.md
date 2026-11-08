# Implementation Plan: W4 — Dữ liệu Print Your Wear

> **Ghi chú người điều phối (2026-11-08)**: hợp đồng đã được điều chỉnh sau khi plan này được viết — xem
> mục "Nhật ký điều chỉnh hợp đồng" trong [plan tổng](../../plan.md). Chỗ nào plan này lệch hợp đồng thì
> **hợp đồng thắng**; các mục "Cần xác nhận" bên dưới đã được trả lời ở đó. Cập nhật plan này trước khi chạy
> `/speckit-tasks`.

**Branch**: `001-w4-pyw-data` | **Date**: 2026-11-08 | **Spec**: [spec.md](./spec.md) | **Plan tổng**: [../../plan.md](../../plan.md)

## Summary

Chuyển hồ sơ brand ([docs/brand.json](../../../../docs/brand.json)) và các Style Guide danh mục
([docs/style-guides/print-your-wear/](../../../../docs/style-guides/print-your-wear/)) thành dữ liệu seed có kiểm
tra, nạp vào Supabase local bằng một script idempotent `scripts/seed-pyw.ts`.

## Technical Context

**Language/Version**: TypeScript 5, Node.js 20 (chạy bằng `tsx`, cài sẵn ở G0 hoặc xin người điều phối)
**Primary Dependencies**: `@supabase/supabase-js` (service role, chỉ local), `yaml`, Zod (`StyleGuideParams` từ W2)
**Storage**: Supabase Postgres local (bảng theo [data-model.md](../../data-model.md))
**Testing**: Vitest — unit cho parse/validate/diff; không gọi DB trong unit test
**Target Platform**: máy dev + Supabase CLI local
**Constraints**: idempotent; không chạy với URL Supabase khác `localhost`/`127.0.0.1` trừ khi có cờ `--allow-remote`
và Hung duyệt (constitution X)
**Scale/Scope**: 1 dự án, 6 danh mục, vài chục dòng

## Constitution Check — PASS

| # | Tuân thủ |
| --- | --- |
| I | Chỉ một script + file dữ liệu; không làm UI nhập liệu |
| II, III | Dùng `StyleGuideParams` của W2, không định nghĩa schema riêng (trừ bản sao tạm trước khi W2 merge) |
| V | Unit test parse/validate/so sánh phiên bản |
| VI | Không đổi data model; thiếu cột → báo người điều phối |
| IX | Service role key đọc từ `.env.local`; không in key ra log |
| X | Chặn chạy lên Supabase thật nếu không có cờ và duyệt |
| A | Brand do dữ liệu, không hard-code trong code ứng dụng; script chỉ đọc file seed |
| C | Style Guide và Brandkit có phiên bản; không sửa đè bản `published` |

## Project Structure

```text
seed/print-your-wear/
├── project.yaml          # projects
├── brandkit.yaml         # brandkits (voice, do, dont, banned_terms; palette: [])
├── categories.yaml       # 6 danh mục
├── style-sets.yaml       # ≥ 1 Style Set mặc định
├── product-types.yaml    # chờ Hung xác nhận
├── personas.yaml         # người mẫu, chờ Hung duyệt
├── refs.yaml             # project_refs (tên/mô tả/link, không file)
└── members.yaml          # email Creator test → cấp dự án (chỉ local)
scripts/
├── seed-pyw.ts           # entry: npm run seed:pyw
└── seed/
    ├── load.ts           # đọc YAML + Markdown frontmatter
    ├── validate.ts       # Zod cho từng file seed + StyleGuideParams
    ├── diff.ts           # so nội dung → quyết định tạo version mới
    └── *.test.ts
docs/style-guides/print-your-wear/*.md   # nội dung do Hung; W4 chỉ sửa khung nếu cần
```

Thêm vào `package.json` (qua người điều phối): `"seed:pyw": "tsx scripts/seed-pyw.ts"`.

## Ánh xạ brand.json → bảng

| Bảng | Nguồn trong `brand.json` | Ghi chú |
| --- | --- | --- |
| `projects` | `brand.storeName`, `brand.url`, `positioning.summary` → `description`, `audience.summary` + `brand.market` → `audience` | `slug: print-your-wear` |
| `brandkits.voice` | `voice.standard` + tagline `"Wear Your Story."` | Dùng như định hướng cảm xúc ảnh |
| `brandkits.do` | Rút gọn cho ảnh: "people-first, inclusive casting", "show products in real everyday use", "warm, practical, lightly playful mood", "celebrate shared symbols and history" | Từ `voice.attributes`, `sensitiveTopics.do` |
| `brandkits.dont` | "no stereotypes or flattening a culture into one image", "no partisan political symbols", "no humor in memorial, sympathy or military remembrance scenes", "no grief used as a sales hook", "no imagery implying 'made in USA' or 'premium cotton' claims", "no fake urgency or sale banners" | Từ `sensitiveTopics.dont`, `editorialStandards.qualityClaims`, `bannedTone` |
| `brandkits.banned_terms` | `vocabulary.avoid` lọc phần là từ đơn: "must-have", "insane", "crazy deal", "best ever", "guaranteed", "#1", "cheap", "knock-off"; thêm `nameUsage.neverInReaderCopy`: "PrintYourWear", "PYW" | Bỏ mục có điều kiện ("unless verified") — chuyển thành `dont` |
| `brandkits.palette`, `typography`, `logo_ref_id` | `visualIdentity.status: unknown` | Rỗng/null (Clarification Q3) |
| `categories` (6) | `categories[]`: `name`, `slug`, `url`; `use_case_notes` = `job` + `signatureFormat` + `commerceLevel`/`productCards` | `default_purposes` = frontmatter Style Guide tương ứng |
| `category_style_guides` | File Markdown, không lấy từ `brand.json` | Theo [contracts/style-guide.md](../../contracts/style-guide.md) |

Bỏ qua (thuộc viết bài, không thuộc ảnh): `headlines`, `articleStructure`, `commerce`, `authors`, `knownGaps`,
`prePublishChecklist`, `writingRules`.

> **Cảnh báo**: `audience.personas` (Busy gift-giver, Parent / party host…) là **độc giả**. Không nạp vào bảng
> `personas`. Bảng `personas` chỉ chứa **người mẫu xuất hiện trong ảnh**. Thông tin độc giả có thể ghi vào
> `projects.audience` hoặc phần hướng dẫn của Style Guide.

## Dữ liệu chờ Hung

| # | Dữ liệu | Đề xuất ban đầu (cần xác nhận) | Chặn bước |
| --- | --- | --- | --- |
| H1 | Nội dung `style.*`, `negative` và các mục `CẦN ĐIỀN` của Style Guide; đổi `status: published` | Ít nhất 3 danh mục cho nghiệm thu L1 (gợi ý: gift-guides, fashion, occasions) | Nghiệm thu, bước 5 |
| H2 | Loại sản phẩm store bán | `t-shirt`, `hoodie`, `sweatshirt`, `mug`, `tote-bag` — **đề xuất, cần xác nhận với catalog thật** | Bước 3 |
| H3 | 1–2 persona người mẫu | Vd `everyday-adult` (30–45, đa dạng, casual), `family-group` — cần Hung duyệt | Bước 3 |
| H4 | `project_refs` | `logo` (nếu có file logo), `product_base` cho từng loại sản phẩm, `design` mẫu; chỉ tên/mô tả/link thư mục team | Bước 3 |
| H5 | Style Set mặc định của dự án | `pyw-editorial-lifestyle` dựa trên `voice.standard` (warm, practical, true-to-life) | Bước 3 |
| H6 | Duyệt `default_purposes` đề xuất trong 6 file Style Guide | Như khung hiện tại | Bước 5 |
| H7 | Duyệt danh sách `dont`/`banned_terms` ở bảng trên | Như trên | Bước 3 |

## Các bước thực thi

1. **Chờ G0** (skeleton). Tạo thư mục `seed/`, `scripts/seed/`; viết schema Zod cho file seed (project, brandkit,
   categories, style-sets, product-types, personas, refs, members); `StyleGuideParams` dùng bản sao tạm theo contract.
2. **Unit test trước** (constitution V): parse frontmatter hợp lệ/không hợp lệ; `published` có `style` rỗng → lỗi
   nêu tên file; `default_purposes` ngoài bộ 9 → lỗi; `diff` cùng nội dung → không tạo phiên bản; nội dung đổi →
   `version + 1`.
3. **Viết file seed** từ `brand.json` theo bảng ánh xạ; các mục H2–H5, H7 ghi đề xuất và đánh dấu
   `# TODO(Hung): xác nhận` cho tới khi được duyệt.
4. **Chờ G1** (bảng + RLS trên `main`). Viết `seed-pyw.ts`: kiểm URL local → validate toàn bộ → upsert theo khoá
   tự nhiên (`projects.slug`, `categories(project_id, slug)`, `product_types(project_id, key)`, `personas(project_id, key)`,
   `project_refs(project_id, type, key)`) → Brandkit/Style Set/Style Guide: so với bản mới nhất, khác thì tạo
   `version + 1`, nếu `published` thì chuyển bản `published` cũ sang `archived` → cấp `project_members` cho email
   trong `members.yaml` (user phải tồn tại sẵn) → in tóm tắt.
5. **Nạp Style Guide** khi H1 xong; chạy quickstart kịch bản 3; khi W2 merge thì thay bản sao tạm bằng import
   `StyleGuideParams`.

Thứ tự chặn: G0 → (bước 1–3) → G1 + H2–H5, H7 → bước 4 → H1, H6 → bước 5 → nghiệm thu.

## Complexity Tracking

Không có vi phạm.
