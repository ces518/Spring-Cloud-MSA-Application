# 대칭키와 비대칭키

- Symmetric Encryption (대칭키)
    - Using Same Key
- Asymmetric Encryption (비대칭키 / RSA Key Pair)
    - Private and Public Key
    - Using Java keyTool

## Flow
- 각 저장소에 암호화한 데이터를 저장해두고, Spring Cloud Config Server 가 이를 복호화해서 각 마이크로서비스에 전파한다.

## JCE (Java Cryptography Extension)
- Java8 이상인 경우 Key Size 오류, 알고리즘 오류 등이 발생할 수 있다.
    - 제한 없는 암호화 정책 파일로 교체 필요
    - https://www.oracle.com/java/technologies/javase-jce-all-downloads.html
- 11 버전 이상인 경우 문제가 없다.
