# 대칭키 암호화

## ConfigServer 설정

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bootstrap</artifactId>
</dependency>
```
- bootstrap.yml 파일 구동방식을 사용하기 위해 의존성 추가

`bootstrap.yml`

```yaml
encrypt:
  key: abcdedfhijk1234 # encrypt key 추가, 해당 키가 존재해야 enc / dec 작업 수행이 가능함
```
- encrypt.key 프로퍼티를 설정하면 지정된 key 를 가지고 enc/dec 작업을 수행한다.

## 암/복호화
- Spring Cloud Config Server 를 사용하면, 암/복호화에 대한 End-Point 가 자동으로 등록된다.
    - 암호화 : /encrypt 
    - 복호화 : /decrypt
- 위 두 End-Point 에 대한 테스트를 수행한 결과는 다음과 같다.

```shell
// encrypt
curl -XPOST http://localhost:8888/encrypt -d 'sa'
e65286fd9c6f3a6e9ed6299ef1c7237bfa02af3577dd58049b4136526e529dc8

// decrypt
curl -XPOST http://localhost:8888/decrypt -d 'e65286fd9c6f3a6e9ed6299ef1c7237bfa02af3577dd58049b4136526e529dc8'
sa
```

## 설정 파일 암호화

`user-service.yml`

```yaml
spring:
  datasource:
    driver-class-name: org.h2.Driver
    url: jdbc:h2:mem:testdb
    username: sa
    password: '{cipher}2277f9e5690bc6f5d0336e81817bae692b06845507fff6d67a64e00e277495b1'
```
- 기존의 user-service 에서 사용하던 datasource 관련 설정을 Config-Server 를 통해 가져올 수 있도록, native/file-repo 혹은 git 에 저장한다.
- 여기서 주의할점은, 만약 **암호화된 정보** 를 사용한다면, prefix 로 {cipher} 라는 값을 넣어 암호화된 값이라는 것을 ConfigServer 에게 알려주어야 한다.
- 만약 {cipher} 라는 prefix 가 없다면, 암호화된 값이어도 plain-text 로 인지하여 복호화를 수행하지 않는다.

## UserService

`bootstrap.yml`

```yaml
spring:
  cloud:
    config:
      uri: http://127.0.0.1:8888
      name: user-service # name 에 명시한 yml 파일을 fetch 해 온다.
```
- UserService 가 기동할때 spring-cloud-config 서버를 통해 user-service.yml 을 가져올 수 있도록 설정을 추가해준다.