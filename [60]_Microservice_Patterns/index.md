# Microservice Pattern

## Event Souring
- 모놀리틱
    - 단일 데이터베이스
    - 트랙잭션 처리 -> ACID
        - Atomicity (원자성)
        - Consistency (일관성)
        - Isolation (독립성)
        - Durable (지속성)
- 마이크로서비스
    - 각 서비스마다 독립적인 DB / 언어
    - API 를 통해 접근
    - Atomicity, Consistency -> Commit Transaction
    - MSA 에서는 지원하기 힘들다.

| ORDER ID | USER ID | CATEGORY ID | STATUS |
| --- | --- | --- | --- |
| 1 | user-001 | category-001 | CONFIRMED |
| 2 | user-001 | category-001 | PENDING |

> 주문 성공시 PENDING 상태 -> CONFIRMED 혹은 취소시 CANCELLED 상태로 변경한다.

`이벤트 소싱`

- 데이터의 마지막 상태만 저장하는 것이 아닌, 해당 데이터에 수행된 전체 이력을 기록한다.
- 데이터 구조 단순
- 일관성과 트랜잭션 처리가 가능하다.
- INSERT 개념만 존재한다.
- 데이터 저장소의 개체를 직접 수정하지 않기 때문에 동시성에 대한 충돌 문제가 발생하지 않는다.
- 모든 이벤트가 히스토리화 되어 남아있다.

`도메인 주도 설계`

- Aggregate
- Projection
- 메시지 중심의 비동기 작업 처리
- 단점
    - 모든 이벤트에 대한 복원 -> 스냅샷을 사용해서 단점 극복
    - 다양한 데이터 조회 -> CQRS

## CQRS
- 명령과 조회의 책임 분리
    - 상태를 변경하는 COMMAND
    - 조회를 담당하는 QUERY

## SAGA
- Application 에서 트랜잭션 처리
- Application 이 분리된 경우 각각 Local, Transaction 만 처리
- 각 APP 에 대한 연속적인 트랜잭션에서 실패할 경우
    - ROLLBACK 처리 구현 -> **보상 Transaction** 이라고 불린다.
- 데이터의 원자성을 보장하지는 않지만, 일관성을 보장한다.

> SAGA Pattern 의 핵심은, ROLLBACK 처리 (보상 트랜잭션) 이 존재한다는 점이다.

`Choreography-based SAGA`

- 주문 서비스에서 주문 요청을 수신하고 PENDING 상태로 만듬
- 주문 생성 이벤트 전달
- 고객 서비스의 이벤트 핸들러가 Credit 예약 시도
- 결과 이벤트 전달
- 주문 서비스의 EventHandler 를 통해 주문 승인 / 거부
    
`Orchestration-based SAGA`

- 주문 서비스를 수신하고, Order saga orchestrator 생성
- Order saga orchestrator 가 PENDING 상태 주문 생성
- Credit 예약 처리
- 결과 메세지 전달
- Order saga orchestrator 에서 주문 승인 / 거부

> Choreography 와 의 큰 차이점은 중앙 메세지 브로커에서 모든것을 통제한다는 점이다.