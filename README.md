# cc-template — Claude Code Workflow Template (Java Spring Boot)

Repo template chuẩn hoá workflow làm việc với **Claude Code** cho các dự án Java backend (Spring Boot). Mục tiêu: clone vào project mới và có ngay một quy trình **RSD → PTTK → Plan → Code** rõ ràng, lặp lại được.

---

## 1. Mục đích

Khi làm việc với Claude Code mà không có quy trình, ta thường gặp 3 vấn đề:

1. **Yêu cầu mơ hồ** → Claude code ra thứ không khớp ý.
2. **Implement quá lớn trong một lượt** → khó review, khó debug, dễ context rotted.
3. **Không trace được** từ requirement → design → task → code khi cần audit hoặc onboard người mới.

Template này giải quyết bằng 5 slash command + 3 loại tài liệu chuẩn:

| Tài liệu | Viết bởi | Phục vụ |
|----------|----------|---------|
| **RSD** (Requirement Spec) | BA/PO viết Word → `/sync-rsd-template` convert | Mô tả "làm gì" nghiệp vụ |
| **PTTK** (Phân tích Thiết kế) | `/rsd-to-pttk` | Mô tả "làm như thế nào" về kỹ thuật |
| **Implementation Plan** | `/pttk-to-plan` | Chia thành task ~30-60' để code lần lượt |
| Code thật | `/implement-task` | Implement + test + update plan |

Command hỗ trợ convert template từ Word:

| Command | Input | Output |
|---------|-------|--------|
| `/sync-rsd-template` | Word RSD (`.claude/output-templates/source/rsd.docx`) | `docs/rsd/<feature>-rsd.md` |
| `/sync-pttk-template` | Word PTTK (`.claude/output-templates/source/pttk.docx`) | `.claude/output-templates/pttk.md` (active template) |

---

## 2. Bootstrap vào project mới

### Cách 1 — Clone repo này và đổi remote

```bash
git clone <url-repo-template> my-new-project
cd my-new-project
rm -rf .git
git init
git remote add origin <url-repo-thật>
```

### Cách 2 — Copy vào project hiện có

```bash
# Từ trong project hiện có
cp -r /path/to/cc-template/.claude ./
cp -r /path/to/cc-template/docs ./
cp -r /path/to/cc-template/templates ./
cp /path/to/cc-template/.gitignore ./.gitignore   # hoặc merge tay nếu đã có
```

> **Lưu ý**: Repo template KHÔNG kèm `CLAUDE.md`. Mỗi project có context riêng nên file đó phải khởi tạo tại project thật (xem mục tiếp theo).

---

## 3. Bước đầu tiên tại project mới

Sau khi bootstrap, làm 2 việc ngay:

1. **Tạo `CLAUDE.md` ở root project** — đây là file context Claude Code sẽ đọc mỗi conversation.
   - Cách nhanh nhất: chạy lệnh `/init` của Claude Code → nó sẽ scan codebase và sinh draft, sau đó customize lại cho khớp convention thực tế của team (test command, package structure, framework version, naming...).
2. **Review `.claude/settings.json`** — chỉnh permissions cho phù hợp project (ví dụ: thêm `Bash(docker compose:*)` nếu dùng docker).

---

## 4. Workflow tổng quan

**Path A — Từ Word RSD (khuyến khích):**
```
.docx (Word)          rsd.md              pttk.md             plan.md
    │                    │                   │                   │
    ▼ /sync-rsd-template ▼ /rsd-to-pttk     ▼ /pttk-to-plan    ▼ /implement-task
.docx ──────────► .md ──────────► .md ──────────► .md ──────────► Code+
```

**Path B — Viết tay RSD (fallback):**
```
Write manually ──► docs/rsd/<feature>-rsd.md ──► ... (từ /rsd-to-pttk trở đi)
```

Ví dụ thực tế cho feature "quản lý đơn hàng" dùng Word:

```text
1. BA viết Word:        .claude/output-templates/source/rsd.docx
2. Chạy:                /sync-rsd-template order-management
                        → sinh docs/rsd/order-management-rsd.md
3. Chạy:                /rsd-to-pttk docs/rsd/order-management-rsd.md
                        → sinh docs/pttk/order-management-pttk.md
4. Chạy:                /pttk-to-plan docs/pttk/order-management-pttk.md
                        → sinh docs/implementation-plan/order-management-plan.md
5. Chạy:                /implement-task order-management TASK-001
                        → code + test + update plan
6. Lặp lại:             /implement-task order-management TASK-002, TASK-003, ...
```

