# TLS Handshake

> `TLS Handshake`는 클라이언트와 서버가 암호화 방식과 키를 결정하고 서버의 신원을 확인하는 과정이다.

`TLS`는 안전한 인터넷 통신을 위한 암호화 및 인증 프로토콜이다. `TLS Handshake`가 완료되면 클라이언트와 서버는 생성한 세션 키로 데이터를 암호화하여 통신한다.

## 발생 시점

`TLS Handshake`는 새로운 `TLS` 연결을 생성할 때 발생한다. 기존 연결을 재사용하면 요청마다 다시 수행하지 않는다.

`TCP` 기반 통신에서는 `TCP Handshake`로 연결을 생성한 뒤 `TLS Handshake`를 수행한다.

## 수행 작업

`TLS Handshake` 과정에서 클라이언트와 서버는 다음 작업을 수행한다.

- 사용할 `TLS` 버전을 결정한다.
- 사용할 암호 제품군을 결정한다.
- 인증 기관(`CA`)의 디지털 서명이 포함된 서버 인증서를 검증하여 서버의 신원을 확인한다.
- 대칭 암호화에 사용할 세션 키를 생성한다.

## TLS 1.2 RSA 핸드셰이크 과정

다음은 `TLS 1.2`에서 `RSA` 키 교환 방식을 사용하는 과정이다.

![TLS 1.2 RSA 핸드셰이크 과정](<TLS Handshake 1.png>)

1. `ClientHello`
   - 클라이언트가 서버에 `ClientHello` 메시지를 보내 핸드셰이크를 시작한다.
   - 클라이언트가 지원하는 `TLS` 버전, 암호 제품군, 무작위 바이트 문자열인 `Client Random`을 포함한다.
2. `ServerHello`
   - 서버가 선택한 `TLS` 버전과 암호 제품군, `Server Random`을 `ServerHello` 메시지에 담아 전송한다.
   - 서버 인증서는 별도의 `Certificate` 메시지로 전송한다.
3. 서버 인증서 검증
   - 클라이언트가 인증 기관을 통해 서버 인증서를 검증하고, 인증서에 명시된 도메인과 접속한 도메인이 일치하는지 확인한다.
4. `Pre-Master Secret` 전송
   - 클라이언트가 무작위 바이트 문자열인 `Pre-Master Secret`을 생성한다.
   - 서버 인증서에서 얻은 공개 키로 암호화한 뒤 `ClientKeyExchange` 메시지로 전송한다.
5. `Pre-Master Secret` 복호화
   - 서버가 개인 키로 `Pre-Master Secret`을 복호화한다.
6. 세션 키 생성
   - 클라이언트와 서버가 `Client Random`, `Server Random`, `Pre-Master Secret`을 이용해 동일한 세션 키를 생성한다.
7. 클라이언트 완료
   - 클라이언트가 `ChangeCipherSpec`과 암호화된 `Finished` 메시지를 전송한다.
8. 서버 완료
   - 서버가 `ChangeCipherSpec`과 암호화된 `Finished` 메시지를 전송한다.
9. 대칭 키 암호화 통신
   - 핸드셰이크가 완료되면 세션 키를 이용해 데이터를 암호화하여 통신한다.

이 과정이 완료되면 `TLS`로 보호된 통신을 시작한다. `HTTPS`에서는 암호화된 `HTTP` 요청과 응답을 주고받는다.

> TLS 1.3은 ChangeCipherSpec을 이용한 암호화 전환 과정이 사라지고 메시지 순서도 달라진다.

## REFERENCE

- https://www.cloudflare.com/ko-kr/learning/ssl/what-happens-in-a-tls-handshake/
