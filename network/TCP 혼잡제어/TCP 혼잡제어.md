# TCP 혼잡제어

> 네트워크 혼잡으로 인한 패킷 손실과 지연을 줄이기 위해 송신자의 전송 속도를 조절하는 기법

네트워크가 처리할 수 있는 양보다 많은 데이터가 한 번에 전송되면 패킷 손실과 지연이 발생할 수 있다.

TCP는 손실이나 지연을 통해 네트워크 혼잡을 감지하고, 혼잡 윈도우를 조절해 전송 속도를 낮추거나 높인다.

- 혼잡 윈도우 : 네트워크 혼잡 상태에 따라 송신자가 한 번에 보낼 수 있는 데이터 양
- `cwnd` : 혼잡 윈도우 크기
- `ssthresh` : `Slow Start`에서 `Congestion Avoidance`로 넘어가는 기준값

## 혼잡 제어의 기본 개념

### AIMD

> `Additive Increase Multiplicative Decrease`, 증가할 때는 조금씩 늘리고 감소할 때는 크게 줄이는 방식

![TCP 혼잡제어 1](<TCP 혼잡제어 1.png>)

패킷이 정상적으로 도착하면 `cwnd`를 조금씩 증가시킨다. 반대로 패킷 손실이나 `Timeout`이 발생하면 혼잡이 발생했다고 판단하고 `cwnd`를 절반으로 줄인다.

`AIMD`는 여러 송신자가 네트워크를 비교적 공평하게 사용할 수 있도록 만든다. 다만 `cwnd`를 하나씩 증가시키기 때문에 초기에 네트워크 대역폭을 충분히 활용하기까지 시간이 오래 걸릴 수 있다.

### Slow Start

> 처음에는 `cwnd`를 빠르게 증가시키다가 일정 기준 이후에는 증가 속도를 줄이는 방식

![TCP 혼잡제어 2](<TCP 혼잡제어 2.png>)

`Slow Start`는 처음에 `cwnd`를 작게 시작한다. 이후 `ACK`를 받을 때마다 `cwnd`를 증가시켜 전송량을 빠르게 늘린다.

`cwnd`가 `ssthresh`에 도달하면 더 이상 빠르게 증가시키지 않고, `Congestion Avoidance` 단계로 넘어간다.

## 혼잡 제어 정책

### 1. TCP Tahoe

> `Slow Start`, `Congestion Avoidance`, `Fast Retransmit`을 사용하는 초기 혼잡 제어 정책

![TCP 혼잡제어 3](<TCP 혼잡제어 3.png>)

- 처음에는 `Slow Start` 방식으로 `cwnd`를 증가시킨다.
- `cwnd`가 `ssthresh`에 도달하면 `AIMD` 방식으로 증가시킨다.
- `Timeout`이나 `3 Duplicate ACK`가 발생하면 `ssthresh`를 현재 `cwnd`의 절반으로 줄이고, `cwnd`를 1로 줄인다.

`TCP Tahoe`는 손실이 발생하면 항상 `cwnd`를 1로 줄인다. 그래서 혼잡 이후 다시 전송 속도를 높이는 데 시간이 오래 걸릴 수 있다.

### 2. TCP Reno

> `TCP Tahoe`에 `Fast Recovery`를 추가한 혼잡 제어 정책

![TCP 혼잡제어 4](<TCP 혼잡제어 4.png>)

`TCP Reno`는 `Timeout`과 `3 Duplicate ACK`를 다르게 처리한다.

- `Timeout`이 발생하면 `ssthresh`를 현재 `cwnd`의 절반으로 줄이고, `cwnd`를 1로 줄인다.(`TCP Tahoe`와 동일)
- `3 Duplicate ACK`가 발생하면 `ssthresh`를 현재 `cwnd`의 절반으로 줄이고, `cwnd`를 1이 아닌 절반 수준으로 줄인다.

`3 Duplicate ACK`는 일부 데이터는 계속 도착하고 있다는 의미이기 때문에 `Timeout`보다 덜 심한 혼잡으로 본다.

### 3. TCP CUBIC

> 고속 네트워크 환경에서 더 효율적으로 동작하도록 설계된 혼잡 제어 알고리즘

위 알고리즘들은 큰 대역폭을 충분히 활용하는 데 시간이 오래 걸릴 수 있다. 또한 `RTT`를 기준으로 `cwnd`를 증가시키기 때문에 장거리 통신에서 효율이 낮을 수 있다.

이러한 문제를 해결하고 고속 네트워크 환경에 최적화된 혼잡 제어 알고리즘이 `TCP CUBIC`이다.

`TCP CUBIC`은 Linux에서 기본 혼잡 제어 알고리즘으로 사용된다.

`TCP CUBIC`은 `TCP Reno`와 대부분 비슷하지만, `Congestion Avoidance` 단계에서 3차 함수를 사용한다.

![TCP 혼잡제어 5](<TCP 혼잡제어 5.png>)

- 손실이 마지막으로 감지되었을 때의 `cwnd` 크기를 `Wmax`로 설정한다.
- `cwnd`가 다시 `Wmax`에 도달할 것으로 예상되는 시점을 `K`라고 한다.
- 현재 시점이 `K`와 멀면 `cwnd`를 크게 증가시킨다.
- 현재 시점이 `K`에 가까우면 `cwnd`를 작게 증가시킨다.

즉, 손실 전의 혼잡 윈도우 크기인 `Wmax`에 가까워지도록 `cwnd`를 빠르게 증가시키고, `Wmax`에 가까워지면 대역폭을 조심스럽게 탐색한다.

## REFERENCE

- [Network TCP 혼잡제어](https://velog.io/@nnnyeong/Network-TCP-%ED%98%BC%EC%9E%A1%EC%A0%9C%EC%96%B4-Congestion-Control)
- [TCP 혼잡제어와 원리](https://velog.io/@young_gyu/TCP-%ED%98%BC%EC%9E%A1%EC%A0%9C%EC%96%B4%EC%99%80-%EC%9B%90%EB%A6%AC%ED%83%80%ED%98%B8-%EB%A6%AC%EB%85%B8-%ED%81%90%EB%B9%85)