---

## 5. Cấu trúc thư mục

```
.
├── .claude/
│   ├── commands/               # 5 slash command chính
│   │   ├── sync-rsd-template.md
│   │   ├── rsd-to-pttk.md
│   │   ├── pttk-to-plan.md
│   │   ├── implement-task.md
│   │   └── sync-pttk-template.md
│   ├── output-templates/       # ACTIVE template: command đọc & gen theo
│   │   ├── source/             # Raw files (Word, v.v.)
│   │   │   ├── rsd.docx        # BA đặt Word RSD vào đây
│   │   │   └── pttk.docx       # User đặt Word PTTK vào đây
│   │   ├── pttk.md             # Auto-gen từ pttk.docx, /rsd-to-pttk đọc
│   │   └── ...
│   └── settings.json           # Permissions: allow / deny / ask
│
├── docs/                       # Output của workflow, commit chung với code
│   ├── rsd/                    # Requirement specs (con người viết)
│   ├── pttk/                   # Phân tích thiết kế (Claude sinh, người review)
│   └── implementation-plan/    # Task lists (Claude sinh, Claude cập nhật khi code)
│
├── templates/                  # Starter cho file viết tay (không có command gen)
│   └── rsd-template.md         # Copy-paste khi viết RSD mới
│
├── README.md                   # File này
└── .gitignore
```

Giải thích từng thư mục:

- **`.claude/commands/`**: 5 slash command. Commit vào repo để cả team dùng chung.
  - `sync-rsd-template.md`: convert Word RSD → markdown RSD
  - `rsd-to-pttk.md`: phân tích RSD → PTTK
  - `pttk-to-plan.md`: từ PTTK → plan tasks
  - `implement-task.md`: implement từng task
  - `sync-pttk-template.md`: convert Word PTTK → markdown PTTK
- **`.claude/output-templates/`**: **ACTIVE template** + Raw files — slash command đọc khi sinh output. Customize per-project.
  - `pttk.md`: cấu trúc output của `/rsd-to-pttk`. Sửa trực tiếp HOẶC sync từ Word.
  - `source/`: folder chứa raw files (Word) từ user. Slash command đọc từ đây rồi convert sang markdown.
    - `source/rsd.docx`: BA đặt Word RSD vào đây, `/sync-rsd-template` convert → `docs/rsd/<feature>-rsd.md`
    - `source/pttk.docx`: user đặt Word PTTK vào đây, `/sync-pttk-template` convert → `pttk.md` (active template)
- **`docs/`**: tài liệu sinh ra trong workflow (RSD, PTTK, plan), commit cùng code.
  - `docs/rsd/`: RSD files (`<feature>-rsd.md`) — từ `/sync-rsd-template` hoặc viết tay
  - `docs/pttk/`: PTTK files (`<feature>-pttk.md`) — sinh từ `/rsd-to-pttk`
  - `docs/implementation-plan/`: Plan files (`<feature>-plan.md`) — sinh từ `/pttk-to-plan`
- **`templates/`**: starter cho file viết tay. Chỉ chứa `rsd-template.md` — guide để BA viết RSD tay nếu không dùng Word.
- **`.claude/settings.json`**: permissions allow/deny/ask. Phần cá nhân để ở `.claude/settings.local.json` (đã ignore mặc định).

### 5.1 RSD workflow — Tay viết vs. Word convert

BA có 2 cách để tạo RSD:

**Cách A — Viết tay markdown** (nhanh, không cần Word):

1. Copy file `templates/rsd-template.md` thành `docs/rsd/<feature>-rsd.md`
2. BA chỉnh sửa theo requirement thực tế
3. Commit vào repo
4. Dev chạy `/rsd-to-pttk docs/rsd/<feature>-rsd.md`

**Cách B — Viết Word, convert markdown** (team có template Word sẵn):

```
.claude/output-templates/source/rsd.docx   ← BA sửa (Word)
              ↓ /sync-rsd-template order-management
docs/rsd/order-management-rsd.md           ← Auto-gen, chuẩn hóa
```

1. BA viết Word RSD theo template team
2. Đặt file Word vào `.claude/output-templates/source/rsd.docx`
3. Dev chạy `/sync-rsd-template <tên-feature>` để convert → `docs/rsd/<feature>-rsd.md`
4. Dev chạy `/rsd-to-pttk docs/rsd/<feature>-rsd.md`

