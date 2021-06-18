# SOA 와 MSA

## 서비스 공유 지향점
- SOA 
    - 재사용을 통한 비용 절감
    - 서비스 공유 최대화
- MSA 
    - 서비스 간 결합도를 낮춰 변화에 능동적으로 대응
    - 서비스 공유 최소화

## 기술 방식
- SOA
    - 공통 서비스를 ESB 에 모아 공통 서비스로 제공
    - 서비스 버스 (공통 서비스) 라는 형태로 제공
- MSA
    - 각 독립된 서비스가 REST API 를 사용
    
## RESTful Web Service

`서비스 성숙도 LEVEL`
- LEVEL 0
    - https://server/getPosts
    - https://server/deletePosts
> 그냥 리소스를 제공하는 수준

- LEVEL 1
    - http://server/accounts
    - http://server/accounts/10
> URI 로 리소스를 잘 표현하고 있지만 적절한 HTTP METHOD 를 사용하고 있진 않음

- LEVEL 2
    - LEVEL 1 + HTTP METHODS
> 리소스를 용도와 상태에 따라 적절히 사용

- LEVEL 3
    - LEVEL 2 + HATEOAS
> 데이터와 리소스에 대한 설명이 되어 있음, 다음 가능한 작업에 대한 표현이 되어 있음

> https://ohjongsung.io/2018/07/21/rest-api-%EC%84%B1%EC%88%99%EB%8F%84-%EB%AA%A8%EB%8D%B8-maturity-model

## API 설계시 고려해야할 사항
- 사용자 입장에서 간단 명료한 설계가 필요
- HTTP의 장점을 최대한 살려야함
- HTTP METHODS 를 사용 (최소한 LEVEL 2)
- HTTP 응답 코드 활용
- 보안과 관련된 정보는 URI 에 존재해서는 안된다.
- 단수형태 URI 가 아닌 복수형태 URI 를 사용해야 한다
- 모든 리소스는 동사보다는 **명사** 로 표현해야한다
- 일관적인 END-POINT 사용
