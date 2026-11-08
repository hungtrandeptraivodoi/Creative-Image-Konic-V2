# Implementation Plan: W6 — Giao diện Creator

> **Ghi chú người điều phối (2026-11-08)**: hợp đồng đã được điều chỉnh sau khi plan này được viết — xem
> mục "Nhật ký điều chỉnh hợp đồng" trong [plan tổng](../../plan.md). Chỗ nào plan này lệch hợp đồng thì
> **hợp đồng thắng**; các mục "Cần xác nhận" bên dưới đã được trả lời ở đó. Cập nhật plan này trước khi chạy
> `/speckit-tasks`.

**Branch**: `001-w6-creator-ui` | **Date**: 2026-11-08 | **Spec**: [spec.md](./spec.md) (phạm vi) ·
[../../spec.md](../../spec.md) (cha) · **Plan tổng**: [../../plan.md](../../plan.md)

## Summary

Làm 4 màn hình cho Creator (danh sách dự án, chọn danh mục + tạo phiên, trang phiên 3 bước, và các hộp
thoại xác nhận), cùng formatter export thuần `lib/export/`. Giao diện gọi API theo
[contracts/api.md](../../contracts/api.md); trước khi W5 merge thì chạy trên mock API cùng hợp đồng.
Ưu tiên usability: mặc định đã điền sẵn, chỉ hỏi Creator chỗ cần quyết, trạng thái rõ ràng, dùng được
bằng bàn phím và ở 360px (constitution IV).

## Technical Context

**Language/Version**: TypeScript 5 strict, React 19, Next.js 15 App Router

**Primary Dependencies**: Tailwind CSS 4, shadcn/ui, `@dnd-kit/core` + `@dnd-kit/sortable`, `lucide-react`
(đã cài ở G0). Không thêm thư viện state, form hay CSV.

**Storage**: không; dữ liệu đọc qua Supabase client có RLS (Server Component) và Route Handlers của W5

**Testing**: Vitest cho `lib/export/**` và các hàm thuần của UI (`lib/creator-ui/*` nếu có); UI kiểm bằng
quickstart thủ công (research R3)

**Target Platform**: trình duyệt desktop (Chrome/Edge/Safari bản hiện hành), dùng được từ 360px

**Project Type**: web application (phần giao diện của project Next.js chung)

**Performance Goals**: phản hồi thao tác cục bộ (sửa slot, gõ outline, preview heading) không trễ thấy được;
màn hình chờ AI hiển thị tiến trình suốt tối đa 60 giây

**Constraints**: chỉ sửa thư mục W6 sở hữu (`app/(creator)/**`, `components/**`, `lib/export/**`,
`app/globals.css`); không đổi hợp đồng; không gọi AI hay service role từ client

**Scale/Scope**: 4 màn hình, ~15 component nghiệp vụ, 1 module export

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.* — **PASS**

| # | Áp dụng cho W6 |
| --- | --- |
| I MVP-first | Không làm lịch sử phiên, không state library, không E2E; mock API chỉ là fixture tĩnh + handler nhỏ |
| II Module hoá | Component gọi API qua một client mỏng `components/creator/api-client.ts`; formatter export thuần, không import React |
| III Tái sử dụng | Dùng shadcn/ui (Button, Card, Dialog, AlertDialog, Select, Input, Textarea, Badge, Tabs, Tooltip, Toast/Sonner, Skeleton, DropdownMenu, Sheet); dùng types của W2, schema request của W2 |
| IV Usability, responsive | Toàn bộ mục "Đặc tả thiết kế UI" dưới đây |
| V Test | Unit test formatter export (SC-010) và hàm thuần của UI (highlight placeholder, slot reorder) |
| VI Không breaking | Không đổi `contracts/`; lệch hợp đồng → báo người điều phối |
| VII Bám spec | Mỗi màn hình ghi FR phụ trách |
| VIII ADR | Không có quyết định kiến trúc mới (theo R9); nếu phải thêm dependency → báo người điều phối |
| IX Secret | Client không đọc env ngoài `NEXT_PUBLIC_*` |
| X Duyệt | Chỉ commit trên `001-w6-creator-ui`; không push/merge |
| A–E | UI không tự ghép prompt, không tự sinh placeholder; chỉ hiển thị dữ liệu API trả về; sửa tay gửi về `PATCH /api/prompts/:pid` để kiểm lại bằng Validator |

