# Contract: HTTP API (Route Handlers)

**Owner**: W5. **Consumer**: W6. Mọi route `runtime = 'nodejs'`, yêu cầu phiên đăng nhập (cookie Supabase),
dùng client có RLS của người gọi; chỉ `lib/log.ts` và đọc `ai_providers` dùng service role. Body/response
JSON; schema request kiểm bằng Zod từ `lib/engine/schemas.ts`.

Lỗi chuẩn: `{ "error": { "code": string, "message": string (tiếng Việt), "issues"?: Issue[], "details"?: object } }`
với HTTP: 400 `VALIDATION`, 401 `UNAUTHENTICATED`, 403 `FORBIDDEN` (không phải thành viên và không phải
admin / tài khoản khoá), 404 `NOT_FOUND`, 409 `CONFIRM_REQUIRED` (cần xác nhận ghi đè), 422
`NO_PUBLISHED_BRANDKIT`, 429 `PROVIDER_DAILY_LIMIT`, 502 `AI_BAD_OUTPUT` / `AI_UPSTREAM`, 503
`NO_ACTIVE_PROVIDER`.

`409 CONFIRM_REQUIRED` có `details: { counts: { slots, prompts, manualEdits }, manualEditIndexes: string[] }`.

**Kiểm quyền** ở mọi route có ghi hoặc gọi AI: `is_admin() OR is_member(project_id)` và tài khoản `active`
(FR-007, FR-009) — Admin không cần là thành viên.

Dữ liệu đọc đơn giản W6 đọc thẳng bằng Supabase client có RLS trong Server Component, không cần route:
danh sách dự án, danh mục, phiên của tôi, cờ "dự án có Brandkit published" và "danh mục có Style Guide
published + version" (đọc `brandkits`/`category_style_guides` với `status = 'published'`).

### Kiểu bổ sung cho `GET /api/sessions/:sid`

```ts
snapshotSummary: { takenAt: string; brandkitVersion: number; styleGuideVersion: number | null;
                   hasPalette: boolean } | null;               // null trước lần phân tích đầu
editorOptions: { purposes: { key; name; group: 'general'|'product'; defaultAspect; requiredRefs; optionalRefs }[];
                 styleSets: { key; name }[]; defaultStyleSetKey: string | null;
                 productTypes: { key; name }[]; personas: { key; name }[];
                 designRefs: { key; name }[] } | null;         // lấy từ snapshot; null trước lần phân tích đầu
filesToPrepare: RefEntry[];                                     // gộp, bỏ trùng từ refs của mọi prompt
```

**Fixture**: W5 sở hữu `tests/fixtures/api/*.json` (một response mẫu cho mỗi route, kể cả lỗi 409/422/429/502)
và đưa lên `main` sớm nhất (trước khi xong route) để W6 mock.

| Method & path | Body | Response 200 | Ghi log |
| --- | --- | --- | --- |
| `POST /api/sessions` | `{ projectId, categoryId }` | `{ session }` | `session.create` |
| `GET /api/sessions/:sid` | — | `{ session, slots: SlotDraft[], prompts: PackPrompt[], issues: Issue[], snapshotSummary, editorOptions, filesToPrepare }` | — |
| `PATCH /api/sessions/:sid` | `{ keyword?, outlineMd?, notes? }` | `{ session, outline: ParsedOutline, issues: Issue[] }` — outline không hợp lệ vẫn được lưu, `issues` cho biết lý do; UI khoá nút "Phân tích" | — |
| `POST /api/sessions/:sid/plan` | `{ confirmReplace?: boolean }` | `{ session, slots, warnings: Issue[] }` | `slots.plan` (+tokens, cost, model, duration) |
| `PUT /api/sessions/:sid/slots` | `{ slots: SlotDraft[] }` | `{ slots, issues }` | `slots.edit` (diff tóm tắt) |
| `POST /api/sessions/:sid/prompts` | `{ confirmReplace?: boolean }` | `{ session, prompts, issues }` | `prompts.generate` (+số prompt, số cảnh báo, tokens, cost) |
| `PATCH /api/prompts/:pid` | `{ promptText }` hoặc `{ restoreOriginal: true }` | `{ prompt }` (warnings đã kiểm lại) | — |
| `POST /api/sessions/:sid/export` | `{ format: 'json'\|'csv'\|'md'\|'clipboard' }` | file (Content-Disposition) hoặc `{ text }` cho clipboard | `pack.export` (format) |

Quy tắc:

- `plan`: lần đầu chụp snapshot (`buildSnapshot`), các lần sau dùng lại. Nếu phiên đã có slot/prompt mà
  `confirmReplace` ≠ true → 409 `CONFIRM_REQUIRED` kèm số slot/prompt sẽ bị thay (FR-014a).
- `prompts`: nếu có prompt `isManualEdit` mà `confirmReplace` ≠ true → 409 (FR-027a). Pack có lỗi vẫn
  lưu và trả về kèm `issues`; UI hiển thị lỗi trên thẻ.
- Trước mỗi lần gọi AI: kiểm quyền (xem trên), kiểm `daily_call_limit` bằng đếm `activity_logs` action
  `slots.plan`/`prompts.generate` có `ok = true` trong ngày (múi giờ UTC) (FR-037). Lượt AI hỏng vẫn ghi log
  với `ok = false` và tokens đã tiêu từ `AIError`.
- Thay slot/prompt dùng RPC `replace_session_slots` / `replace_session_prompts` (data-model) để nguyên tử.
- Copy một prompt đơn lẻ làm ở client, không gọi API, không ghi log.
