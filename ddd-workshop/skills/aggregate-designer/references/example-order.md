# 예제: Order Aggregate (B2C 커머스)

파일 경로: `docs/domains/order/order.md`

---

```markdown
---
aggregate: Order
bc: Order
classification: Core
status: active
version: 0.1.0
last_reviewed: 2026-04-21
---

# Order

## Role

구매자의 구매 의사를 기록하고 **상태 전이**를 관리한다.
결제 승인은 Payment BC, 재고 확보는 Inventory BC의 책임이다.
환불·배송·정산은 이 Aggregate의 범위 밖이다.

## Ubiquitous Language

| 용어 | 의미 |
|------|------|
| 주문 (Order) | 구매자가 한 번에 결제하려는 상품 묶음 단위 |
| 주문항목 (OrderItem) | 주문에 포함된 상품 1종 + 수량. Order 없이 단독 존재 불가. |
| 주문상태 (OrderStatus) | `PENDING → PAID / FAILED`, `PAID → CANCELED` |
| 총금액 (Money) | 주문의 지불 금액 (VO: amount + currency) |

## Structure

- **Root**: Order
- **내부 Entity**: OrderItem
- **VO**: OrderStatus, Money, ShippingAddress
- **외부 참조 (ID only)**:
  - `buyerId: UserId` (Identity BC)
  - `items[].productId: ProductId` (Catalog BC)

## Invariants

- 주문은 최소 1개 이상의 OrderItem을 가진다.
- 총금액은 모든 OrderItem(단가 × 수량)의 합과 일치한다.
- 상태 전이: `PENDING → PAID`, `PENDING → FAILED`, `PAID → CANCELED`만 허용.
- `FAILED` 상태는 `PENDING`으로 되돌릴 수 없다.
- OrderItem의 단가·수량은 생성 후 변경 불가 (스냅샷 원칙).
- 모든 OrderItem의 통화는 동일해야 한다.

## Policies

### DiscountPolicy
- **위치**: Strategy (interface + 다형 구현체)
- **이유**: 마케팅팀이 자주 바꿈. 이벤트 쿠폰, VIP 할인, 시즌 할인 등이 빈번히 추가됨. 프로덕션 무중단 교체 요구.
- **설명**: OrderItem 목록 + 구매자 정보 + 현재 시점을 입력받아 할인 금액을 산출한다.
- **교체 시나리오**: A/B 테스트, 블랙프라이데이 한정 전략.

### MinOrderAmountSpecification
- **위치**: Specification
- **이유**: "주문 최소 금액 미만 시 주문 불가"는 조건 판별이며 배송 정책과도 조합된다.
- **설명**: 배송 방법·지역별로 다른 최소 금액을 적용.

### OrderExpirationPolicy
- **위치**: Event Handler (Scheduled)
- **이유**: 결제 미완료 주문을 일정 시간 후 자동 취소. 이벤트 기반 비동기 반응.
- **설명**: `주문됨` 이벤트 후 N분 내 `결제완료됨`이 오지 않으면 `주문만료됨` 발행.

## External Dependencies

| Aggregate / BC | 참조 방식 | 비고 |
|----------------|-----------|------|
| Identity (User) | `buyerId: UserId` | 사용자 상세는 Identity 소관. 본 Aggregate는 ID만 보유. |
| Catalog (Product) | `productId: ProductId` | 생성 시점 가격·옵션을 스냅샷. 이후 상품 변경과 독립. |
| Payment | 이벤트 구독 (`결제완료됨`, `결제실패됨`) | 결과 이벤트로 상태 전이. |
| Inventory | 이벤트 발행 (`주문됨` → 재고 확인 요청) | ACL로 양 도메인 분리. |

## Domain Events (발행)

- `주문됨` — 생성 직후. payload: orderId, buyerId, items(productId/qty/unitPrice), totalAmount.
- `주문결제확정됨` — `PAID` 전이 시.
- `주문취소됨` — `CANCELED` 전이 시. 사유 포함.
- `주문만료됨` — 타임아웃으로 자동 취소 시.

## Domain Events (구독)

- `결제완료됨` (Payment BC) — `PENDING → PAID` 전이 트리거.
- `결제실패됨` (Payment BC) — `PENDING → FAILED` 전이 트리거.
- `재고부족됨` (Inventory BC) — `PENDING → FAILED` 전이 트리거.

## Open Questions

- ⚠️ **부분 취소 지원 여부**: 항목 일부만 취소 시 Invariant("최소 1개 항목") 완화 vs 새 Aggregate로 분리?
- ⚠️ **통화 혼합 허용 케이스**: 해외 직구 확장 시 같은 주문 내 다통화 허용 재검토.
- ⚠️ **주문 만료 시간 설정값**: 정책 vs Invariant — 현재는 Event Handler의 설정으로 관리.
```