## Project Structure

### Documentation

```text
specs/001-prompt-pack-mvp/worktrees/W6-creator-ui/
├── spec.md      # phạm vi
├── plan.md      # file này
└── tasks.md     # /speckit-tasks (sau)
```

### Source Code (thư mục W6 sở hữu)

```text
app/(creator)/
├── layout.tsx                              # header: tên app, tên người dùng, đăng xuất
├── projects/page.tsx                       # M1 danh sách dự án (C1)
├── projects/[pid]/page.tsx                 # M2 chọn danh mục + tạo phiên (C1)
└── projects/[pid]/sessions/[sid]/page.tsx  # M3 trang phiên 3 bước (C2–C6)
components/
├── ui/                                     # shadcn (copy)
└── creator/
    ├── api-client.ts                       # fetch tới /api/*, map lỗi chuẩn → ApiError
    ├── session-stepper.tsx                 # thanh 3 bước
    ├── intake-form.tsx                     # keyword, outline, notes, preview heading
    ├── outline-preview.tsx
    ├── slot-list.tsx                       # sortable + phím tắt
    ├── slot-card.tsx                       # cấu hình 1 chỗ ảnh
    ├── product-fields.tsx                  # trường phụ nhóm sản phẩm
    ├── pack-view.tsx                       # danh sách thẻ prompt + toolbar export
    ├── prompt-card.tsx                     # 1 prompt: copy, sửa tay, khôi phục, lỗi
    ├── placeholder-text.tsx                # tô placeholder
    ├── files-to-prepare.tsx
    ├── issue-list.tsx
    ├── confirm-replace-dialog.tsx          # 409 CONFIRM_REQUIRED
    └── ai-progress.tsx                     # trạng thái chờ AI
lib/export/
├── fields.ts                               # thứ tự + header 12 trường
├── json.ts, csv.ts, markdown.ts            # formatter thuần
└── *.test.ts
mocks/creator-api/                          # chỉ dùng khi NEXT_PUBLIC_MOCK_API=1 (xem Mock)
```

**Structure Decision**: theo plan tổng; mock đặt trong `mocks/creator-api/` thuộc W6, xoá hoặc giữ làm
fixture test khi nối API thật (báo người điều phối vì thư mục này không có trong bản đồ sở hữu).

## Đặc tả thiết kế UI

### 1. Sơ đồ màn hình và điều hướng

```text
/login (W1)
  └─► M1 /projects  ── chọn thẻ dự án ──► M2 /projects/[pid]
                                            └─ chọn danh mục + "Bắt đầu bài mới"
                                               (POST /api/sessions) ──► M3 /projects/[pid]/sessions/[sid]
M3: [1 Nhập bài] ─Phân tích─► [2 Chỗ ảnh] ─Tạo prompt─► [3 Prompt Pack]
     ▲ quay lại bất kỳ bước nào qua stepper; bước chưa mở khoá bị disable
```

- Bước mở khoá theo `session.status`: `draft` → chỉ bước 1; `planned` → 1, 2; `prompted` → 1, 2, 3.
- URL giữ bước hiện tại bằng `?step=intake|slots|pack` để F5 không mất vị trí; mặc định mở bước cao nhất
  đã mở khoá.
- Breadcrumb: `Dự án / <tên dự án> / <danh mục> · <keyword hoặc "Bài mới">`.

### 2. Bố cục chung

- Container tối đa 1200px, lề ngang 16px ở < 640px, 24px ở ≥ 640px. Không phần tử nào có chiều rộng cố
  định > 328px (360 − 2×16). Bảng chuyển thành thẻ ở < 768px. Chuỗi dài (prompt, tên file) dùng
  `break-words`; code/prompt trong khối có `overflow-x: auto` riêng, không làm trang cuộn ngang.
