# Spring Cloud Netflix Eureka

## Service Discovery
- Spring Cloud Netflix Eureka 에서 제공해주는 기능
- 서비스별 각 인스턴스를 등록해두고, 등록된 이름을 통해 서버에 접근하는 방식
    - 외부에서 마이크로서비스들이 다른 서비스들을 검색하기 위해 사용함
    - 일종의 전화번호부 같은 개념

> API Gateway 에 요청 정보를 전달 -> Service Discovery 가 적절한 서비스 정보 제공


## Eureka Server Config

```groovy
plugins {
    id 'org.springframework.boot' version '2.5.1'
    id 'io.spring.dependency-management' version '1.0.11.RELEASE'
    id 'java'
}

group = 'me.june'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '11'

configurations {
    compileOnly {
        extendsFrom annotationProcessor
    }
}

repositories {
    mavenCentral()
}

ext {
    set('springCloudVersion', "2020.0.3")
}

dependencies {
    implementation 'org.springframework.cloud:spring-cloud-starter-netflix-eureka-server'
    compileOnly 'org.projectlombok:lombok'
    developmentOnly 'org.springframework.boot:spring-boot-devtools'
    annotationProcessor 'org.projectlombok:lombok'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}

dependencyManagement {
    imports {
        mavenBom "org.springframework.cloud:spring-cloud-dependencies:${springCloudVersion}"
    }
}

test {
    useJUnitPlatform()
}

```

```yaml
# Eureka Server Port
server:
  port: 8761

# Eureka Server App Name
spring:
  application:
    name: discoveryservice

# Eureka Lib 가 포함되어 있는경우 기본적으로 Eureka Client 로 동작한다.
# 현재는 Eureka Server 이기 때문에 불필요하므로 비활성화 시킴 (기본은 true)
eureka:
  client:
    register-with-eureka: false
    fetch-registry: false
```