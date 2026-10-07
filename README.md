# k_server_develop

<br>

# ERD
<img width="960" height="685" alt="image" src="https://github.com/user-attachments/assets/615cdd5e-4722-4f83-8b70-7d74fe6cf199" />

<br>

# API 명세서

## 커피 메뉴 목록 조회 API

| 항목 | 내용 |
| --- | --- |
| Method | GET |
| URL | /api/menus |
| 설명 | 판매 중인 커피 메뉴 목록을 조회한다. |

### Request Body

### Response (200 OK)

```json
{
  "code": "SUCCESS",
  "data": [
    {
      "menuId": 1,
      "name": "아메리카노",
      "price": 4000
    },
    {
      "menuId": 2,
      "name": "카페라떼",
      "price": 4500
    }
  ]
}
```

### Error Response

`500 INTERNAL_SERVER_ERROR` - 서버 오류 발생

<br>

## 포인트 충전하기 API

| 항목 | 내용 |
| --- | --- |
| Method | POST |
| URL | /api/points/charge |
| 설명 | 사용자의 포인트를 충전한다. |

### Request Body
```
{
  "customerId": 1,
  "amount": 10000
}
```

### Response (200 OK)
```
{
  "code": "SUCCESS",
  "data": {
    "customerId": 1,
    "balance": 15000
  }
}
```
### Error Response

`400 BAD_REQUEST`
```
{
  "code": "INVALID_AMOUNT",
  "message": "충전 금액은 0보다 커야 합니다."
}
```

`404 NOT_FOUND`
```
{
  "code": "CUSTOMER_NOT_FOUND",
  "message": "존재하지 않는 사용자입니다."
}
```
<br>

## 동시성 제어
point의 데이터 무결성을 유지하기 위해 customer 전체에 락을 적용하는것은 너무 비효율적이라 판단, 또한 미래에 포인트 기한 만료나 포인트 환불가 같은 기능의 확장이 일어날 수 있음을 고려하여 customer에서 point 테이블을 따로 분리하였습니다.
동시에 포인트 추가 요청이 들어오는 경우를 방지하기 위해, point repository에 `비관적 락` 적용.
`point service`의 충전 매서드를 `트랜잭션`으로 감싸고 findbyid에 lock을 적용시켜 구현한다. 
```
적용 예시
//PointRepository
@Lock(LockModeType.PESSIMISTIC_WRITE)
@Query("SELECT p FROM Point p WHERE p.customer.id = :customerId")
Optional<Point> findByCustomerIdWithLock(Long customerId);

//PointService
@Transactional
public ~~~ charge(){
        Point point = pointRepository.findByCustomerIdWithLock(...);
}

//OrderService
public ~~~ order(){
        Point point = pointRepository.findByCustomerIdWithLock(...);
}
```
## 커피 주문, 결제하기 API

| 항목 | 내용 |
| --- | --- |
| Method | POST |
| URL | /api/orders |
| 설명 | 커피 주문을 생성 및 결제한다. |

### Request Body
```
{
  "customerId": 1,
  "menuId": 1
}
```

### Response (200 OK)
```
{
  "code": "SUCCESS",
  "data": {
    "orderId": 1,
    "customerId": 1,
    "menuId": 1,
    "price": 4000,
    "status": "COMPLETED"
  }
}
```

### Error Response

`400 BAD_REQUEST`
```
{
  "code": "INSUFFICIENT_POINT",
  "message": "포인트가 부족합니다."
}
```

`404 NOT_FOUND`
```
{
  "code": "MENU_NOT_FOUND",
  "message": "존재하지 않는 메뉴입니다."
}
```

<br>

### 동시성 처리
주문 생성 및 결제에서도, Order Service내의 `주문 및 결제` 전체 과정을 `트랜잭션`으로 감싸고, pointrepository에 미리 만들어둔 findbycustomeridwithlock 쿼리를 사용하여 비관적 락을 적용한다.

## 인기 메뉴 목록 조회 API

| 항목 | 내용 |
| --- | --- |
| Method | GET |
| URL | /api/menus/popular |
| 설명 |  최근 7일간 주문 횟수가 많은 메뉴 3개를 조회한다. |

### Request Body

### Response (200 OK)
```
{
  "code": "SUCCESS",
  "data": [
    {
      "menuId": 1,
      "name": "아메리카노",
      "orderCount": 152
    },
    {
      "menuId": 2,
      "name": "카페라떼",
      "orderCount": 121
    },
    {
      "menuId": 3,
      "name": "바닐라라떼",
      "orderCount": 98
    }
  ]
}
```

### 인기 메뉴 조회 방식

- 최근 7일 이내의 주문관리
- 주문 상태가 `COMPLETED` 인것만 집계
- 가장 주문횟수가 많은 메뉴로부터 내림차순 후, 상위 3개 메뉴 반환하기 ( 만약, 반환된 메뉴가 3개보다 많은경우 메뉴id가 낮은 순서부터 출 )