- Thanh hành động chính của mỗi bước dính dưới (`sticky bottom-0`) ở < 640px, nằm cuối nội dung ở desktop.
- Thông báo kết quả ngắn dùng toast (Sonner); lỗi chặn thao tác hiển thị inline cạnh chỗ gây lỗi.

### 3. M1 — Danh sách dự án (FR-010)

- **Desktop**: lưới thẻ 3 cột (`Card`): tên dự án, domain, số danh mục. **< 640px**: 1 cột.
- **Loading**: 3 `Skeleton` thẻ. **Empty**: "Bạn chưa được cấp dự án nào. Liên hệ Admin để được cấp quyền."
  **Error**: "Không tải được danh sách dự án." + nút "Thử lại".
- Cả thẻ là link (Enter mở); focus ring rõ.

### 4. M2 — Chọn danh mục và tạo phiên (FR-010)

- **Desktop**: 2 cột: trái là cây danh mục (danh sách có thụt lề theo cha/con, chọn bằng radio group),
  phải là ghi chú use case của danh mục đang chọn + mục đích ảnh mặc định (`Badge`) + trạng thái Style
  Guide ("Có Style Guide v3" / cảnh báo vàng "Danh mục chưa có Style Guide — sẽ dùng phong cách mặc định
  của dự án"). **< 640px**: `Select` danh mục ở trên, ghi chú bên dưới.
- Nút chính "Bắt đầu bài mới" (disable khi chưa chọn danh mục) → `POST /api/sessions` → chuyển tới M3.
- Nếu dự án chưa có Brandkit đã xuất bản (API trả lỗi): banner đỏ "Dự án chưa có Brandkit đã xuất bản.
  Liên hệ Admin." và disable nút.

### 5. M3 bước 1 — Nhập bài (FR-011, FR-012, FR-013, FR-014a)

- **Desktop**: 2 cột 60/40. Trái: `Input` "Keyword" (bắt buộc), `Textarea` "Outline (markdown H2/H3)"
  cao tối thiểu 16 dòng có bộ đếm "8.240 / 15.000 ký tự", `Textarea` "Ghi chú (tuỳ chọn)". Phải:
  `outline-preview` "Heading đã nhận" — danh sách H2 (đậm) và H3 (thụt lề), cập nhật khi gõ (debounce
  300ms, parse ở client bằng `parseOutline` của W2). **< 640px**: preview là `Collapsible` dưới outline,
  tiêu đề "Heading đã nhận (6 H2, 11 H3)".
- Lưu nháp: `PATCH /api/sessions/:sid` khi rời ô (blur) và trước khi Phân tích; chỉ báo "Đã lưu" nhỏ.
- Kiểm trước khi gọi API, hiển thị inline và disable nút:
  - Không có H2: "Outline cần ít nhất một heading H2 (dòng bắt đầu bằng ##)."
  - Vượt 15.000 ký tự: "Outline dài 16.020 ký tự, vượt giới hạn 15.000. Hãy rút gọn."
  - Thiếu keyword: "Nhập keyword của bài."
- Nút chính **"Phân tích"**. Nếu phiên đã có chỗ ảnh/prompt → API trả 409 → `confirm-replace-dialog`.
- Đang phân tích: `ai-progress` (xem §10); form khoá chỉ đọc.

### 6. M3 bước 2 — Chỗ ảnh (FR-015…FR-019)

- Đầu bước: dòng tóm tắt "8 chỗ ảnh · 10 ảnh" + cảnh báo từ API (`warnings`) dạng `Alert`.
- `slot-list`: danh sách thẻ `slot-card` theo `position`, kéo thả bằng tay nắm (`GripVertical`).
- **slot-card** (thu gọn mặc định, mở bằng nút "Cấu hình"/Enter):
  - Dòng đầu: mã `S3` (badge), heading gắn với (hoặc "Không gắn heading"), mục đích (`Badge`), tỉ lệ,
    "× 3 ảnh" nếu count > 1. Dòng 2 (chữ nhỏ, mờ): "Lý do AI đề xuất: …".
  - Khi mở: `Select` Mục đích (9 mục, nhãn tiếng Việt + key), `Select` Tỉ lệ (mặc định theo mục đích, hiển
    thị "(mặc định)"), stepper Số ảnh 1–6 (`−`/`+`, có nhãn "Số ảnh"), `Select` Style Set (chỉ danh sách
    được phép; mục mặc định ghi "(mặc định)"), `Select` "Cần ảnh khác trước" (tuỳ chọn, danh sách các
    slot khác), `Textarea` Ghi chú.
  - **Trường phụ nhóm sản phẩm** (`product-fields`, chỉ hiện khi `purpose.group = 'product'`, có animation
    mở ngắn; ẩn hoàn toàn với mục đích khác): `Select` Loại sản phẩm (danh sách của store), `Select`
    Người mẫu (persona + "Không có người"), Design tham chiếu: `RadioGroup` "Chọn từ danh sách" (kèm
    `Select` tham chiếu loại design) / "Gắn sau", `Input` Bối cảnh. Trường bắt buộc của mục đích chưa chọn
    → viền đỏ + "Mục đích này cần chọn loại sản phẩm."
  - Menu "⋯" (`DropdownMenu`): "Lên trên", "Xuống dưới", "Nhân đôi", "Xoá".
- Nút "+ Thêm chỗ ảnh" cuối danh sách: thêm slot mã kế tiếp với mục đích mặc định đầu tiên của danh mục.
- Xoá slot đang được slot khác chọn làm "Cần ảnh khác trước": `AlertDialog` "S2 đang là ảnh cần tạo trước
  của S5, S7. Xoá S2 sẽ bỏ liên kết này. Tiếp tục?" (US3 kịch bản 4).
- Lưu: `PUT /api/sessions/:sid/slots` tự động sau mỗi thay đổi (debounce 800ms), chỉ báo "Đang lưu… / Đã
  lưu"; lỗi lưu → toast + nút "Thử lại".
- Nếu phiên đang `prompted` mà Creator sửa slot: banner vàng "Prompt Pack hiện tại đã cũ so với chỗ ảnh.
  Bấm Tạo prompt để cập nhật."
- Nút chính **"Tạo prompt"** (disable khi còn lỗi bắt buộc ở slot). Có prompt sửa tay → 409 → dialog.
- **Desktop**: danh sách 1 cột rộng tối đa 880px; thẻ mở hiển thị trường 2 cột. **< 640px**: trường 1 cột,
  tay nắm kéo vẫn có nhưng hành động chính là menu "⋯".

### 7. Kéo thả và phím tắt thay thế

- `@dnd-kit/sortable` với `KeyboardSensor`: focus vào tay nắm → Space để nhấc, ↑/↓ để di chuyển, Space để
  thả, Esc để huỷ. Thông báo cho trình đọc màn hình qua `announcements` tiếng Việt ("Đã nhấc S3, vị trí 3
  trên 8", "Đã thả S3 ở vị trí 5").
- Thay thế không cần kéo: menu "⋯" → "Lên trên"/"Xuống dưới"; phím tắt khi focus trong thẻ: `Alt+↑` /
  `Alt+↓`. Mã slot (S1…) giữ nguyên khi đổi thứ tự; chỉ `position` đổi.

### 8. M3 bước 3 — Prompt Pack (FR-020…FR-033)

- **Toolbar** (đầu bước, sticky ở desktop): tổng "10 prompt · 2 lỗi · 3 cảnh báo"; nút "Copy cả pack";
  `DropdownMenu` "Export" → JSON / CSV / Markdown; nút "Tạo lại cả pack" (phụ). Còn lỗi → `Alert` đỏ
  "Pack còn 2 lỗi. Sửa chỗ ảnh rồi tạo lại, hoặc sửa tay trên thẻ." kèm link nhảy tới thẻ lỗi đầu tiên.
  Export vẫn được phép khi còn lỗi (lỗi đi kèm cột `warnings`).
- **files-to-prepare** "File cần chuẩn bị" (`Card`, thu gọn được): mỗi dòng: placeholder (tô màu như §9),
  nhãn, các ảnh dùng nó ("dùng cho S2-1, S4-1"), link thư mục nếu có (`link_url`, mở tab mới). Trống →
  "Pack này không cần gắn file nào."
- **prompt-card** (theo `genOrder`):
  - Đầu thẻ: `#3` thứ tự tạo, mã `S2-1`, mục đích + tỉ lệ, badge "đã sửa tay" (nếu `isManualEdit`),
    nhãn phụ thuộc "Cần ảnh S1-1 trước" (mỗi phụ thuộc một badge, bấm để cuộn tới thẻ đó).
  - Thân: prompt trong khối chữ đơn cách, placeholder tô theo §9; nút "Copy" (toast "Đã copy S2-1").
  - Cảnh báo `TEXT_CHECK_REMINDER`: `Alert` vàng "Ảnh có chữ — kiểm chữ trước khi dùng."
  - Lỗi/cảnh báo Validator (`issue-list`): đỏ = error, vàng = warning, mỗi dòng có icon + message tiếng
    Việt từ API.
  - Phần phụ (`Collapsible` "Chi tiết"): negative, tham chiếu, alt text, tên file (mỗi trường có nút copy).
  - **Sửa tay**: nút "Sửa" → khối chữ thành `Textarea` + bộ đếm "1.840 / 2.000"; "Lưu" →
    `PATCH /api/prompts/:pid { promptText }` → thay warnings bằng kết quả kiểm lại; "Huỷ"; Esc = Huỷ,
    Ctrl/Cmd+Enter = Lưu. Thẻ đã sửa tay có nút "Khôi phục bản gốc" (`AlertDialog` xác nhận) →
    `{ restoreOriginal: true }`.
- **Desktop**: 1 cột thẻ rộng tối đa 880px, mục "File cần chuẩn bị" ở cột phải 320px (≥ 1200px) hoặc trên
  cùng (< 1200px). **< 640px**: toolbar gom vào nút "Thao tác" (`Sheet` từ dưới lên).

### 9. Tô placeholder (không chỉ dựa vào màu)

| Loại | Kiểu hiển thị | Màu nền / chữ (đạt ≥ 4.5:1) | Icon kèm |
| --- | --- | --- | --- |
| `REF` | chip viền liền, chữ đậm | nền xanh dương nhạt / chữ xanh dương đậm | `Paperclip` |
| `GEN` | chip viền nét đứt | nền tím nhạt / chữ tím đậm | `Link2` |
| `TEXT` | chip gạch chân lượn | nền hổ phách nhạt / chữ nâu đậm | `Type` |

- Giữ nguyên chuỗi `[[…]]` trong chip để copy/đọc chính xác; icon có `aria-hidden`, chip có `title` và nhãn
  cho trình đọc màn hình: "Tham chiếu: logo", "Ảnh tạo trước: S1-1", "Chữ trên ảnh: headline".
- Tách placeholder bằng hàm thuần `splitPlaceholders(text)` (regex `\[\[(REF|GEN|TEXT):[^\]]+\]\]`), có
  unit test; chế độ sửa tay hiển thị chữ thuần (không chip).

### 10. Trạng thái chờ AI và lỗi API

- `ai-progress`: thay nút bằng trạng thái "Đang phân tích outline…" / "Đang viết prompt cho 10 ảnh…", thanh
  tiến trình không xác định + đồng hồ "đã 12 giây"; sau 45 giây thêm dòng "Bài nhiều ảnh có thể mất tới 1
  phút." Vùng có `aria-live="polite"`. Không cho bấm lại khi đang chạy.
- Map lỗi chuẩn (contracts/api.md) → thông báo:

| Mã | Hiển thị |
| --- | --- |
| `VALIDATION` | Inline tại trường + `issues` |
| `UNAUTHENTICATED` | Chuyển tới `/login` |
| `FORBIDDEN` | Banner "Bạn không còn quyền với dự án này hoặc tài khoản đã bị khoá." |
| `NOT_FOUND` | Trang "Không tìm thấy phiên" + link về M1 |
| `CONFIRM_REQUIRED` | `confirm-replace-dialog` |
| `PROVIDER_DAILY_LIMIT` | `Alert` "Hệ thống đã dùng hết lượt AI hôm nay. Bạn vẫn có thể sửa tay và export." |
| `AI_BAD_OUTPUT`, `AI_UPSTREAM` | `Alert` "AI chưa trả kết quả hợp lệ. Dữ liệu của bạn vẫn được giữ." + "Thử lại" |
| `NO_ACTIVE_PROVIDER` | `Alert` "Chưa cấu hình nhà cung cấp AI. Liên hệ Admin." |

- **confirm-replace-dialog** (`AlertDialog`), nội dung lấy số lượng từ response 409:
  - Phân tích lại: tiêu đề "Phân tích lại outline?"; thân "8 chỗ ảnh và 10 prompt hiện có (2 prompt đã
    sửa tay) sẽ bị thay bằng kết quả mới. Cấu hình brand của phiên giữ nguyên."; nút "Phân tích lại" (huỷ
    là mặc định focus).
  - Tạo lại pack: tiêu đề "Tạo lại Prompt Pack?"; thân "2 prompt đã sửa tay (S2-1, S4-2) sẽ bị thay bằng
    prompt mới."; nút "Tạo lại".
  - Xác nhận → gọi lại cùng route với `confirmReplace: true`.

