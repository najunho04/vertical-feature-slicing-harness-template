# 슬라이스: order-cancel

> 작성 예시. 실제 프로젝트에서는 삭제하고 사용.

## 목적
사용자가 본인 주문을 취소하고 환불 기록을 남긴다.

## 경로
- 백: backend/src/feature/order-cancel/
- 프론트: front/src/feature/order-cancel/

## 데이터
- 읽기: orders, order_items
- 쓰기: orders(status), refunds

## 계약
- POST /api/orders/{orderId}/cancel
- Request: { reason: string }
- Response: { order: OrderDto, refundId: string }
- Error: ORDER_ALREADY_SHIPPED, ORDER_NOT_FOUND, FORBIDDEN

## 상하류
- 상류: 주문 상세 화면에서 orderId로 호출
- 하류: 없음 (알림은 이벤트로)

## 공유 상태
- 무효화: orderList, orderDetail
- 발행 이벤트: order.cancelled

## 횡단 관심사
- 권한: 본인 주문만, 관리자는 전체
- 트랜잭션: 환불 생성 + 상태 변경이 하나의 트랜잭션

## 불변조건
- status가 shipped 이후면 취소 불가
- 이미 cancelled면 멱등적 처리

## 충돌 주의
- orders 쓰는 다른 피쳐: order-create, shipping-update

## 허용 의존
- `common/`, `shared/`만

## UI 결과
- 취소 완료 후 주문 상세에 상태 `CANCELLED`와 환불 ID 표시
- 필요 상태: 로딩 / 빈 상태 / 에러
