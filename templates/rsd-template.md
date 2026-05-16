# RSD: <Tên Feature>

> **RSD** = Requirement Specification Document — tài liệu yêu cầu nghiệp vụ. Đây là **input** cho slash command `/rsd-to-pttk`.
>
> Hướng dẫn: Copy file này thành `docs/rsd/<feature-name>-rsd.md`, điền vào các phần `<...>`, xoá hướng dẫn này khi xong.

---

## 1. Thông tin chung

| Trường | Giá trị |
|--------|---------|
| Tên feature | <ví dụ: Quản lý đơn hàng> |
| Mã feature | <ví dụ: ORD-001> |
| Owner nghiệp vụ | <PO / BA phụ trách> |
| Owner kỹ thuật | <Tech Lead> |
| Version | 1.0 |
| Ngày tạo | <YYYY-MM-DD> |
| Trạng thái | Draft / Reviewed / Approved |

## 2. Mục đích nghiệp vụ

<Viết 2-4 câu trả lời câu hỏi: Vì sao cần feature này? Nó giải quyết vấn đề gì cho người dùng / cho business?>

## 3. Phạm vi

### 3.1 In-scope
- <Tính năng A>
- <Tính năng B>

### 3.2 Out-of-scope
- <Những thứ KHÔNG làm trong scope này, để tránh hiểu lầm>

## 4. Yêu cầu chức năng

> Đánh số dạng FR-001, FR-002, ... để có thể trace ngược từ PTTK / code / test.

- **FR-001**: <Mô tả yêu cầu cụ thể, đo lường được. Ví dụ: "Khách hàng có thể tạo đơn hàng mới với ít nhất 1 sản phẩm trong giỏ.">
- **FR-002**: <...>
- **FR-003**: <...>

## 5. Yêu cầu phi chức năng

- **Performance**: <ví dụ: API tạo đơn < 500ms p95 với 100 RPS>
- **Security**: <ví dụ: chỉ user đã đăng nhập, mã hoá thông tin thanh toán>
- **Tính sẵn sàng**: <ví dụ: 99.5% uptime tháng>
- **Khả năng mở rộng**: <ví dụ: chịu được x3 tải vào dịp sale>
- **Tương thích**: <ví dụ: hỗ trợ trình duyệt Chrome 100+, mobile webview>
- **Audit / Compliance**: <log thao tác, GDPR, PCI-DSS nếu áp dụng>

## 6. Acceptance Criteria

> Điều kiện để feature được coi là "Done". Mỗi tiêu chí phải verify được (test pass / không pass).

- [ ] <AC-1: ví dụ "User tạo đơn thành công nhận được mã đơn dạng ORD-YYYYMMDD-XXXXX">
- [ ] <AC-2: ...>
- [ ] <AC-3: ...>

## 7. Câu hỏi mở / Giả định

> Phần này ghi lại những điểm CHƯA chắc chắn để PO / BA xác nhận trước khi sang bước PTTK.

- **Q1**: <Câu hỏi cần trả lời>
- **Giả định 1**: <Điều mà tài liệu này đang giả định là đúng — cần xác nhận>

## 8. Tham chiếu

- Wireframe / Figma: <link>
- Tài liệu liên quan: <link>
- Quy định nghiệp vụ tham chiếu: <link>
