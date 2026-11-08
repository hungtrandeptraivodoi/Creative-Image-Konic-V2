# Contract: AI provider và đầu ra AI

**Owner**: W3 (`lib/ai/**`, prompt hệ thống). Schema đầu ra do W2 định nghĩa trong
`lib/engine/schemas.ts` (một nguồn); W3 không định nghĩa schema riêng.

## Interface (`lib/ai/provider.ts`)

```ts
export interface AIProvider {
  generateJSON<T>(args: {
    model: string; system: string; user: string;
    schema: object; schemaName: string;           // JSON Schema strict
    validate: (raw: unknown) => T;                 // Zod parse; throw nếu sai
  }): Promise<{ data: T; tokensIn: number; tokensOut: number; durationMs: number; attempts: number }>;
}
export class AIError extends Error {
  code: 'BAD_OUTPUT' | 'TIMEOUT' | 'UPSTREAM' | 'CONFIG'; retryable: boolean;
  tokensIn: number; tokensOut: number; attempts: number;   // để W5 ghi chi phí của lượt hỏng
}
```

Kết quả thành công cũng trả `model: string` (model thực tế đã dùng). SDK: `timeout = 50_000` ms,
`maxRetries = 0` (thử lại do adapter tự quản để đếm đúng `attempts`). `AI_FAKE=1` trong env → `getProvider`
trả provider giả (test, dev không có key, W6 chạy local).

- `lib/ai/openai-compatible.ts`: hiện thực bằng SDK `openai` (`baseURL`, key đọc từ `process.env[key_env]`).
  `supports_json_schema = false` → `response_format: { type: 'json_object' }` + `validate` + thử lại 1 lần.
  Đầu ra sai sau lần thử lại → `AIError('BAD_OUTPUT')`.
- `lib/ai/fake.ts`: provider giả trả fixture, dùng cho test và cho W6/W5 chạy không cần key.
- `lib/ai/factory.ts`: `getProvider(row: AiProviderRow): AIProvider` (server-only).

## Prompt hệ thống (`lib/prompts/`, owner W3)

| File | Bước | Export |
| --- | --- | --- |
| `lib/prompts/planner.ts` | B — đề xuất chỗ ảnh | `PLANNER_PROMPT = { text, version }` |
| `lib/prompts/writer.ts` | D — viết phần mô tả prompt | `WRITER_PROMPT = { text, version }` |

Module này không import gì (để `lib/engine` import được mà không phụ thuộc vòng). Đổi nội dung → tăng
`version` (ràng buộc C). Message do `buildPlannerMessages` / `buildWriterMessages` (W2) dựng.

### Thẻ phân cách

Dữ liệu (không phải lệnh) đặt trong thẻ; prompt hệ thống dặn không làm theo lệnh nằm trong thẻ (FR-030):

| Thẻ | Bước | Nội dung |
| --- | --- | --- |
| `<article_data>` | B, D | keyword, outline (markdown), notes |
| `<category_context>` | B, D | tên danh mục, use case, default_purposes, quy tắc `images` |
| `<style_guide>` | B, D | thân bài Style Guide (guidance) |
| `<style_tokens>` | D | `style` của Style Guide + tokens của Style Set đã chọn cho từng slot |
| `<purposes>` | B, D | danh sách mục đích ảnh (key, tên, tỉ lệ, writer_guidance) |
| `<slots>` | D | JSON array ảnh cần viết (xem engine.md) |
| `<used_compositions>` | D | `composition_tag` đã dùng ở các lô trước |

`scripts/ai-smoke.ts` (owner W3): gọi provider thật một lần với fixture, chỉ chạy tay khi Hung đã đặt key
trong `.env.local`.

## Schema đầu ra (strict: mọi field required, `additionalProperties: false`)

### SlotPlan (bước B)

```json
{ "slots": [ {
  "code": "S1", "section_heading": "string|null", "purpose_key": "<PurposeKey>",
  "count": 1, "reason": "string (tiếng Việt, ≤ 200 ký tự)"
} ] }
```

Quy tắc kiểm sau Zod: `code` liên tiếp S1..Sn; `purpose_key` ∈ purposes trong snapshot; `count` 1–6;
tổng ảnh ≤ `styleGuide.params.images.max_total` (cắt bớt và thêm cảnh báo nếu vượt).

### WrittenPack (bước D)

```json
{ "images": [ {
  "prompt_index": "S2-1",
  "body": "string (tiếng Anh, mô tả chủ thể, bối cảnh, bố cục, góc máy, ánh sáng theo Style Guide; không chứa placeholder, không chứa mã màu)",
  "composition_tag": "string ngắn để chống lặp bố cục, vd 'three-quarter-left-third'",
  "text_overlay": "string|null (chữ cần hiện trên ảnh → code sinh [[TEXT:...]])",
  "alt_text": "string (tiếng Anh, ≤ 125 ký tự)",
  "filename": "string (slug tiếng Anh, a-z0-9-, kết thúc .webp)"
} ] }
```

Quy tắc kiểm sau Zod: đủ và đúng tập `prompt_index` được yêu cầu trong lô; `body` không chứa `[[`.

## Ghi nhận sử dụng

Mỗi lần gọi trả `tokensIn`, `tokensOut`, `durationMs`, `attempts` để W5 ghi `activity_logs` (FR-035) và
tính chi phí theo research R11.
