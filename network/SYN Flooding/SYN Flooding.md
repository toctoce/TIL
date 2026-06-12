# SYN Flooding

## 정의
TCP 3 way handshake를 할 때 큰 흐름은 아래와 같다.

1. Client가 Server에게 SYN 패킷을 전송한다.
2. Server는 Client가 보낸 SYN에 대한 ACK와, 자신의 SYN 패킷을 함께 전송한다.
3. Client는 Server가 보낸 SYN에 대한 ACK를 전송한다.

2번 단계가 실행되고 난 후, Server는 Client의 ACK를 기다리는 SYN_RECEIVED(=half-open) 상태가 된다.
그리고 이 상태로 일정 시간 안에 ACK 응답이 오지 않으면 해당 연결 정보를 제거하게 되는데, 제거되기 전까지 이 연결 정보는 SYN Backlog에 계속 쌓이게 된다.

때문에 공격자가 SYN 패킷만 계속 서버로 보내고 SYN + ACK 패킷에 응답하지 않으면 결국 SYN Backlog가 가득 차게 되고, 정상적인 요청이 들어왔을 때 새로운 연결을 처리하지 못하면서 DoS 상태를 유발한다.

## 해결 방법
### 1. 방화벽 또는 DDoS 장비 설정
방화벽이나 DDoS 장비를 통해 동일한 IP 주소의 SYN 요청에 대한 임계치를 설정하고 임계치를 초과하는 경우 해당 연결 요청을 차단하는 방법이 있다.
하지만 봇넷을 통해 스푸핑된 IP로 들어오는 분산된 공격을 막기는 어렵다.

### 2. SYN Cookies 설정
SYN Cookies 설정은 클라이언트의 유효성을 확인하는 방법이다.

서버에 SYN 패킷이 도착했을 때, IP, 포트, MSS, 시간 정보 등의 값을 가지고 단방향 암호화 해시 함수를 사용해 쿠키 값을 만든다.
그리고 이 쿠키 값을 서버의 ISN(Initial Sequence Number)에 담아 SYN + ACK 패킷과 함께 클라이언트에게 보낸다.
쿠키가 생성될 때는 SYN 패킷들을 SYN Backlog에 저장하지 않는다.
그리고 클라이언트로부터 돌아오는 ACK 번호를 이용해 쿠키 값을 검증하고 유효한 경우에만 연결을 맺게 된다.

SYN Cookies 설정은 다음 명령어를 통해 활성화할 수 있다.
```bash
sysctl -w net.ipv4.tcp_syncookies=1
```
설정을 활성화하더라도 SYN Flooding에 의해 SYN Backlog가 가득 찼을 때만 해당 기능이 실제로 동작한다.

## REFERENCE
- https://wildeveloperetrain.tistory.com/393#google_vignette
