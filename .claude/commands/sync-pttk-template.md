---
description: Convert Word PTTK template (.docx) thành active markdown template để /rsd-to-pttk dùng
argument-hint: [đường-dẫn-tới-file-docx]  (mặc định .claude/output-templates/source/pttk.docx)
---

# /sync-pttk-template — Đồng bộ Word template → Markdown active template

Workflow: Team thường có file PTTK template viết bằng **Word (.docx)** làm source of truth. Slash command này convert file đó sang markdown chuẩn để `/rsd-to-pttk` đọc.

```
.claude/output-templates/source/pttk.docx   ← USER sửa (Word)
                ↓ /sync-pttk-template
.claude/output-templates/pttk.md            ← Auto-gen, /rsd-to-pttk đọc
                ↓ /rsd-to-pttk <rsd>
docs/pttk/<feature>-pttk.md                 ← PTTK thực tế của feature
```

## Quy trình 6 bước

### Bước 1 — Xác định input path

- Nếu `$ARGUMENTS` rỗng → dùng path mặc định `.claude/output-templates/source/pttk.docx`.
- Nếu `$ARGUMENTS` có giá trị → dùng path đó (cho phép user override khi file Word đặt chỗ khác).
- Kiểm tra file tồn tại. Nếu KHÔNG → DỪNG, báo user:
  > "Không tìm thấy `<path>`. Hãy đặt file Word PTTK template vào `.claude/output-templates/source/pttk.docx` rồi chạy lại."

### Bước 2 — Kiểm tra pandoc

Chạy `pandoc --version`. Nếu lỗi (pandoc chưa cài) → DỪNG và hướng dẫn:

```
- Windows: winget install --id JohnMacFarlane.Pandoc
- macOS:   brew install pandoc
- Linux:   sudo apt install pandoc  (hoặc dnf/pacman tương ứng)
```

### Bước 3 — Convert .docx → markdown thô

Chạy lệnh:

```bash
pandoc "<input-path>" -t gfm --wrap=none -o ".claude/output-templates/.pttk.raw.md"
```

- `-t gfm`: GitHub Flavored Markdown (giữ table, list, heading).
- `--wrap=none`: không wrap line, dễ Edit sau.
- Output ra file tạm `.pttk.raw.md` trong cùng thư mục.

Nếu pandoc trả error → DỪNG, in stderr cho user.

### Bước 4 — Refine markdown thô thành active template

Đọc file `.pttk.raw.md`, sau đó **biến đổi** thành active template chuẩn:

1. **Header**: Thêm khối comment chuẩn ở đầu file (giải thích đây là active template). Mẫu:

   ```html
   <!--
   =========================================================================
   ACTIVE TEMPLATE — đọc bởi slash command /rsd-to-pttk
   =========================================================================
   File này được auto-generate từ:
     .claude/output-templates/source/pttk.docx
   bằng lệnh /sync-pttk-template.

   KHÔNG sửa trực tiếp file này — sửa file Word rồi chạy lại lệnh.
   Last synced: <YYYY-MM-DD HH:mm>
   =========================================================================
   -->
   ```

2. **Tiêu đề chính**: Đảm bảo dòng `# PTTK: <Tên Feature>` ở đầu. Nếu Word đã có heading kiểu "PTTK Template" hoặc "Mẫu PTTK" → đổi thành `# PTTK: <Tên Feature>`.

3. **Placeholder hóa nội dung mẫu**:
   - Nội dung cụ thể (ví dụ tên feature thật, tên bảng cụ thể, BR-001 với mô tả chi tiết) → thay bằng `<...>` placeholder ngắn gọn.
   - Bảng có row mẫu → giữ 1 row mẫu với placeholder.
   - Code block mẫu (Java entity, SQL) → giữ skeleton + placeholder cho tên class/method.

4. **Comment hướng dẫn**: Trước các section quan trọng, thêm `<!-- -->` ghi rule. Ví dụ trước section Business Rules:

   ```html
   <!--
   QUAN TRỌNG: Mỗi BR PHẢI mapping với 1 hoặc nhiều FR-XXX trong RSD.
   Nếu không mapping được, ghi vào "Câu hỏi mở" và hỏi user.
   -->
   ```

5. **Normalize heading**: Đảm bảo level đúng. Section chính `##`, sub-section `###`, không nhảy cấp.

6. **Giữ NGUYÊN structure**: Mọi section custom của Word (kể cả section không có trong fallback của `/rsd-to-pttk`) phải được giữ. Không cắt bớt.

7. **Loại bỏ artifact** pandoc hay sinh ra:
   - `{.class-name}` attribute cuối heading → bỏ.
   - Image link `![](media/image1.png)` → thay bằng `<!-- Diagram: mô tả ngắn -->` hoặc nếu là sơ đồ, suggest dùng mermaid.
   - Footnote rỗng → bỏ.
   - Multiple blank lines → gộp thành 1.

### Bước 5 — Ghi ra active template

1. Nếu `.claude/output-templates/pttk.md` đã tồn tại → backup thành `.claude/output-templates/pttk.md.bak` (ghi đè bak cũ nếu có).
2. Ghi nội dung đã refine vào `.claude/output-templates/pttk.md`.
3. Xóa file tạm `.pttk.raw.md`.

### Bước 6 — Báo cáo

Format:

```
✅ Sync xong

Input:  <path-tới-docx>
Output: .claude/output-templates/pttk.md
Backup: .claude/output-templates/pttk.md.bak (nếu có file cũ)

Sections detected (N):
- <section 1 từ Word>
- <section 2>
...

Placeholders inserted: <số>
Pandoc warnings:       <nếu có>

Next: chạy /rsd-to-pttk docs/rsd/<feature>-rsd.md để gen PTTK mới.
```

## Nguyên tắc bắt buộc

- **Word file là source of truth**. KHÔNG sửa `.claude/output-templates/pttk.md` thủ công — sẽ bị ghi đè khi sync.
- **Idempotent**: chạy lệnh nhiều lần phải cho cùng kết quả (cùng input → cùng output).
- **Không tự thêm section** không có trong Word. Nếu thấy Word thiếu section quan trọng (ví dụ "Test Strategy"), báo cáo cho user trong "Pandoc warnings", không tự sinh.
- **Giữ ngôn ngữ trong Word**: nếu Word viết tiếng Việt thì giữ tiếng Việt, không dịch.
- **Lỗi rõ ràng**: nếu pandoc lỗi hoặc file không parse được, báo cụ thể lỗi gì để user fix Word file.
