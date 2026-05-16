# Bootstrap Spec: Claude Code Workflow Template Repo

Bạn đang ở trong một repository trống. Nhiệm vụ của bạn là khởi tạo nó thành một **template repository** chuẩn hóa workflow làm việc với Claude Code cho các dự án Java backend (Spring Boot). Repo này sẽ được clone hoặc copy vào project mới mỗi khi bắt đầu một dự án.

**Lưu ý quan trọng**: KHÔNG tạo file `CLAUDE.md` trong repo này. File đó sẽ được khởi tạo riêng tại từng project thật.

---

## Cấu trúc thư mục cần tạo

```
.
├── .claude/
│   ├── commands/
│   │   ├── rsd-to-pttk.md
│   │   ├── pttk-to-plan.md
│   │   └── implement-task.md
│   └── settings.json
├── docs/
│   ├── rsd/
│   │   └── .gitkeep
│   ├── pttk/
│   │   └── .gitkeep
│   └── implementation-plan/
│       └── .gitkeep
├── templates/
│   ├── rsd-template.md
│   ├── pttk-template.md
│   └── implementation-plan-template.md
├── README.md
└── .gitignore
```

---

## Yêu cầu chi tiết từng file

### 1. `.claude/commands/rsd-to-pttk.md`

Slash command để phân tích RSD và sinh ra PTTK (Phân tích Thiết kế).

- Frontmatter YAML có `description` và `argument-hint`
- Quy trình 4 bước:
  1. Đọc RSD tại đường dẫn được truyền vào qua `$ARGUMENTS`
  2. Đặt câu hỏi làm rõ nếu RSD có điểm mơ hồ (dừng lại hỏi user, không tự đoán)
  3. Phân tích tác động lên code hiện có (scan module liên quan, kiểm tra conflict, migration DB)
  4. Sinh file PTTK lưu vào `docs/pttk/<tên-feature>-pttk.md`
- Cấu trúc PTTK output bao gồm các section:
  - Tổng quan (mục đích, phạm vi in-scope/out-of-scope, link RSD)
  - Phân tích nghiệp vụ (actors, use case chính + luồng phụ, business rules đánh số BR-001, BR-002...)
  - Thiết kế kỹ thuật (API Endpoints dạng bảng, Data Model với mermaid ERD, Service Layer với method signatures, Integration Points)
  - Non-functional Requirements (performance, security, logging, monitoring)
  - Tác động và rủi ro (module bị ảnh hưởng, breaking changes, rollback strategy)
  - Test Strategy (unit test scope, integration test scenarios, edge cases)
- Nhấn mạnh ở cuối file: **KHÔNG viết code ở bước này, chỉ thiết kế**. Sử dụng mermaid cho diagram. Mỗi business rule phải mapping được với requirement trong RSD.

### 2. `.claude/commands/pttk-to-plan.md`

Slash command để sinh implementation plan từ file PTTK.

- Frontmatter YAML có `description` và `argument-hint`
- Quy trình:
  1. Đọc file PTTK tại `$ARGUMENTS` và CLAUDE.md của project
  2. Scan codebase tìm pattern hiện có (controller mẫu, service mẫu, test mẫu)
  3. Chia nhỏ thành các task ~30-60 phút mỗi task
  4. Lưu plan vào `docs/implementation-plan/<feature>-plan.md`
- Cấu trúc plan output:
  - Metadata (PTTK reference, estimated effort, created date, status)
  - Progress Summary (total tasks, completed count, last updated)
  - Tasks: mỗi task có ID (TASK-001), Status (⬜🟡✅❌), Type (Database/Backend/Test/Config), Files to create/modify, Description, Acceptance criteria (checklist), Dependencies, Test requirements, Notes
  - Implementation Order (thứ tự thực hiện, có thể parallel)
  - Risks & Decisions Log (sẽ được update khi implement)
- Nguyên tắc chia task (ghi rõ ở cuối file):
  - Mỗi task tự đứng được, có thể commit độc lập
  - Database trước → entity → service → controller
  - Test đi kèm với code, không tách task test riêng
  - Task đầu tiên thường là setup (migration, base entity)

### 3. `.claude/commands/implement-task.md`

Slash command để implement một task cụ thể từ plan.

- Frontmatter YAML có `description` và `argument-hint` với format `<feature-name> <task-id>` (ví dụ: `order-management TASK-003`)
- Quy trình 6 bước:
  1. **Load context**: Đọc `docs/implementation-plan/<feature-name>-plan.md`, tìm task có ID khớp, đọc phần liên quan trong `docs/pttk/<feature-name>-pttk.md`, kiểm tra dependencies đã done chưa. Nếu chưa, DỪNG và báo cáo.
  2. **Verify task status**: Nếu task đã ✅ Done thì hỏi user có muốn redo không. Update status thành 🟡 In Progress trong plan file.
  3. **Implement**: Tuân thủ acceptance criteria, viết code theo convention trong CLAUDE.md, viết test cùng lúc với code, scan các file pattern tương tự trong codebase để follow style.
  4. **Verify**: Chạy test (tìm command test trong CLAUDE.md hoặc pom.xml/build.gradle), đảm bảo build pass. Nếu lỗi thì fix, không bỏ qua.
  5. **Update plan file**: Status task → ✅ Done, tick checklist acceptance criteria, thêm Notes (files đã tạo/sửa, decisions trong khi code), update Progress Summary, nếu phát hiện vấn đề ảnh hưởng task khác thì thêm vào Risks & Decisions Log.
  6. **Báo cáo**: Tóm tắt files đã thay đổi, test pass/fail, gợi ý task tiếp theo theo Implementation Order.
