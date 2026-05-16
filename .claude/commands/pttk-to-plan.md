---
description: Sinh implementation plan (danh sách task ~30-60 phút) từ file PTTK
argument-hint: <đường-dẫn-tới-file-pttk>
---

# /pttk-to-plan — Sinh Implementation Plan từ PTTK

Bạn là Tech Lead chịu trách nhiệm chuyển PTTK thành kế hoạch implement chi tiết. Nhiệm vụ: đọc file PTTK tại `$ARGUMENTS`, kết hợp với convention trong CLAUDE.md, scan codebase, rồi sinh ra file plan có thể được implement từng task một.

## Quy trình

### Bước 1 — Load context

1. Đọc file PTTK tại `$ARGUMENTS`. Nếu không tồn tại, DỪNG và báo user.
2. Đọc `CLAUDE.md` ở root project (nếu có) để nắm convention: framework version, style, test command, package structure.
3. Tóm tắt scope sẽ implement bằng 3-5 gạch đầu dòng.

### Bước 2 — Scan codebase tìm pattern

- Tìm controller mẫu (ví dụ: `*Controller.java` đơn giản nhất) để follow style.
- Tìm service + test mẫu để biết cách viết unit / integration test.
- Tìm cách tổ chức entity / repository / migration hiện có.
- Ghi lại các pattern này — sẽ reference trong task notes.

### Bước 3 — Chia nhỏ thành task

Mỗi task phải đạt các tiêu chí:

- **Kích thước**: ~30-60 phút làm việc (đủ nhỏ để commit độc lập, đủ lớn để không vụn vặt).
- **Tự đứng được**: làm xong là project vẫn build/test pass.
- **Có acceptance criteria rõ ràng**: tick được hoặc không.
- **Test đi kèm code**, KHÔNG tách task test riêng (trừ khi là integration test cuối cùng).

Thứ tự gợi ý:
1. Migration DB / setup
2. Entity + Repository
3. Service (interface → implementation → unit test)
4. Controller (DTO → mapper → endpoint → controller test)
5. Integration test end-to-end
6. Tài liệu / config / feature flag

### Bước 4 — Sinh file plan

Lưu vào `docs/implementation-plan/<feature>-plan.md`. Cấu trúc bắt buộc:

````markdown
# Implementation Plan: <Tên Feature>

## Metadata
- **PTTK reference**: [docs/pttk/<feature>-pttk.md](../pttk/<feature>-pttk.md)
- **Estimated effort**: <tổng giờ ước tính>
- **Created date**: <YYYY-MM-DD>
- **Status**: 🟡 In Progress | ✅ Completed | ⏸ Paused

## Progress Summary
- **Total tasks**: N
- **Completed**: 0 / N
- **Last updated**: <YYYY-MM-DD>

## Legend
- ⬜ Not started
- 🟡 In Progress
- ✅ Done
- ❌ Blocked / Failed

---

## Tasks

### TASK-001: <Tên task ngắn gọn>
- **Status**: ⬜
- **Type**: Database | Backend | Test | Config
- **Files to create/modify**:
  - `src/main/resources/db/migration/V20260101__create_orders.sql`
- **Description**: <2-3 câu mô tả task làm gì>
- **Acceptance criteria**:
  - [ ] Migration chạy thành công trên DB local
  - [ ] Có script rollback (`U20260101__...sql`)
  - [ ] Index trên cột `code` được tạo
- **Dependencies**: (không có / TASK-XXX)
- **Test requirements**:
  - Smoke test: chạy `mvn flyway:migrate` không lỗi
- **Notes**: (sẽ được điền khi implement)

### TASK-002: ...

---

## Implementation Order

```
TASK-001 (DB migration)
    ↓
TASK-002 (Entity + Repository) ─┐
                                ├─ có thể parallel
TASK-003 (DTO + Mapper) ────────┘
    ↓
TASK-004 (Service)
    ↓
TASK-005 (Controller + Controller test)
    ↓
TASK-006 (Integration test)
```

## Risks & Decisions Log

> Sẽ được update khi implement. Mỗi entry: ngày, người (hoặc Claude), quyết định, lý do.

| Date | Task | Decision / Risk | Resolution |
|------|------|------------------|------------|
| | | | |
````

## Nguyên tắc chia task (BẮT BUỘC)

- **Mỗi task tự đứng được, có thể commit độc lập** — không để code dở dang giữa task.
- **Database trước → Entity → Service → Controller** — theo thứ tự dependency tự nhiên.
- **Test đi kèm với code, không tách task test riêng** — task "viết service" phải bao gồm unit test cho service đó.
- **Task đầu tiên thường là setup** (migration DB, base entity, config feature flag).
- Nếu một module logic > 60 phút, chia tiếp thành 2-3 task con (ví dụ: service phức tạp tách thành "interface + happy path" và "error handling + edge case").

## Output cuối

Sau khi sinh file plan, báo cáo:
- Đường dẫn file plan
- Tổng số task, ước tính tổng effort
- Task nào có thể parallel
- Gợi ý bước tiếp: `/implement-task <feature-name> TASK-001`
