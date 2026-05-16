---
description: Convert Word RSD template (.docx) thành rsd.md chuẩn hóa để /rsd-to-pttk dùng
argument-hint: [đường-dẫn-tới-file-docx] [tên-feature]  (mặc định .claude/output-templates/source/rsd.docx)
---

# /sync-rsd-template — Đồng bộ Word RSD → Markdown RSD

Workflow: BA hoặc Product Owner thường viết RSD bằng **Word (.docx)**. Slash command này convert file đó sang markdown chuẩn hóa để `/rsd-to-pttk` đọc và tiếp tục workflow.

```
.claude/output-templates/source/rsd.docx       ← BA sửa (Word)
                ↓ /sync-rsd-template [tên-feature]
docs/rsd/<tên-feature>-rsd.md                  ← Chuẩn hóa, /rsd-to-pttk dùng
                ↓ /rsd-to-pttk docs/rsd/<tên-feature>-rsd.md
docs/pttk/<tên-feature>-pttk.md                ← PTTK sinh ra
```

## Quy trình 6 bước

### Bước 1 — Xác định input path

- Nếu `$ARGUMENTS` rỗng → dùng path mặc định `.claude/output-templates/source/rsd.docx`.
- Nếu `$ARGUMENTS` có giá trị → argument đầu tiên là đường dẫn file .docx (cho phép override).
- Kiểm tra file tồn tại. Nếu KHÔNG → DỪNG, báo user:
  > "Không tìm thấy `<path>`. Hãy đặt file Word RSD vào `.claude/output-templates/source/rsd.docx` rồi chạy lại."

### Bước 2 — Xác định feature name

- Nếu `$ARGUMENTS` có 2+ giá trị → argument thứ 2 là feature name (kebab-case, ví dụ: `order-management`).
- Nếu chỉ 1 hoặc không có argument → **hỏi user**: "Tên feature là gì? (dạng kebab-case, ví dụ: order-management)"
- Output path sẽ là: `docs/rsd/<feature-name>-rsd.md`

### Bước 3 — Kiểm tra pandoc

Chạy `pandoc --version`. Nếu lỗi (pandoc chưa cài) → DỪNG và hướng dẫn:

```
- Windows: winget install --id JohnMacFarlane.Pandoc
- macOS:   brew install pandoc
- Linux:   sudo apt install pandoc  (hoặc dnf/pacman tương ứng)
```

### Bước 4 — Convert .docx → markdown thô

Chạy lệnh:

```bash
pandoc "<input-path>" -t gfm --wrap=none -o ".claude/output-templates/.rsd.raw.md"
```

- `-t gfm`: GitHub Flavored Markdown (giữ table, list, heading).
- `--wrap=none`: không wrap line, dễ Edit sau.
- Output ra file tạm `.rsd.raw.md` trong cùng thư mục.

Nếu pandoc trả error → DỪNG, in stderr cho user.

### Bước 5 — Refine markdown thô thành RSD chuẩn

Đọc file `.rsd.raw.md`, sau đó **biến đổi** thành RSD chuẩn:

1. **Header**: Thêm khối comment chuẩn ở đầu file. Mẫu:

   ```html
   <!--
   =========================================================================
   RSD (Requirement Specification) — Chuẩn hóa từ Word
   =========================================================================
   File này được auto-generate từ:
     .claude/output-templates/source/rsd.docx
   bằng lệnh /sync-rsd-template.

   KHÔNG sửa trực tiếp file này — sửa file Word rồi chạy lại lệnh.
   Last synced: <YYYY-MM-DD HH:mm>
   =========================================================================
   -->
   ```

2. **Tiêu đề chính**: Đảm bảo dòng `# RSD: <Tên Feature>` ở đầu. Normalize heading level cho toàn file (section chính `##`, sub-section `###`).

3. **Chuẩn hóa cấu trúc**: Kiểm tra các section quan trọng có mặt không. Reference `templates/rsd-template.md` để biết expected structure:
   - **Thông tin chung** (Purpose, Scope, etc.)
   - **Yêu cầu chức năng** (FR-001, FR-002, ...)
   - **Yêu cầu phi chức năng**
   - **Tiêu chí chấp nhận**
   
   Nếu Word thiếu section quan trọng → thêm section rỗng với note `<!-- TODO: BA fill in -->`.

4. **Loại bỏ artifact** pandoc hay sinh ra:
   - `{.class-name}` attribute cuối heading → bỏ.
   - Image link `![](media/image1.png)` → thay bằng `<!-- Diagram: mô tả ngắn -->`.
   - Footnote rỗng → bỏ.
   - Multiple blank lines → gộp thành 1.
   - Table có empty cell → check xem có intentional không, nếu không → clean up.

5. **Giữ ngôn ngữ**: Nếu Word viết tiếng Việt → giữ tiếng Việt, không dịch.

6. **Comment hướng dẫn** (optional): Trước các section quan trọng, có thể thêm HTML comment ghi rule. Ví dụ:
   ```html
   <!--
   QUAN TRỌNG: Mỗi FR PHẢI có:
   - Mô tả rõ ràng
   - Actor / Role
   - Acceptance Criteria
   -->
   ```

### Bước 6 — Ghi ra RSD chuẩn + Báo cáo

1. Tạo thư mục `docs/rsd/` nếu chưa tồn tại.
2. Ghi nội dung đã refine vào `docs/rsd/<feature-name>-rsd.md`.
3. Xóa file tạm `.rsd.raw.md`.

Format báo cáo:

```
✅ Sync xong

Input:   <path-tới-docx>
Output:  docs/rsd/<feature-name>-rsd.md

Sections detected (N):
- <section 1 từ Word>
- <section 2>
...

Sections missing (suggest thêm):
- <section nếu thiếu>

Last synced: <YYYY-MM-DD HH:mm>

Next: chạy /rsd-to-pttk docs/rsd/<feature-name>-rsd.md để gen PTTK.
```

## Nguyên tắc bắt buộc

- **Word file là source of truth**. KHÔNG sửa `docs/rsd/<feature-name>-rsd.md` thủ công — sẽ bị ghi đè khi sync.
- **Idempotent**: chạy lệnh nhiều lần phải cho cùng kết quả (cùng input → cùng output).
- **Không tự thêm FR/NFR mới** không có trong Word. Nếu thấy thiếu section, báo cáo cho user, không tự sinh.
- **Giữ cấu trúc**: RSD là requirement, không thiết kế. Giữ nguyên structure từ Word, chỉ chuẩn hóa format.
- **Lỗi rõ ràng**: nếu pandoc lỗi hoặc file không parse được, báo cụ thể lỗi gì để user fix Word file.