### 11. Khả năng truy cập bàn phím

- Thứ tự Tab theo thứ tự đọc; mọi nút icon có `aria-label` tiếng Việt; focus ring 2px dễ thấy trên cả nền
  sáng/tối.
- Stepper là `nav` với `aria-current="step"`; bước bị khoá có `aria-disabled` + tooltip "Cần Phân tích
  trước".
- Dialog giữ focus bên trong và trả focus về nút mở khi đóng (mặc định của shadcn/Radix).
- Sau khi Phân tích/Tạo prompt xong: focus chuyển tới tiêu đề bước mới; thông báo qua `aria-live`.
- Lỗi trường gắn với input bằng `aria-describedby`; trường lỗi có `aria-invalid`.

### 12. Chữ giao diện chính (tiếng Việt)

Nút: "Bắt đầu bài mới", "Phân tích", "Phân tích lại", "Tạo prompt", "Tạo lại cả pack", "Thêm chỗ ảnh",
"Cấu hình", "Sửa", "Lưu", "Huỷ", "Khôi phục bản gốc", "Copy", "Copy cả pack", "Export", "Thử lại".
Nhãn: "Heading đã nhận", "Lý do AI đề xuất", "Cần ảnh {code} trước", "File cần chuẩn bị", "đã sửa tay",
"Ảnh có chữ — kiểm chữ trước khi dùng", "Không có người", "Gắn sau", "(mặc định)". Tên 9 mục đích hiển
thị: Ảnh đại diện (hero), Minh hoạ bối cảnh (scene), Bước hướng dẫn (howto_step), Sản phẩm trải phẳng
(product_flatlay), Sản phẩm trên người mẫu (product_on_model), Sản phẩm trong đời thực
(product_lifestyle), Bộ sưu tập design (design_gallery), So sánh (comparison), Pin Pinterest
(pinterest_pin). Toàn bộ chữ đặt trong `components/creator/strings.ts` (một nơi, dễ sửa).

