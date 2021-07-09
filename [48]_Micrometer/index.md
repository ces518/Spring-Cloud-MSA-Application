# Micrometer

## Turbine Server
- 마이크로서비스에 설치된 Hystrix 클라이언트의 스트림을 통합
- 마이크로서비스에서 생성된 Hystrix 클라이언트 스트림 메시지를 터빈 서버로 수집한다.

## Hystrix Dashboard
- Hystrix 클라이언트에서 생성하는 스트림을 시각화
- Web Dashboard

> Micrometer + Monitoring System 사용 권고

## Micrometer
- https://micrometer.io/
- JVM 기반 애플리케이션 메트릭 제공
- SpringFramework 5, Spring Boot 2 부터 Spring 의 Metrics 처리
- Prometheus 등 모니터링 시스템 지원

## Timer
- 짧은 지연 시간, 이벤트의 사용 빈도를 측정
- 시계열로 이벤트의 시간, 호출 빈도 등을 제공
- @Timed 제공

## 설정

`의존성 추가`

```xml
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-registry-prometheus</artifactId>
</dependency>
```

`yaml`

```yaml
management:
  endpoints:
    web:
      exposure:
        include: refresh, health, beans, busrefresh, info, prometheus, metrics 
```