# Spring Cloud Config - Profiles

## Profile
- 각 배포 환경 (dev, prod 등..) 에 맞게 설정 정보를 관리하고, 적용이 가능함
- application-name-<profile>.yaml
    - profile 정보가 없다면 default 파일을 fetch 한다.

`bootstrap.yml`

```yaml
spring:
  cloud:
    config:
      uri: http://127.0.0.1:8888
      name: ecommerce
  profiles:
    active: dev
```

- yml 파일에 profiles.active 혹은 VM Options 로 넣어주면 해당 프로파일에 맞는 설정을 fetch 한다.