## Export (`lib/export/`, SC-010)

- Hàm thuần: `toJson(pack)`, `toCsv(pack)`, `toMarkdown(pack)`, `toClipboard(pack)`; đầu vào là
  `ExportPack = { session, filesToPrepare: RefEntry[], prompts: PackPrompt[] }`, sắp theo `genOrder`.
- Theo đúng [contracts/export.md](../../contracts/export.md): 12 trường cùng thứ tự; CSV RFC 4180, BOM
  `﻿`, CRLF, bọc `"` khi có `,` `"` `\r` `\n` (nhân đôi `"`); tên file `<keyword-slug>-prompt-pack.csv`.
- Unit test: thứ tự cột và header; escape dấu phẩy, ngoặc kép, xuống dòng trong prompt; tiếng Việt có dấu
  trong keyword/cảnh báo; refs/dependsOn/warnings nối đúng ký tự; `manual_edit` yes/no; sắp theo genOrder;
  JSON và Markdown cùng tập trường với CSV.
- Ở L1, W5 gọi formatter trong route export (ghi log `pack.export`); "Copy cả pack" gọi
  `POST /export { format: 'clipboard' }` để có log, rồi ghi `text` vào clipboard. Copy 1 prompt làm ở client.

## Mock API và chuyển sang API thật

