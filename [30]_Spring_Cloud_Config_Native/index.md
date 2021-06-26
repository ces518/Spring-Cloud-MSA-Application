# Spring Cloud Config - Native

## Native Repository
- Native File Repository 지원 기능

`Config Server's application.yaml`

```yaml
server:
  port: 8888 # Config Server Default Port : 8888

spring:
  application:
    name: config-service
  profiles:
    active: native # Native File Repo 를 사용할 경우 설정
  cloud:
    config:
      server:
        native:
          search-locations: file://${user.home}/work/native/file-repo # Native File Repo 를 사용할 경우 설정
#        git:
#          uri: file:///Users/kakaocommerce/work/git-local-repo
```
> 주의할점은 profile 을 native 로 요청해야 해당 설정 정보를 가져온다.