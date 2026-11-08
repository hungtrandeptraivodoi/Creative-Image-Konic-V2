# Contract: Prompt Engine (`lib/engine`)

**Owner**: W2. **Consumers**: W3 (schemas), W5 (gọi pipeline), W6 (types để hiển thị). Mọi hàm là hàm
thuần: không import Supabase, `openai`, `next/*`, `lib/ai`. Được import `lib/prompts` (chỉ chứa chuỗi văn bản
và số phiên bản, không import gì — owner W3). Thay đổi chữ ký = breaking change (constitution VI).

Hướng phụ thuộc: `lib/prompts` ← `lib/engine` ← `lib/ai` (chỉ import `schemas`) ← `lib/api` ← `app/`.

## Module

| File | Export chính |
| --- | --- |
| `lib/engine/types.ts` | Mọi type dưới đây |
| `lib/engine/schemas.ts` | Zod schema + JSON Schema cho đầu ra AI (xem [ai.md](./ai.md)), cho `StyleGuideParams` (xem [style-guide.md](./style-guide.md)), cho request body API |
| `lib/engine/outline.ts` | `parseOutline` |
| `lib/engine/resolver.ts` | `buildSnapshot`, `resolveSlotContext` |
| `lib/engine/planner-input.ts` | `buildPlannerMessages` |
| `lib/engine/writer-input.ts` | `buildWriterMessages`, `splitIntoBatches` |
| `lib/engine/assembler.ts` | `finishPrompt`, `assemblePack` |
| `lib/engine/graph.ts` | `computeGenOrder` |
| `lib/engine/validator.ts` | `validatePack`, `validatePrompt` |
| `lib/engine/constants.ts` | `PROMPT_MAX_CHARS = 2000`, `OUTLINE_MAX_CHARS = 15000`, `BATCH_THRESHOLD = 20`, `BATCH_SIZE = 10` |
| `lib/engine/__fixtures__/` | Fixture dùng chung cho mọi worktree: snapshot PYW không palette, snapshot có palette, outline 6 H2, SlotPlan và WrittenPack giả |

Phiên bản prompt hệ thống **không** nằm trong engine: `lib/prompts/{planner,writer}.ts` (W3) export
`{ text: string; version: string }`; message builder đọc từ đó và ghi version vào `templateVersions`.

## Types chính

```ts
type Aspect = '16:9' | '3:2' | '4:3' | '1:1' | '4:5' | '2:3';
type PurposeKey = 'hero' | 'scene' | 'howto_step' | 'product_flatlay' | 'product_on_model'
  | 'product_lifestyle' | 'design_gallery' | 'comparison' | 'pinterest_pin';
type RefType = 'LOGO' | 'PRODUCT' | 'MODEL' | 'DESIGN' | 'GEN' | 'TEXT';

interface OutlineHeading { level: 2 | 3; text: string; index: number }
interface ParsedOutline { headings: OutlineHeading[]; h2Count: number; charCount: number }

interface ContextSnapshot {            // lưu trong sessions.context_snapshot
  takenAt: string;
  project: { id: string; slug: string; name: string; audience: string | null };
  brandkit: { id: string; version: number; palette: { name: string; hex: string; role: string }[];
              do: string[]; dont: string[]; bannedTerms: string[]; voice: string | null;
              logoRefKey: string | null };
  category: { id: string; slug: string; name: string; useCaseNotes: string | null;
              defaultPurposes: PurposeKey[] };
  styleGuide: { id: string; version: number; params: StyleGuideParams; guidanceMd: string } | null;
  styleSets: { key: string; version: number; name: string; tokens: StyleTokens; negative: string;
               checkBrandHex: boolean }[];          // chỉ các Style Set creator được phép
  defaultStyleSetKey: string | null;
  purposes: PurposeTemplate[];                       // bản published
  productTypes: { key: string; name: string; mockupNotes: string | null; baseRefKey: string | null }[];
  personas: { key: string; name: string; description: Record<string, string>; photoRefKey: string | null }[];
  refs: { type: string; key: string; name: string; description: string | null; linkUrl: string | null }[];
  limits: { promptMaxChars: number };                // mặc định 2000
}

interface SlotDraft {                  // = image_slots (không có id DB)
  code: string; position: number; sectionHeading: string | null; purposeKey: PurposeKey;
  aspect: Aspect; count: number; styleSetKey: string | null; aiReason: string | null;
  options: { productTypeKey?: string; personaKey?: string | 'none';
             designRef?: { mode: 'library' | 'later'; key?: string };
             setting?: string; notes?: string; dependsOnSlot?: string };
}

interface RefEntry { placeholder: string; type: RefType; key: string; label: string;
                     source: 'project' | 'creator' | 'pack'; linkUrl: string | null }

interface Issue { code: IssueCode; severity: 'error' | 'warning'; promptIndex: string | null;
                  slotCode: string | null; message: string }   // message tiếng Việt cho UI
type IssueCode = 'PLACEHOLDER_MISSING_REF' | 'REF_NOT_IN_PROMPT' | 'REQUIRED_REF_MISSING'
  | 'GEN_CYCLE' | 'GEN_TARGET_MISSING' | 'PROMPT_TOO_LONG' | 'BANNED_TERM' | 'BRAND_HEX_MISSING'
  | 'TEXT_CHECK_REMINDER' | 'NO_STYLE_GUIDE';

interface PackPrompt {                 // = prompts (không có id DB)
  index: string; slotCode: string; genOrder: number; body: string; promptText: string;
  originalText: string; negative: string; refs: RefEntry[]; dependsOn: string[];
  altText: string; filename: string; warnings: Issue[]; isManualEdit: boolean;
  model: string; templateVersions: Record<string, number | string>;
}
```

