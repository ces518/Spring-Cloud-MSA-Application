# Spring Cloud Config

- 분산 시스템에서 서버/클라이언트 서버 구성 정보를 외부 시스템에서 관리
- 하나의 중앙화된 저장소에서 구성 요소 관리 가능
- 각 서비스를 빌드/배포 없이 바로 적용 가능
- CI/CD 를 통해 DEV - UAT - PROD 환경에 맞는 구성정보 사용

## 지원 저장소

- Private Git Repository
- Secure Vault
- Secure File Storage

## 우선 순위

- application.yml
- application-{name (service name)}.yml
- application-{name (service name)}-<profile>.yml

## Config Server

```xml

<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-server</artifactId>
</dependency>
```

```yaml
server:
  port: 8888 # Config Server Default Port : 8888

spring:
  application:
    name: config-service
  cloud:
    config:
      server:
        git:
          uri: file:///Users/kakaocommerce/work/git-local-repo
```

- config 서버로 git 을 사용
- local git repository 를 사용하기 때문에 spring.cloud.config.server.git.uri 에 해당 경로를 설정한다.

```java

@SpringBootApplication
@EnableConfigServer // Config Server 활성화
public class ConfigServerApplication {

	public static void main(String[] args) {
		SpringApplication.run(ConfigServerApplication.class, args);
	}

}
```

- git.uri 에 설정한 경로에 ecommerce.yml 파일을 생성한다
    - 아래 설정 파일의 내용은 마이크로서비스에서 공통으로 사용할 설정 정보들이다

```yaml
token:
  expiration_time: 864000000
  secret: user_token

gateway:
  ip: 192.168.0.15
```

`Config Server 를 통해 설정 정보 취득`

```shell
 curl http://localhost:8888/ecommerce/default
{"name":"ecommerce","profiles":["default"],"label":null,"version":"bf4e962bc8d4e09614c6160c51cb116873ea9107","state":null,"propertySources":[{"name":"file:///Users/kakaocommerce/work/git-local-repo/ecommerce.yml","source":{"token.expiration_time":864000000,"token.secret":"user_token","gateway.ip":"192.168.0.15"}}]}%
```