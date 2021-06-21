# ECommerce Application

## 구성 요소
- EurekaService (Eureka Server)
- API Gateway
- Config Server
- Kafka
- Microservices

## Service API 목록
- Category Service
    - GET /catalog-service/catalogs : 상품 목록 제공
- User Service
    - POST /user-service/users : 사용자 정보 등록
    - GET /user-service/users : 전체 사용자 조회
    - GET /user-service/users/{user_id} : 사용자 정보, 주문 내역 조회
- Order Service
    - POST /order-service/users/{user_id}/orders : 주문 등록
    - /order-service/users/{user_id}/orders : 주문 확인