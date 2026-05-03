# 예제: Order Aggregate (B2C 커머스)

파일 경로: `docs/shared/contexts/order/order.md`

---

```markdown
# Order

> **Status**: active

## Role

### 책임지는 것
- 구매자의 구매 의사 기록 및 **상태 전이** 관리
- 주문 항목·총액 스냅샷 보존 (가격 변동과 독립)
- 주문 생애 이벤트 발행

### 책임지지 않는 것
- 결제 승인 — Payment BC
- 재고 확보 — Inventory BC
- 환불·배송·정산 — 각각 별 BC 소관

## Ubiquitous Language (이 Aggregate 범위)

| 용어 | Code Identifier | 의미 |
|---|---|---|
| 주문 | Order | 구매자가 한 번에 결제하려는 상품 묶음 단위 |
| 주문항목 | OrderItem | 주문에 포함된 상품 1종 + 수량. Order 없이 단독 존재 불가 |
| 주문상태 | OrderStatus | `PENDING → PAID / FAILED`, `PAID → CANCELED` |
| 총금액 | Money | 주문의 지불 금액 (VO: amount + currency) |

## Event Storming (Design-Level)

### 관련 플로우

```
[구매자] →(주문하기)→ [주문됨]
  ↓ 정책: 재고 확인 + 결제 요청
  [결제완료됨] → PAID or [결제실패됨] → FAILED
  [주문만료됨] (타임아웃 시)
```

### 해소된 핫스팟
- 결제 실패 시 재고 복원: 즉시 (Inventory BC가 `결제실패됨` 구독)

### 미해소 핫스팟
- ⚠️ 부분 취소 지원 여부

## Structure

- **Root**: Order
- **내부 Entity**: OrderItem
- **VO**: OrderStatus, Money, ShippingAddress
- **외부 참조 (ID only)**:
  - `buyerId: UserId` (Identity BC)
  - `items[].productId: ProductId` (Catalog BC)

## Invariants

1. 주문은 최소 1개 이상의 OrderItem을 가진다.
2. 총금액은 모든 OrderItem(단가 × 수량)의 합과 일치한다.
3. 상태 전이: `PENDING → PAID`, `PENDING → FAILED`, `PAID → CANCELED` 만 허용.
4. `FAILED` 상태는 `PENDING`으로 되돌릴 수 없다.
5. OrderItem의 단가·수량은 생성 후 변경 불가 (스냅샷 원칙).
6. 모든 OrderItem의 통화는 동일해야 한다.

## Policies

### DiscountPolicy
- **위치**: Strategy (interface + 다형 구현체)
- **이유**: 마케팅이 자주 바꿈. 프로덕션 무중단 교체 요구.
- **설명**: OrderItem·구매자·현재시점을 입력받아 할인 금액 산출.
- **교체 시나리오**: A/B 테스트, 블랙프라이데이 한정 전략.

### MinOrderAmountSpecification
- **위치**: Specification
- **이유**: "주문 최소 금액 미만 시 주문 불가"는 조건 판별이며 배송 정책과도 조합된다.

### OrderExpirationPolicy
- **위치**: Event Handler (Scheduled)
- **이유**: 결제 미완료 주문을 일정 시간 후 자동 취소.

## External Dependencies

| 대상 | 관계 | 참조 방식 | 비고 |
|---|---|---|---|
| Identity BC (User) | Customer-Supplier | `buyerId: UserId` | Identity 소관, ID만 보유 |
| Catalog BC (Product) | Customer-Supplier | `productId: ProductId` | 생성 시 가격·옵션 스냅샷 |
| Payment BC | 이벤트 구독 | `결제완료됨/결제실패됨` | 결과로 상태 전이 |
| Inventory BC | 이벤트 발행·구독 | `주문됨` 발행 → `재고부족됨` 수신 | ACL로 분리 |

## Domain Events (발행)

- `주문됨` (OrderPlaced) — 생성 직후. payload: orderId, buyerId, items, totalAmount.
- `주문결제확정됨` (OrderConfirmed) — `PAID` 전이 시.
- `주문취소됨` (OrderCancelled) — `CANCELED` 전이 시. 사유 포함.
- `주문만료됨` (OrderExpired) — 타임아웃 자동 취소.

## Domain Events (구독)

- `결제완료됨` (Payment BC) — `PENDING → PAID` 트리거.
- `결제실패됨` (Payment BC) — `PENDING → FAILED` 트리거.
- `재고부족됨` (Inventory BC) — `PENDING → FAILED` 트리거.

## Exposed Queries

| Query | 반환 | 원천 | 주의 |
|---|---|---|---|
| `getOrder(orderId, viewerId)` | `OrderDetail` | Query Service | 권한 체크 포함 |
| `listMyOrders(buyerId, status?, range?)` | `OrderSummary[]` | Query Service | 페이지네이션 |
| `getOrderStatus(orderId)` | `OrderStatus` | Projection (빠른 조회) | 상태 변경 이벤트 구독 갱신 |

**Read-side 규약**: Aggregate 재조립 금지. Projection은 `주문결제확정됨/주문취소됨/주문만료됨` 구독으로 동기화.

## Open Questions

- ⚠️ 부분 취소 지원 여부: 항목 일부만 취소 시 Invariant("최소 1개 항목") 완화 vs 새 Aggregate 분리?
- ⚠️ 통화 혼합 허용 케이스: 해외 직구 확장 시 같은 주문 내 다통화 재검토.
- ⚠️ 주문 만료 시간 설정값: Policy vs Invariant — 현재 Event Handler 설정.
```
