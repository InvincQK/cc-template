---
description: Implement một task cụ thể từ implementation plan và cập nhật plan
argument-hint: <feature-name> <task-id>
---

# /implement-task — Implement một task từ Plan

Bạn là Software Engineer đang implement một task cụ thể trong implementation plan. Arguments: `$ARGUMENTS` có dạng `<feature-name> <task-id>` (ví dụ: `order-management TASK-003`).

**Parse arguments**: tách `$ARGUMENTS` thành 2 phần:
- `<feature-name>` — dùng để xác định plan file và pttk file
- `<task-id>` — ID của task cần implement (ví dụ `TASK-003`)

Nếu không parse được (thiếu argument, sai format), DỪNG và yêu cầu user cung cấp lại theo format đúng.

## Quy trình 6 bước

### Bước 1 — Load context

1. Đọc `docs/implementation-plan/<feature-name>-plan.md`. Nếu không tồn tại, DỪNG.
2. Tìm task có ID khớp `<task-id>`. Nếu không có, liệt kê các task ID có sẵn và DỪNG.
3. Đọc phần liên quan trong `docs/pttk/<feature-name>-pttk.md` (đặc biệt: business rules / API spec / data model tương ứng task này).
4. Kiểm tra **Dependencies** của task:
   - Mỗi task dependency phải có status ✅ Done.
   - Nếu chưa, DỪNG và báo cáo: "Task X phụ thuộc TASK-Y nhưng TASK-Y đang ở trạng thái ⬜/🟡. Vui lòng hoàn thành TASK-Y trước."

### Bước 2 — Verify task status

- Nếu task đang ⬜ → tiếp tục.
- Nếu task đang 🟡 → hỏi user: "Task này đang In Progress, có phải bạn muốn tiếp tục dở dang không?"
- Nếu task đã ✅ Done → hỏi user: "Task đã Done. Bạn có chắc muốn redo không? (sẽ ghi đè code và reset checklist)". Nếu không, DỪNG.
- Sau khi user xác nhận tiếp tục, **update status task thành 🟡 In Progress trong plan file** và update `Last updated` ở Progress Summary.

### Bước 3 — Implement

- Đọc các file pattern tương tự trong codebase trước khi viết (controller mẫu, service mẫu, test mẫu) → follow style.
- Tuân thủ **acceptance criteria** trong task — không thêm/bớt scope.
- Viết code theo convention trong `CLAUDE.md` (naming, package structure, lombok / non-lombok, validation lib...).
- **Viết test cùng lúc với code**, không tách thành 2 bước.
- Nếu phát hiện cần file ngoài "Files to create/modify", **DỪNG và hỏi user** trước khi thêm.

### Bước 4 — Verify

1. Tìm command test trong `CLAUDE.md`. Nếu không có, infer từ `pom.xml` (Maven: `mvn test`) hoặc `build.gradle` (Gradle: `./gradlew test`).
2. Chạy test liên quan task (nếu xác định được class cụ thể) hoặc full test suite.
3. Nếu lỗi → fix, không bỏ qua. Nếu lỗi do test cũ không liên quan task này, ghi vào Notes và hỏi user trước khi sửa.
4. Đảm bảo build pass.

### Bước 5 — Update plan file

Edit `docs/implementation-plan/<feature-name>-plan.md`:

- Status task: 🟡 → ✅ Done
- Tick từng acceptance criteria đã hoàn thành: `- [ ]` → `- [x]`
- Phần **Notes** điền:
  - Files thực tế đã tạo/sửa (đường dẫn đầy đủ)
  - Quyết định trong khi code (vì sao chọn approach A thay vì B)
  - Lưu ý cho task sau (gotcha, side effect)
- Update **Progress Summary**: `Completed: X / N` và `Last updated`
- Nếu phát hiện vấn đề ảnh hưởng task khác (ví dụ: schema cần đổi, hoặc dependency bị thiếu), thêm vào **Risks & Decisions Log** với date hôm nay.

### Bước 6 — Báo cáo

Trả về cho user format ngắn gọn:

```
✅ TASK-XXX done

Files changed:
- path/to/file1
- path/to/file2

Tests: <X passed, Y failed>

Decisions / Notes:
- <điểm đáng chú ý>

Next task (theo Implementation Order): TASK-YYY
```

## Nguyên tắc bắt buộc

- **KHÔNG implement nhiều task cùng lúc.** Một lần chạy = một task.
- **KHÔNG sửa file ngoài "Files to create/modify"** mà không hỏi user trước.
- **Nếu phát hiện plan có lỗi** (ví dụ: acceptance criteria không khả thi, dependency thiếu, design conflict) → **DỪNG, báo cáo, không tự ý thay đổi**. Đề xuất sửa plan trước rồi mới implement lại.
- **KHÔNG bỏ qua test fail** dù trông giống flaky. Investigate trước, ghi notes nếu thực sự không liên quan.
- **Mỗi task = một commit logic.** Sau khi báo cáo done, nhắc user commit nếu họ chưa làm: `git add -A && git commit -m "<feature>: TASK-XXX <short desc>"`.
