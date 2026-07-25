# CORS (Cross-Origin Resource Sharing)

> 브라우저에서 다른 출처의 서버로 요청할 때 해당 응답에 접근할 수 있는지 제어하는 방식

정확히는 서버가 응답 헤더를 통해 허용 여부를 알려주고, 브라우저가 이를 확인하여 응답 접근을 허용하거나 차단하는 구조이다.

CORS는 브라우저가 적용하는 정책이므로 서버 간 요청이나 Postman, `curl`에는 적용되지 않는다.

## SOP (Same-Origin Policy)

CORS를 알기 전에 SOP를 먼저 알아야 한다.

브라우저가 한 Origin에서 실행된 JavaScript가 다른 Origin의 데이터를 마음대로 읽지 못하도록 제한하는 보안 정책이다.

악성 사이트의 JavaScript가 은행 사이트에서 사용자의 개인정보를 읽을 수 있는 것은 문제이다. SOP가 이를 차단한다.

```text
evil.com의 JavaScript
    ↓
bank.com의 계좌 정보 요청
    ↓
SOP가 응답 접근 차단
```

SOP는 기본적으로 Cross-Origin 접근을 제한한다. 그러나 실제 서비스에서는 프론트엔드와 API 서버를 서로 다른 Origin으로 운영하는 경우가 많다.

따라서 신뢰할 수 있는 Origin에 한해서 접근을 허용할 방법이 필요하다.

## Origin

Origin은 스킴, 호스트, 포트의 조합이다.

예를 들어 다음 주소의 Origin은 아래와 같다.

```text
https://example.com:443/users
```

```text
스킴: https
호스트: example.com
포트: 443
```

세 요소 중 하나라도 다르면 서로 다른 Origin이다.

같은 Origin

```text
https://example.com
```

다른 Origin

```text
http://example.com
https://api.example.com
https://example.com:8080
```

## 언제 발생하나?

```text
프론트엔드: http://localhost:3000
백엔드 API: http://localhost:8080
```

프론트엔드의 JavaScript가 백엔드 API를 호출한다.

```js
fetch("http://localhost:8080/users");
```

두 주소는 포트가 다르므로 서로 다른 Origin이다.

SOP에 의해 **브라우저**는 기본적으로 응답 접근을 차단한다. 이때 CORS 에러가 발생한다.

## CORS

> SOP에 의해 제한되는 Cross-Origin 요청을 서버가 선택적으로 허용하는 방법

예를 들어 서버가 다음 응답 헤더를 반환할 수 있다.

```http
Access-Control-Allow-Origin: http://localhost:3000
```

이는 `http://localhost:3000`에서 시작된 요청에 브라우저가 응답 접근을 허용할 수 있다는 의미이다.

전체 흐름은 다음과 같다.

```text
localhost:3000에서 API 요청
    ↓
localhost:8080 서버가 응답
    ↓
Access-Control-Allow-Origin 확인
    ↓
허용된 Origin이면 브라우저가 응답 접근 허용
```

단순 요청은 실제 서버로 전송된다. 브라우저는 응답의 `Access-Control-Allow-Origin` 헤더를 확인한 뒤 JavaScript가 응답에 접근할 수 있는지 결정한다.

Preflight가 필요한 요청은 사전 요청이 실패하면 실제 요청을 전송하지 않는다.

## Preflight 요청

일부 Cross-Origin 요청은 브라우저가 실제 요청을 보내기 전에 서버에 허용 여부를 먼저 확인한다.

이를 Preflight 요청이라고 하며, 브라우저가 `OPTIONS` 메서드를 사용해 자동으로 전송한다.

1. 브라우저가 `OPTIONS` 요청 전송

    ```http
    OPTIONS /users/1
    Origin: http://localhost:3000
    Access-Control-Request-Method: DELETE
    Access-Control-Request-Headers: Authorization
    ```

2. 서버가 허용할 Origin, 메서드, 헤더 응답

    ```http
    Access-Control-Allow-Origin: http://localhost:3000
    Access-Control-Allow-Methods: GET, POST, DELETE
    Access-Control-Allow-Headers: Authorization
    ```

3. 브라우저가 허용 여부 확인
4. 허용되면 실제 요청 전송


메서드가 `GET`, `HEAD`, `POST`라면 Preflight를 하지 않을 수 있다. 

다음과 같은 요청에는 Preflight가 필요하다.

- `PUT`, `PATCH`, `DELETE` 사용
- `Authorization`처럼 CORS-safelisted request header가 아닌 헤더 사용
- `Content-Type`이 `application/json`
