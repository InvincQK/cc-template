---
description: Import file RSD .md từ user, rà soát đầy đủ thông tin và copy vào docs/rsd/
argument-hint: <đường-dẫn-tới-file-md> [tên-feature]
---

# /import-rsd — Rà soát & import RSD markdown vào workflow

Workflow: User đã có file RSD viết tay (`.md`) — có thể export từ Notion, Obsidian, hoặc viết thẳng. Command này **rà soát đầy đủ thông tin theo yêu cầu workflow**, đánh dấu TODO chỗ thiếu, rồi copy vào `docs/rsd/<feature>-rsd.md` để dùng tiếp.

```
<path>/<file>.md                    ← User viết tay
        ↓ /import-rsd <path> [feature]
docs/rsd/<feature>-rsd.md           ← Đã validate + TODO markers (nếu thiếu)
        ↓ /rsd-to-pttk
docs/pttk/<feature>-pttk.md
```

**Khác với `/sync-rsd-template`**: command này nhận `.md` (đã sẵn), không cần pandoc/refine artifact. Chỉ validate + copy.

## Quy trình 5 bước

### Bước 1 — Xác định input path

- `$ARGUMENTS` argument đầu tiên = path tới file `.md`. BẮT BUỘC.
- Nếu rỗng → DỪNG, hỏi user: "Đường dẫn tới file RSD .md là gì?"
- Kiểm tra file tồn tại và là `.md`. Nếu không → DỪNG, báo lỗi cụ thể.

### Bước 2 — Xác định feature name

- `$ARGUMENTS` argument thứ 2 = feature name (kebab-case).
- Nếu không có → thử detect từ:
  1. Heading `# RSD: <Tên Feature>` trong file (nếu có).
  2. Tên file (vd `employee-onboarding.md` → `employee-onboarding`).
- Nếu vẫn không xác định được → **hỏi user**: "Tên feature là gì? (kebab-case, vd: employee-onboarding)"
- Output path: `docs/rsd/<feature-name>-rsd.md`

### Bước 3 — Rà soát thông tin (CỐT LÕI)

Reference `templates/rsd-template.md` để biết structure chuẩn. Đọc file input và check từng mục:

**3.1. Thông tin chung**
- [ ] Tên feature rõ ràng
- [ ] Mục đích (Why — lý do cần feature này)
- [ ] Scope: in-scope, out-of-scope
- [ ] Stakeholder / Actor liên quan

**3.2. Yêu cầu chức năng (FR-xxx)**
- [ ] Có ít nhất 1 FR
- [ ] Mỗi FR có ID dạng `FR-XXX`
- [ ] Mỗi FR có **mô tả cụ thể** (không mơ hồ kiểu "implement something")
- [ ] Mỗi FR có **Acceptance Criteria** rõ ràng
- [ ] Mỗi FR có actor / role thực hiện

**3.3. Yêu cầu phi chức năng (NFR)**
- [ ] Performance (response time, throughput) — nếu áp dụng
- [ ] Security (auth, authorization, data sensitivity)
- [ ] Logging / Monitoring

**3.4. Edge cases & Validation**
- [ ] Đã liệt kê edge case (dữ liệu rỗng, vượt giới hạn, trùng lặp)
- [ ] Đã định nghĩa validation rule (nếu có form input)

### Bước 4 — Build output file

Đọc nội dung file input. Build output theo flow:

1. **Header**: Thêm khối comment ở đầu file:

   ```html
   <!--
   =========================================================================
   RSD — Imported by /import-rsd
   =========================================================================
   Source: <path tới file gốc>
   Imported on: <YYYY-MM-DD HH:mm>

   File này được copy từ input markdown. Đã rà soát theo yêu cầu workflow.
   Các section còn TODO marker → user cần bổ sung trước khi chạy /rsd-to-pttk.
   =========================================================================
   -->
   ```

2. **Normalize**: Đảm bảo dòng `# RSD: <Tên Feature>` ở đầu (sau header comment). Normalize heading level.

3. **Giữ NGUYÊN nội dung từ input** — KHÔNG paraphrase, KHÔNG dịch, KHÔNG tự điền.

4. **TODO markers**: Với mỗi mục check fail ở Bước 3, chèn comment ở vị trí phù hợp:

   ```html
   <!-- TODO: [missing] <mô tả cụ thể cái còn thiếu>
        Ví dụ: "Acceptance Criteria cho FR-002 chưa được định nghĩa.
                BA cần bổ sung tiêu chí pass/fail rõ ràng."
   -->
   ```

   Vị trí chèn:
   - Section thiếu hoàn toàn → chèn TODO ở cuối file với heading `## TODO (cần BA bổ sung)`.
   - Section có nhưng nội dung thiếu (vd FR thiếu AC) → chèn TODO ngay trong section đó.

5. **KHÔNG tự sinh nội dung** điền vào chỗ thiếu. Tôn trọng rule: BA là người cung cấp requirement.

### Bước 5 — Ghi file + Báo cáo

1. Tạo thư mục `docs/rsd/` nếu chưa có.
2. Nếu `docs/rsd/<feature>-rsd.md` đã tồn tại → hỏi user confirm overwrite hay đặt tên khác.
3. Ghi nội dung đã build vào `docs/rsd/<feature>-rsd.md`.

Format báo cáo:

```
✅ Import xong

Input:   <path tới file .md gốc>
Output:  docs/rsd/<feature-name>-rsd.md

Validation result:
✓ Sections đầy đủ (N/N):
  - Thông tin chung
  - Yêu cầu chức năng (X FRs)
  - ...

⚠ Sections thiếu / chưa đủ (M):
  - [Acceptance Criteria for FR-002] → đã chèn TODO marker
  - [NFR - Performance] → đã chèn TODO marker
  - ...

Next:
  → Nếu có TODO: BA bổ sung trực tiếp vào docs/rsd/<feature-name>-rsd.md
    rồi chạy lại /import-rsd để re-validate.
  → Nếu không TODO: chạy /rsd-to-pttk docs/rsd/<feature-name>-rsd.md
```

## Nguyên tắc bắt buộc

- **KHÔNG tự điền** thông tin thiếu. Chỉ chèn TODO marker để BA biết cần bổ sung gì.
- **KHÔNG paraphrase / refactor** nội dung user viết. Giữ nguyên ngôn ngữ và cấu trúc.
- **Idempotent**: chạy lại nhiều lần phải cho cùng kết quả. Nếu user đã bổ sung TODO → lần chạy sau TODO đó biến mất.
- **TODO marker phải SPECIFIC**: Không ghi chung chung "thiếu thông tin", phải nói rõ thiếu cái gì (vd "FR-003 thiếu Acceptance Criteria").
- **Không validate quá strict**: NFR section có thể optional với một số feature đơn giản — chỉ flag TODO khi rõ ràng cần (vd feature có form input thì cần validation rule).
