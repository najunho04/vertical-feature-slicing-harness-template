# 전체 API 명세

> 사람이 조망하는 용도. 세션에는 넣지 않는다. 여기서 나온 DTO/enum은 `project/*/src/common/`에 코드로 내려놓는다.

| 슬라이스 | Method | Path | Request | Response | Error |
|---|---|---|---|---|---|
| order-cancel (예시) | POST | /api/orders/{orderId}/cancel | `{ reason: string }` | `{ order: OrderDto, refundId: string }` | ORDER_ALREADY_SHIPPED, ORDER_NOT_FOUND, FORBIDDEN |