**Lưu ý**: 
- Nếu dùng **Cách B**, KHÔNG sửa `docs/rsd/<feature>-rsd.md` thủ công — sẽ bị ghi đè lần sync sau. Sửa Word rồi chạy lại `/sync-rsd-template`.
- Nếu dùng **Cách A**, có thể sửa `docs/rsd/<feature>-rsd.md` tự do (không có automation).

### 5.2 Customize active template PTTK — 2 cách

**Cách A — Sửa markdown PTTK template trực tiếp** (dev quen markdown):

1. Mở `.claude/output-templates/pttk.md`
2. Thêm/bớt section, đổi cột bảng, đổi loại diagram theo team
3. Commit → từ lần chạy `/rsd-to-pttk` tiếp theo, mọi PTTK theo cấu trúc mới
4. Nếu xoá file → command rơi về cấu trúc fallback mặc định

**Cách B — Sửa Word, sync ra markdown** (khi team có template chuẩn bằng .docx):

```
.claude/output-templates/source/pttk.docx   ← Sửa file Word ở đây
              ↓ /sync-pttk-template
.claude/output-templates/pttk.md            ← Auto-gen, /rsd-to-pttk dùng
```

1. Đặt file Word vào `.claude/output-templates/source/pttk.docx`
2. Chạy `/sync-pttk-template` (cần [pandoc](https://pandoc.org/installing.html) cài sẵn)
3. Command convert .docx → markdown, refine thành active template, ghi đè `pttk.md`
4. KHÔNG sửa `pttk.md` trực tiếp khi dùng cách B — sẽ bị ghi đè lần sync sau

Ví dụ customize theo team:
- Team fintech: thêm section "Threat Model" và "Compliance Mapping"
- Team API-heavy: thêm cột "Rate limit", "Auth required" vào bảng API

---

## 6. Tips sử dụng

### 6.1 Tận dụng Plan mode

Với task lớn (ví dụ implement nguyên một service phức tạp), bật **Plan mode** bằng `Shift+Tab` để Claude lập kế hoạch trước khi gõ code — bạn có cơ hội duyệt approach trước khi nó write file.

### 6.2 `/clear` giữa các task

Mỗi task implement nên reset context bằng `/clear` để tránh Claude bị "ám" bởi quyết định của task trước (đặc biệt khi task trước có nhiều thử-sai). Slash command `/implement-task` đã được thiết kế để self-contained — chỉ cần đọc plan + pttk là đủ context.

### 6.3 Reference file bằng `@<path>`

Trong prompt, gõ `@docs/pttk/order-management-pttk.md` để Claude đọc thẳng file đó vào context. Hữu ích khi muốn hỏi câu hỏi liên quan tới 1 tài liệu cụ thể mà không qua slash command.

### 6.4 Commit thường xuyên

Quy ước: mỗi task done → một commit. Format gợi ý:

```
<feature>: TASK-XXX <ngắn gọn việc làm>

Refs: docs/pttk/<feature>-pttk.md
```

Lợi ích: dễ revert một task riêng lẻ mà không ảnh hưởng task khác.

### 6.5 Review PTTK kỹ TRƯỚC khi sang bước plan

Sửa PTTK rẻ hơn sửa code rất nhiều. Đặc biệt review:
- Business rule có đầy đủ không, có trace ngược về FR trong RSD không?
- Data model có miss bảng/cột nào không?
- API endpoint có thiếu status code, validation rule không?

---

## 7. Mở rộng

Khi quy trình ổn định, có thể thêm các slash command sau (chưa bao gồm trong template để tránh phình to):

- `/code-review` — review diff hiện tại, focus vào convention và security.
- `/debug-issue` — đọc stack trace, log, đề xuất root cause.
- `/write-tests` — sinh thêm test cho code đã có nhưng coverage thấp.
- `/refactor-suggest` — phát hiện smell (long method, god class, duplicate logic).
- `/db-migration-review` — kiểm tra migration có reversible không, có lock bảng lớn không.

Mỗi command đặt thêm một file `.md` vào `.claude/commands/` với cùng format frontmatter.

---

## 8. Đóng góp

Nếu chỉnh sửa template repo:
- Slash command đổi → cập nhật template tương ứng để cấu trúc khớp.
- Đổi cấu trúc thư mục `docs/` → cập nhật README + tất cả slash command tham chiếu đến nó.
