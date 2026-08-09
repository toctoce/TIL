# HTTP vs HTTPS

> `HTTPS`는 `HTTP`에 `TLS`를 적용하여 통신 내용을 암호화하고 데이터의 무결성을 검증한다.

`HTTPS`는 `HTTP`에 `TLS`를 적용해 더 안전하다.

## HTTP

`HTTP(Hypertext Transfer Protocol)`는 네트워크를 통해 데이터를 주고받는 데 사용하는 프로토콜이다. 웹 사이트 콘텐츠와 API 호출 등 웹에서 이루어지는 통신에 사용된다.

### HTTP 요청 예시

```http
GET /hello.txt HTTP/1.1
User-Agent: curl/7.63.0 libcurl/7.63.0 OpenSSL/1.1.1 zlib/1.2.11
Host: www.example.com
Accept-Language: en
```

`HTTP`는 데이터를 암호화하지 않는다. 따라서 네트워크 경로에서 통신을 가로챈 공격자는 요청과 응답의 내용을 읽거나 변경할 수 있다.

## HTTPS

`HTTPS`의 `S`는 `Secure`를 의미한다. `HTTPS`는 `TLS`를 사용하여 `HTTP` 요청과 응답을 암호화한다.

네트워크 경로에서 통신을 가로채더라도 원래 내용을 확인하기 어렵다. 다음은 이해를 돕기 위해 텍스트로 표현한 암호문 예시이다.

```text
t8Fw6T8UV81p...
```

## TLS 암호화 과정

공개 키 암호화 방식을 사용하여 암호화된다.
TLS 인증서를 통해 공개 키가 클라이언트와 공유되고, 공개 키와 개인 키를 이용해 세션 키를 안전하게 생성한다. 이후 `HTTP` 요청과 응답을 이 세션 키로 암호화한다.

## REFERENCE
- 
- https://www.cloudflare.com/ko-kr/learning/ssl/why-is-http-not-secure/
