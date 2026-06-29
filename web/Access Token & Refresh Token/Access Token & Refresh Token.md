# Access Token & Refresh Token

> Access Token: API 요청에 사용하는 인증 토큰
> 
> Refresh Token: Access Token을 다시 발급받기 위한 토큰

Access Token만 사용하면 구조는 단순하지만, 만료될 때마다 다시 로그인해야 한다.
Refresh Token을 함께 사용하면 Access Token의 만료 시간을 짧게 가져가면서도 로그인 상태를 유지할 수 있다.

| 항목 | Access Token | Refresh Token |
| --- | --- | --- |
| 목적 | 리소스 접근 권한 증명 | Access Token 재발급 |
| 만료 시간 | 짧음 | 긺 |
| 사용 위치 | API 요청 | 토큰 재발급 요청 |
| 저장 위치 | 메모리, 쿠키 등 | 서버 저장소 또는 `HttpOnly` 쿠키 |
| 노출 시 위험도 | 높음 | 매우 높음 |
| 서버 저장 필요 여부 | 보통 필요 없음 | 필요할 수 있음 |

## Access Token

Access Token은 사용자가 인증된 상태임을 증명하는 토큰이다.

클라이언트는 API를 요청할 때 Access Token을 `Authorization` 헤더에 담아 서버로 보낸다.

```text
GET /profile HTTP/1.1
Authorization: Bearer <AccessToken>
```

Access Token이 `JWT`라면 서버는 토큰의 서명, 만료 시간, 권한 등을 확인하고 요청을 처리한다.
Access Token은 보통 만료 시간을 짧게 둔다. 탈취되더라도 사용할 수 있는 시간을 줄이기 위해서다.

## Refresh Token

Refresh Token은 Access Token이 만료되었을 때 새로운 Access Token을 발급받기 위해 사용하는 토큰이다.

Refresh Token은 Access Token보다 유효 기간이 길다. 그만큼 탈취되면 위험하기 때문에 서버 저장소에 저장하거나 `HttpOnly` 쿠키를 사용하는 방식이 자주 사용된다.

> HttpOnly 쿠키는 JavaScript로 읽을 수 없게 막은 쿠키이며, 토큰 탈취 위험을 줄이기 위해 사용한다.

Refresh Token은 필수는 아니지만, 로그인 상태를 오래 유지해야 하는 서비스에서 자주 사용된다.

## 인증 흐름

1. **로그인**
   - 로그인 성공 시 서버가 Access Token과 Refresh Token을 발급한다.

2. **API 요청**
   - 클라이언트는 Access Token을 `Authorization` 헤더에 담아 API를 요청한다.

3. **Access Token 만료**
   - Access Token이 만료되면 서버는 `401 Unauthorized`를 반환한다.

4. **토큰 재발급 요청**
   - 클라이언트는 Refresh Token을 이용해 새로운 Access Token 발급을 요청한다.

5. **새 Access Token 발급**
   - 서버가 Refresh Token을 검증한 후 새로운 Access Token을 발급한다.
   - 클라이언트는 새 Access Token으로 다시 API를 요청한다.

6. **Refresh Token 만료**
   - Refresh Token도 만료되면 서버는 `401 Unauthorized`를 반환한다.
   - 사용자는 다시 로그인하여 새로운 토큰을 발급받아야 한다.

7. **로그아웃**
   - Refresh Token을 삭제하거나 무효화해 더 이상 Access Token을 재발급받지 못하게 한다.
