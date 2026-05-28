# TCP Header

TCP(Transmission Control Protocol)는 신뢰성 있는 전송 프로토콜이다.
왜냐하면 데이터를 보내기 전에 먼저 연결을 설정하고, TCP가 보낸 모든 데이터는 수신자로부터 확인 응답을 받기 때문이다.

TCP 헤더와 그 안의 여러 필드를 더 자세히 살펴보자.

![TCP 헤더 1](TCP%20%ED%97%A4%EB%8D%94%201.png)

## Source Port

출발지 포트는 **16비트 필드**이며, 송신자의 포트 번호를 나타낸다.

## Destination Port

목적지 포트는 **16비트 필드**이며, 수신자의 포트 번호를 나타낸다.

## Sequence Number

Sequence Number는 **32비트 필드**이며, TCP 세션 동안 얼마나 많은 데이터가 전송되었는지를 나타낸다.

새로운 TCP 연결을 설정할 때, 즉 3-way handshake를 할 때 초기 Sequence Number는 임의의 32비트 값으로 정해진다.

수신자는 이 Sequence Number를 사용하고, 그에 대한 확인 응답을 다시 보낸다.

## Acknowledgment Number

Acknowledgment Number는 **32비트 필드**이며, 수신자가 다음 TCP 세그먼트를 요청할 때 사용한다.

이 값은 보통 Sequence Number에 `1`을 더한 값이다.

## DO

DO는 **4비트 Data Offset 필드**이며, Header Length라고도 부른다.

이 필드는 TCP 헤더의 길이를 나타낸다.

이를 통해 실제 데이터가 어디서부터 시작되는지 알 수 있다.

> Options 필드로 인해 TCP 헤더의 길이는 고정되어 있지 않다.

## RSV

RSV는 **3비트 Reserved 필드**이다.

사용되지 않는 필드이며, 항상 `0`으로 설정된다.

## Flags

Flags는 **9비트 필드**이며, Control Bits라고도 부른다.

TCP는 이 플래그들을 사용해서 **연결을 설정**하고, **데이터를 전송**하고, **연결을 종료**한다.

1. URG : Urgent Pointer로, 이 비트가 설정되면 해당 데이터는 다른 데이터보다 우선적으로 처리되어야 한다.

2. ACK : 상대방이 보낸 데이터를 잘 받았음을 알려줄 때 사용한다.

3. PSH : Push 기능을 의미한다.
   - TCP 세그먼트가 가득 찰 때까지 기다리지 않고 애플리케이션에게 바로 데이터를 전달하라는 의미이다.

4. RST : 연결을 초기화하거나 강제로 종료할 때 사용된다.
    - RST를 받으면 **즉시 연결을 종료**해야 한다.
    - 복구할 수 없는 오류가 있을 때 주로 사용되며, TCP 연결을 정상적으로 종료하는 방식은 아니다.

5. SYN : TCP 연결을 설정할 때 사용된다.(3-way handshake)
    - 초기 Sequence Number를 설정할 때 사용된다.

6. FIN : TCP 연결을 종료할 때 사용된다.(4-way handshake)
    - TCP는 전이중 통신, 즉 양방향 통신이기 때문에 양쪽 모두 FIN을 사용해야 연결이 정상적으로 종료된다.

## Window

Window는 **16비트 필드**이며, 수신자가 받을 의사가 있는 바이트 수를 나타낸다.

이 필드는 수신자가 송신자에게 현재 받고 있는 양보다 더 많은 데이터를 받을 수 있는지 알려주는 데 사용된다.

이를 위해 ACK 필드의 Sequence Number 이후로 몇 바이트를 더 받을 수 있는지를 지정한다.

## Checksum

Checksum은 **16비트 필드**이며, TCP 헤더가 정상인지 확인하는 데 사용된다.

즉, 전송 중 TCP 헤더나 데이터에 오류가 생겼는지 확인하기 위한 값이다.

## Urgent Pointer

Urgent Pointer는 **16비트 필드**이며, URG 비트가 설정되었을 때 사용된다.

긴급 데이터가 어디에서 끝나는지를 나타낸다.

## Options

Options 필드는 선택적인 필드이다.

길이는 `0비트`부터 `320비트`까지 가능하다.


## REFERENCE
- https://networklessons.com/ip-routing/tcp-header