- **Bật mock**: `NEXT_PUBLIC_MOCK_API=1` → `api-client.ts` gọi handler trong `mocks/creator-api/` (hàm thuần
  chạy trong trình duyệt, giữ state phiên trong bộ nhớ), không gọi `/api`. Đọc dự án/danh mục cũng lấy từ
  fixture khi mock.
- **Fixtures**: dự án Print Your Wear + 6 danh mục (lấy tên/slug từ `docs/brand.json`), 1 phiên Gift Guides
  với outline 6 H2, 8 slot / 10 prompt (có gallery 3 ảnh, 1 `[[GEN:S1-1]]`, 1 `[[TEXT:headline]]`, 1 prompt
  `isManualEdit`, 1 lỗi `REQUIRED_REF_MISSING`, 1 cảnh báo). Nếu W2 đã merge fixture test thì dùng lại.
- **Mock phải giả lập**: độ trễ AI 2–5 giây, 409 `CONFIRM_REQUIRED` khi có slot/prompt, lỗi
  `AI_BAD_OUTPUT` khi outline chứa chuỗi `#mock-fail`, và `PATCH /api/prompts` kiểm lại độ dài > 2.000.
- **Chuyển sang API thật** (sau khi W5 merge): tắt cờ, chạy quickstart kịch bản 4 bước 1–8, sửa chỗ lệch
  hợp đồng bằng cách báo người điều phối (không sửa route của W5). Đổi types sao chép sang import
  `lib/engine/types.ts` khi W2 merge.

