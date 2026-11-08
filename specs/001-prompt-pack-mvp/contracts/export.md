# Contract: Export Prompt Pack

**Owner**: W6 (`lib/export/*`, hàm thuần, có unit test). W5 gọi từ route export. Thứ tự dòng = `gen_order`.

Trường chung (FR-032), cùng thứ tự ở cả 3 định dạng:

| Field | CSV header | Ghi chú |
| --- | --- | --- |
| `index` | `image` | S2-1 |
| `genOrder` | `order` | 1..n |
| `purposeKey` | `purpose` | |
| `aspect` | `aspect` | |
| `promptText` | `prompt` | giữ nguyên placeholder |
| `negative` | `negative` | |
| `refs` | `references` | CSV: `[[REF:LOGO]] = Logo; …` nối bằng `; ` |
| `dependsOn` | `depends_on` | CSV: nối bằng `, ` |
| `altText` | `alt_text` | |
| `filename` | `filename` | |
| `warnings` | `warnings` | CSV: message nối bằng ` | ` |
| `isManualEdit` | `manual_edit` | `yes`/`no` |

- **JSON**: `{ "session": {keyword, project, category, exportedAt, snapshot: {brandkitVersion, styleGuideVersion}}, "filesToPrepare": RefEntry[], "images": [...] }`.
- **CSV**: RFC 4180, UTF-8 có BOM, CRLF, mọi ô có dấu phẩy/ngoặc kép/xuống dòng được bọc `"`. Tên file
  `<keyword-slug>-prompt-pack.csv`.
- **Markdown**: tiêu đề bài, mục "File cần chuẩn bị", rồi mỗi ảnh một mục `### S2-1 (order 3) — product_on_model 4:5`
  với prompt trong code fence `text`, các trường còn lại dạng danh sách.
- **Clipboard (copy cả pack)**: như Markdown nhưng bỏ phần đầu.