## Hàm

```ts
parseOutline(md: string): ParsedOutline
buildSnapshot(input: SnapshotSource): ContextSnapshot          // SnapshotSource = các dòng DB đã đọc (W5 truyền vào)
resolveSlotContext(snap: ContextSnapshot, slot: SlotDraft): ResolvedSlot   // áp thứ tự ưu tiên ràng buộc A
buildPlannerMessages(snap, intake: { keyword; outline: ParsedOutline; outlineMd; notes }): { system; user }
buildWriterMessages(snap, intake, slots: SlotDraft[], batch: { indexes: string[]; usedCompositions: string[] }): { system; user }
splitIntoBatches(slots: SlotDraft[]): string[][]               // ≤ 20 ảnh → 1 lô; > 20 → lô 10
finishPrompt(snap, slot: ResolvedSlot, written: WrittenImage): Omit<PackPrompt, 'genOrder' | 'warnings' | 'model' | 'templateVersions'>
computeGenOrder(prompts: Pick<PackPrompt,'index'|'dependsOn'>[]): { order: Record<string, number> } | { cycle: string[] }
assemblePack(snap, slots, written: WrittenImage[], meta: { model: string }): { prompts: PackPrompt[]; issues: Issue[] }
validatePack(snap, slots, prompts: PackPrompt[]): Issue[]
validatePrompt(snap, slot, prompt: PackPrompt, allIndexes: string[]): Issue[]   // dùng khi sửa tay
```

`ContextSnapshot.limits.promptMaxChars` lấy từ `PROMPT_MAX_CHARS`. `category.defaultPurposes` trong snapshot =
`styleGuide.params.default_purposes` nếu có Style Guide published, ngược lại = `categories.default_purposes`.

Placeholder design: `designRef.mode = 'library'` → `[[REF:DESIGN:<key>]]`; `mode = 'later'` →
`[[REF:DESIGN#n]]`, `n` đánh số 1..k theo thứ tự xuất hiện trong cả pack (gen_order rồi index), cùng một chỗ
ảnh gallery dùng chung một số.

Message bước B/D: phần dữ liệu đặt trong các thẻ (xem [ai.md](./ai.md) §Thẻ phân cách); danh sách ảnh cần viết
trong thẻ `<slots>` là JSON array `{prompt_index, slot_code, purpose_key, aspect, section_heading, options}`
để provider giả đọc được.

Quy tắc bắt buộc của `finishPrompt` (ràng buộc A): `promptText = body + "\n" + refsInstructions +
"\nAvoid: " + dont(brandkit ∪ styleGuide) + "\nAspect ratio " + aspect + [brand colors nếu palette ≠ ∅] +
"\n" + negative`. Placeholder chỉ được sinh ở đây (AI không viết placeholder). `GEN` đến từ
`options.dependsOnSlot` hoặc quy tắc mục đích (`pinterest_pin` → hero nếu có; `howto_step` → bước trước nếu
Creator bật).

## Test bắt buộc (constitution V)

Outline 6 H2 mẫu; GEN vòng; GEN trỏ slot đã xoá; placeholder thiếu/thừa; `dont` của Brandkit không bị
ghi đè bởi notes; palette rỗng → không có dòng màu và không báo `BRAND_HEX_MISSING`; prompt > 2.000 ký tự;
từ cấm (không phân biệt hoa thường, theo ranh giới từ); `validatePrompt` sau sửa tay.
