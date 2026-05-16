# Implementation Plan: <Tên Feature>

> File này là **output** của `/pttk-to-plan` và sẽ được `/implement-task` cập nhật khi code. Cấu trúc dưới khớp với output mà command sinh ra.

---

## Metadata

- **PTTK reference**: [docs/pttk/<feature>-pttk.md](../pttk/<feature>-pttk.md)
- **Estimated effort**: <ví dụ: 8 giờ>
- **Created date**: <YYYY-MM-DD>
- **Status**: 🟡 In Progress

## Progress Summary

- **Total tasks**: 6
- **Completed**: 0 / 6
- **Last updated**: <YYYY-MM-DD>

## Legend

- ⬜ Not started
- 🟡 In Progress
- ✅ Done
- ❌ Blocked / Failed

---

## Tasks

### TASK-001: Tạo migration cho bảng `orders` và `order_items`

- **Status**: ⬜
- **Type**: Database
- **Files to create/modify**:
  - `src/main/resources/db/migration/V20260101_0001__create_orders.sql`
  - `src/main/resources/db/migration/U20260101_0001__drop_orders.sql`
- **Description**: Tạo schema cho 2 bảng `orders`, `order_items` theo ERD trong PTTK section 3.2. Thêm index trên `orders.code` (unique) và `orders.customer_id`.
- **Acceptance criteria**:
  - [ ] Migration chạy thành công trên DB local (`mvn flyway:migrate`)
  - [ ] Có script rollback
  - [ ] Index unique trên `orders.code` được tạo
  - [ ] FK `order_items.order_id → orders.id` với `ON DELETE CASCADE`
- **Dependencies**: (không có)
- **Test requirements**:
  - Smoke test: chạy migrate trên DB rỗng, sau đó rollback, không lỗi.
- **Notes**: <điền khi implement>

### TASK-002: Tạo Entity `Order`, `OrderItem` + JPA Repository

- **Status**: ⬜
- **Type**: Backend
- **Files to create/modify**:
  - `src/main/java/.../order/domain/Order.java`
  - `src/main/java/.../order/domain/OrderItem.java`
  - `src/main/java/.../order/domain/OrderStatus.java` (enum)
  - `src/main/java/.../order/repository/OrderRepository.java`
- **Description**: Map JPA entity tương ứng schema TASK-001. Repository extends `JpaRepository<Order, Long>` + custom method `findByCode(String)`.
- **Acceptance criteria**:
  - [ ] Entity có annotation đầy đủ, lifecycle callback `@PrePersist` set `createdAt`
  - [ ] Unit test cho repository `findByCode` (Testcontainers Postgres)
  - [ ] Build pass
- **Dependencies**: TASK-001
- **Test requirements**:
  - `OrderRepositoryTest` với Testcontainers
- **Notes**: <điền khi implement>

### TASK-003: DTO + Mapper

- **Status**: ⬜
- **Type**: Backend
- **Files to create/modify**:
  - `src/main/java/.../order/dto/CreateOrderRequest.java`
  - `src/main/java/.../order/dto/OrderResponse.java`
  - `src/main/java/.../order/mapper/OrderMapper.java`
- **Description**: DTO cho API + MapStruct mapper giữa Entity ↔ DTO.
- **Acceptance criteria**:
  - [ ] DTO có Bean Validation annotation (`@NotNull`, `@Size`, ...)
  - [ ] Mapper unit test (entity → response, request → entity)
- **Dependencies**: TASK-002
- **Test requirements**: `OrderMapperTest`
- **Notes**: <điền khi implement>

### TASK-004: `OrderService` — luồng tạo đơn (happy path + business rules)

- **Status**: ⬜
- **Type**: Backend
- **Files to create/modify**:
  - `src/main/java/.../order/service/OrderService.java` (interface)
  - `src/main/java/.../order/service/OrderServiceImpl.java`
  - `src/test/java/.../order/service/OrderServiceTest.java`
- **Description**: Implement `createOrder` theo BR-001 đến BR-003 trong PTTK. Inventory reservation mock ở task này.
- **Acceptance criteria**:
  - [ ] BR-001 (tổng tiền > 0) test pass
  - [ ] BR-002, BR-003 test pass
  - [ ] Sinh mã đơn dạng `ORD-YYYYMMDD-XXXXX`
- **Dependencies**: TASK-002, TASK-003
- **Test requirements**: Coverage ≥ 80% cho service
- **Notes**: <điền khi implement>

### TASK-005: `OrderController` + Controller test

- **Status**: ⬜
- **Type**: Backend + Test
- **Files to create/modify**:
  - `src/main/java/.../order/api/OrderController.java`
  - `src/test/java/.../order/api/OrderControllerTest.java`
- **Description**: REST endpoint `POST /api/v1/orders`, `GET /api/v1/orders/{id}`. Validation, error handling, security (chỉ owner / admin).
- **Acceptance criteria**:
  - [ ] Endpoint trả status code đúng theo PTTK section 3.1
  - [ ] Authorization test: user khác không xem được đơn của mình
  - [ ] Validation error trả 400 với body chuẩn
- **Dependencies**: TASK-004
- **Test requirements**: `@WebMvcTest` cho controller
- **Notes**: <điền khi implement>

### TASK-006: Integration test end-to-end + observability

- **Status**: ⬜
- **Type**: Test + Config
- **Files to create/modify**:
  - `src/test/java/.../order/OrderIntegrationTest.java`
  - `src/main/resources/application.yml` (thêm metric config)
- **Description**: Chạy E2E tạo đơn với DB thật (Testcontainers) + payment mock. Thêm metric `order.created.count`.
- **Acceptance criteria**:
  - [ ] Test tạo đơn end-to-end pass
  - [ ] Test rollback khi payment fail pass
  - [ ] Metric expose ở `/actuator/prometheus`
- **Dependencies**: TASK-005
- **Test requirements**: Test phải chạy trong CI
- **Notes**: <điền khi implement>

---

## Implementation Order

```
TASK-001 (DB migration)
    ↓
TASK-002 (Entity + Repository) ─┐
                                ├─ có thể parallel sau khi TASK-001 done
TASK-003 (DTO + Mapper) ────────┘
    ↓
TASK-004 (Service + unit test)
    ↓
TASK-005 (Controller + controller test)
    ↓
TASK-006 (Integration test + observability)
```

## Risks & Decisions Log

| Date | Task | Decision / Risk | Resolution |
|------|------|------------------|------------|
| <YYYY-MM-DD> | TASK-XXX | <Ví dụ: phát hiện inventory API chưa có endpoint reserve> | <Tạo issue, mock tạm trong task này, ticket riêng cho inventory team> |
