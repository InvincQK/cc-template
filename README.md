# cc-template — Claude Code Workflow Template (Java Spring Boot)

Repo template chuẩn hoá workflow làm việc với **Claude Code** cho các dự án Java backend (Spring Boot). Mục tiêu: clone vào project mới và có ngay một quy trình **RSD → PTTK → Plan → Code** rõ ràng, lặp lại được.

---

## 1. Mục đích

Khi làm việc với Claude Code mà không có quy trình, ta thường gặp 3 vấn đề:

1. **Yêu cầu mơ hồ** → Claude code ra thứ không khớp ý.
2. **Implement quá lớn trong một lượt** → khó review, khó debug, dễ context rotted.
3. **Không trace được** từ requirement → design → task → code khi cần audit hoặc onboard người mới.

Template này giải quyết bằng 3 slash command + 3 loại tài liệu chuẩn:

| Tài liệu | Viết bởi | Phục vụ |
|----------|----------|---------|
| **RSD** (Requirement Spec) | Con người (BA/PO/Dev) | Mô tả "làm gì" nghiệp vụ |
| **PTTK** (Phân tích Thiết kế) | `/rsd-to-pttk` | Mô tả "làm như thế nào" về kỹ thuật |
| **Implementation Plan** | `/pttk-to-plan` | Chia thành task ~30-60' để code lần lượt |
| Code thật | `/implement-task` | Implement + test + update plan |

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

```
┌──────────┐   /rsd-to-pttk   ┌──────────┐   /pttk-to-plan   ┌──────────────┐   /implement-task   ┌──────────┐
│ RSD file │  ───────────────►│ PTTK file│  ────────────────►│ Plan file    │  ──────────────────►│ Code +   │
│ (.md)    │                  │ (.md)    │                   │ (.md, có task│                     │ Plan đã  │
│          │                  │          │                   │ list)        │                     │ update   │
└──────────┘                  └──────────┘                   └──────────────┘                     └──────────┘
     ▲                              ▲                               ▲                                    │
     │                              │                               │                                    │
     └──── docs/rsd/ ────────────── docs/pttk/ ────────────────── docs/implementation-plan/ ◄────────────┘
```

Ví dụ thực tế cho feature "quản lý đơn hàng":

```text
1. Viết tay:    docs/rsd/order-management-rsd.md     (dùng templates/rsd-template.md)
2. Chạy:        /rsd-to-pttk docs/rsd/order-management-rsd.md
                → sinh docs/pttk/order-management-pttk.md
3. Chạy:        /pttk-to-plan docs/pttk/order-management-pttk.md
                → sinh docs/implementation-plan/order-management-plan.md
4. Chạy:        /implement-task order-management TASK-001
                → code + test + update plan
5. Lặp lại:     /implement-task order-management TASK-002, TASK-003, ...
```

---

## 5. Cấu trúc thư mục

```
.
├── .claude/
│   ├── commands/               # 4 slash command chính
│   │   ├── rsd-to-pttk.md
│   │   ├── pttk-to-plan.md
│   │   ├── implement-task.md
│   │   └── sync-pttk-template.md
│   ├── output-templates/       # ACTIVE template: command đọc & gen theo
│   │   ├── source/             # Word file PTTK (source of truth)
│   │   │   └── pttk.docx       # User đặt file Word vào đây
│   │   └── pttk.md             # Auto-gen từ docx, /rsd-to-pttk đọc
│   └── settings.json           # Permissions: allow / deny / ask
│
├── docs/                       # Output của workflow, commit chung với code
│   ├── rsd/                    # Requirement specs (con người viết)
│   ├── pttk/                   # Phân tích thiết kế (Claude sinh, người review)
│   └── implementation-plan/    # Task lists (Claude sinh, Claude cập nhật khi code)
│
├── templates/                  # GENERIC samples: tham khảo / copy-paste tay
│   ├── rsd-template.md
│   ├── pttk-template.md
│   └── implementation-plan-template.md
│
├── README.md                   # File này
└── .gitignore
```

Giải thích:

- **`.claude/`**: commit vào repo để cả team dùng chung. KHÔNG ignore. Phần cá nhân để ở `.claude/settings.local.json` (đã được ignore mặc định).
- **`docs/`**: mọi tài liệu sinh ra trong workflow đều ở đây để cùng versioning với code.
- **`.claude/output-templates/`** vs **`templates/`** — hai loại template khác mục đích, đừng nhầm (xem mục 5.1).

### 5.1 Phân biệt `.claude/output-templates/` vs `templates/`

| | `.claude/output-templates/` | `templates/` |
|--|--|--|
| **Vai trò** | **Active** — slash command đọc & sinh output theo file này | **Generic samples** — để người đọc / copy-paste tay |
| **Ai sửa?** | Bạn (mỗi project customize riêng) | Hiếm khi sửa, là một phần của repo template |
| **Ảnh hưởng khi sửa** | Thay đổi NGAY output mà command sinh ra | Không ảnh hưởng command, chỉ thay sample |
| **Khi nào dùng?** | Mỗi project bootstrap → customize cho convention team | Khi muốn viết tay PTTK / cần xem mẫu chi tiết |

**Workflow customize active template** — có 2 cách:

**Cách A — Sửa markdown trực tiếp** (dev quen markdown):

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
