# Contract: Style Guide danh mục

**Nguồn soạn thảo**: `docs/style-guides/<project-slug>/<category-slug>.md` (xem
[docs/style-guides/README.md](../../../docs/style-guides/README.md)). **Schema** (Zod, trong
`lib/engine/schemas.ts`, owner W2): `StyleGuideParams`. **Nạp vào DB**: script seed của W4 →
bảng `category_style_guides`.

```ts
StyleGuideParams = {
  project: string; category: string; category_name: string;
  version: number; status: 'draft' | 'published' | 'archived'; updated: string; owner: string;
  default_purposes: PurposeKey[];
  images: { hero: boolean; per_h2: number; max_total: number; pinterest_pin: boolean };
  style: { medium: string; lighting: string; composition: string; mood: string; camera: string;
           color_treatment: string };
  negative: string; check_brand_hex: boolean;
  dont: string[]; banned_terms: string[];
}
```

- Thân bài (sau frontmatter) lưu nguyên văn ở `markdown`; snapshot giữ `guidanceMd` = thân bài.
- Seed chỉ đổi DB khi nội dung khác bản mới nhất: tạo `version + 1`; `status: published` thì đưa bản cũ về
  `archived`.
- Bản `published` phải có `style.*` không rỗng (seed từ chối nếu rỗng).