## Thứ tự hiện thực

1. **Nền W6** (sau G0): thêm component shadcn cần dùng, `strings.ts`, `api-client.ts`, mock + fixtures,
   `app/(creator)/layout.tsx`.
2. **US4 export trước phần còn lại** (thuần, độc lập): `lib/export/*` + unit test xanh.
3. **US1**: M1 → M2 → M3 bước 1 (intake + preview + Phân tích + progress) → bước 3 (thẻ prompt, placeholder,
   File cần chuẩn bị, sửa tay/khôi phục, copy, export). Bước 2 tạm hiển thị danh sách slot chỉ đọc + nút Tạo
   prompt.
4. **US3**: bước 2 đầy đủ (slot-card cấu hình, trường sản phẩm, thêm/xoá/nhân đôi, kéo thả + phím tắt,
   autosave, cảnh báo pack cũ).
5. **Hoàn thiện**: dialog 409, map lỗi, a11y (§11), kiểm 360px, đối chiếu chữ §12.
6. **Nối API thật** khi W5 merge.

## Cách kiểm 360px và a11y

- DevTools device toolbar ở 360×740 và 640×900 cho M1, M2, M3 (cả 3 bước, thẻ mở/đóng, dialog, sheet):
  `document.documentElement.scrollWidth <= 360` (chạy trong Console) và không phần tử nào bị cắt.
- Đi toàn luồng chỉ bằng bàn phím (§11), kể cả đổi thứ tự slot bằng Space/↑/↓ và Alt+↑/↓.
- Kiểm tương phản chip placeholder và chữ cảnh báo bằng công cụ contrast của DevTools (≥ 4.5:1).

## Complexity Tracking

Không có vi phạm. Ghi chú: thư mục `mocks/creator-api/` chưa có trong bản đồ sở hữu của plan tổng — cần
người điều phối xác nhận.