- Nhấn mạnh:
  - KHÔNG implement nhiều task cùng lúc
  - KHÔNG sửa file ngoài "Files to create/modify" mà không hỏi
  - Nếu phát hiện plan có lỗi thì DỪNG, báo cáo, không tự ý thay đổi

### 4. `.claude/settings.json`

File JSON với cấu hình permissions cơ bản:
- Cho phép Claude đọc/ghi trong `docs/` và `src/`
- Từ chối các thao tác nguy hiểm: `rm -rf`, sửa `.git/`, sửa `.claude/settings.json` chính nó
- Nếu không chắc cú pháp chuẩn của settings.json, để file dạng `{}` rỗng và thêm comment ngoài hướng dẫn user customize sau

### 5. `templates/rsd-template.md`

Template mẫu cho file RSD (Requirement Specification Document). Cấu trúc gợi ý:
- Thông tin chung (tên feature, owner, version, ngày tạo)
- Mục đích nghiệp vụ
- Phạm vi
- Yêu cầu chức năng (đánh số FR-001, FR-002...)
- Yêu cầu phi chức năng
- Acceptance criteria
- Câu hỏi mở / Giả định

Để các section dạng placeholder cho user điền.

### 6. `templates/pttk-template.md`

Template mẫu cho PTTK, cấu trúc khớp 100% với output mà slash command `/rsd-to-pttk` sinh ra (xem mục 1).

### 7. `templates/implementation-plan-template.md`

Template mẫu cho implementation plan, cấu trúc khớp với output của `/pttk-to-plan` (xem mục 2).

### 8. `.gitignore`

Phù hợp cho Java/Maven/Gradle project + IDE common files:
- `target/`, `build/`, `out/`
- `.idea/`, `.vscode/`, `*.iml`
- `.DS_Store`, `Thumbs.db`
- `*.log`, `*.tmp`
- `.env`, `.env.local`

**Quan trọng**: KHÔNG ignore thư mục `.claude/` vì đây là phần workflow cần share trong team.

### 9. `README.md`

Hướng dẫn sử dụng template repo. Bao gồm:

- **Mục đích**: Repo này là gì, giải quyết vấn đề gì
- **Cách bootstrap vào project mới**: hướng dẫn 2 cách
  - Clone repo này rồi đổi remote
  - Copy các thư mục `.claude/`, `docs/`, `templates/` vào project hiện có
- **Bước đầu tiên tại project mới**: tạo file `CLAUDE.md` ở root project (vì template không kèm sẵn), gợi ý dùng lệnh `/init` của Claude Code để auto-generate rồi customize
- **Workflow tổng quan**:
  ```
  RSD file → /rsd-to-pttk → PTTK file
           → /pttk-to-plan → Implementation Plan
           → /implement-task → Code + Updated Plan
  ```
- **Cấu trúc thư mục**: giải thích từng phần
- **Tips sử dụng**:
  - Plan mode (Shift+Tab) cho task lớn
  - `/clear` giữa các task để reset context
  - Reference file bằng `@<path>`
  - Commit thường xuyên sau mỗi task done
  - Review PTTK kỹ trước khi sang bước plan (rẻ hơn sửa code sau)
- **Mở rộng**: gợi ý các slash command có thể thêm sau (`/code-review`, `/debug-issue`, `/write-tests`)

### 10. `.gitkeep` trong các thư mục `docs/rsd/`, `docs/pttk/`, `docs/implementation-plan/`

File rỗng để git track được thư mục rỗng.

---

## Quy tắc thực thi

1. **Xác nhận kế hoạch trước**: Trước khi tạo file, liệt kê thứ tự bạn sẽ thực hiện để tôi kịp dừng nếu thấy sai.
2. **Tạo lần lượt**: Không tạo song song nhiều file phức tạp một lúc.
3. **Sau khi hoàn tất**: Chạy `tree -a -I '.git'` (hoặc `find . -not -path './.git*' | sort` nếu không có `tree`) để verify cấu trúc cuối cùng.
4. **KHÔNG khởi tạo git commit**: Tôi sẽ tự review rồi commit.
5. **KHÔNG tự đoán khi gặp ambiguity**: Nếu có quyết định thiết kế nào bạn không chắc, liệt kê ra cho tôi quyết định TRƯỚC khi tạo file.
6. **KHÔNG tạo CLAUDE.md**: Như đã nói ở đầu, file này thuộc về từng project riêng, không phải template repo.

Bắt đầu bằng việc xác nhận lại kế hoạch tạo file (liệt kê thứ tự bạn sẽ làm), sau đó thực hiện.
