# k_server_develop

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
