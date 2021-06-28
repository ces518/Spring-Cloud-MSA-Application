# 비대칭키 암호화

## 비대칭 암호화
- 비대칭 암호화는 Enc/Dec 에 각각 서로 다른 키를 사용한다.
- Public, Private Key 를 생성해야 한다. -> JDK keytool 을 사용
- 암호화시 반드시 Public, 복호화시 반드시 Private 을 사용해야 하는것은 아니지만, 일반적으로 암호화시 Public Key, 복호화시 Private Key 를 사용한다.

## Keytool 을 활용한 생성

`key 생성`

```shell
$ keytool -genkeypair -alias apiEncryptionKey -keyalg RSA \
 -dname "CN=ncucu, OU=API Development, O=ncucu.me, L=Seoul, C=KR" \
 -keypass "1q2w3e4r" -keystore apiEncryptionKey.jks -storepass "1q2w3e4r"
```

`key 확인`

```shell
keytool -list -keystore apiEncryptionKey.jks -v
키 저장소 비밀번호 입력:
키 저장소 유형: PKCS12
키 저장소 제공자: SUN

키 저장소에 1개의 항목이 포함되어 있습니다.

별칭 이름: apiencryptionkey
생성 날짜: 2021. 6. 28.
항목 유형: PrivateKeyEntry
인증서 체인 길이: 1
인증서[1]:
소유자: CN=ncucu, OU=API Development, O=ncucu.me, L=Seoul, C=KR
발행자: CN=ncucu, OU=API Development, O=ncucu.me, L=Seoul, C=KR
일련 번호: 4d91ac58
적합한 시작 날짜: Mon Jun 28 21:44:46 KST 2021 종료 날짜: Sun Sep 26 21:44:46 KST 2021
인증서 지문:
	 SHA1: 8D:5D:B5:98:00:CD:DC:3F:11:0B:80:6A:D4:08:CF:E4:51:2E:7C:30
	 SHA256: 07:E9:D2:FD:FC:0D:DF:61:25:E9:CF:48:2D:F4:2E:E0:FB:72:58:5B:FC:53:AE:E1:AB:F4:A4:F1:F0:89:49:78
서명 알고리즘 이름: SHA256withRSA
주체 공용 키 알고리즘: 2048비트 RSA 키
버전: 3

확장:

#1: ObjectId: 2.5.29.14 Criticality=false
SubjectKeyIdentifier [
KeyIdentifier [
0000: 18 7C EB FC 06 B8 50 9A   26 40 61 69 6D E3 8A BE  ......P.&@aim...
0010: 4C 9D AA 22                                        L.."
]
]



*******************************************
*******************************************
```

`keytool 을 사용해 인증서파일 생성`

```shell
keytool -export -alias apiEncryptionKey -keystore apiEncryptionKey.jks -rfc -file trustServer.cer
키 저장소 비밀번호 입력:
인증서가 <trustServer.cer> 파일에 저장되었습니다.
```

`keytool 을 사용해 public key import 하기`

```shell
keytool -import --alias trustServer -file trustServer.cer -keystore publicKey.jks
키 저장소 비밀번호 입력:
새 비밀번호 다시 입력:
소유자: CN=ncucu, OU=API Development, O=ncucu.me, L=Seoul, C=KR
발행자: CN=ncucu, OU=API Development, O=ncucu.me, L=Seoul, C=KR
일련 번호: 4d91ac58
적합한 시작 날짜: Mon Jun 28 21:44:46 KST 2021 종료 날짜: Sun Sep 26 21:44:46 KST 2021
인증서 지문:
	 SHA1: 8D:5D:B5:98:00:CD:DC:3F:11:0B:80:6A:D4:08:CF:E4:51:2E:7C:30
	 SHA256: 07:E9:D2:FD:FC:0D:DF:61:25:E9:CF:48:2D:F4:2E:E0:FB:72:58:5B:FC:53:AE:E1:AB:F4:A4:F1:F0:89:49:78
서명 알고리즘 이름: SHA256withRSA
주체 공용 키 알고리즘: 2048비트 RSA 키
버전: 3

확장:

#1: ObjectId: 2.5.29.14 Criticality=false
SubjectKeyIdentifier [
KeyIdentifier [
0000: 18 7C EB FC 06 B8 50 9A   26 40 61 69 6D E3 8A BE  ......P.&@aim...
0010: 4C 9D AA 22                                        L.."
]
]
```

## Config Server 설정 변경

`bootstrap.yml`
```yaml
encrypt:
#  key: abcdedfhijk1234 # encrypt key 추가, 해당 키가 존재해야 enc / dec 작업 수행이 가능함
  key-store:
    location: file://${user.home}/work/keystore/apiEncryptionKey.jks
    password: 1q2w3e4r
    alias: apiEncryptionKey
```
- 기존에 대칭키를 사용하던 key 프로퍼티를 제거하고, 앞서 생성한 jks 파일을 key-store 로 설정한다.