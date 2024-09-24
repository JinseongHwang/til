## NTP(Network Time Protocol)

## 궁금증

- 모든 서버는 어떻게 현재 시간을 알 수 있을까? 서버의 시간은 믿을만한가?
- Java의 LocalDateTime.now() 는 모든 인스턴스에서 동일한 값을 반환할까?

## 3줄요약

- 시간 오차는 생길 수 밖에 없고, 주기적으로 NTP 서버와 통신하며 시간을 동기화 하는 것이다.
- 언제든 오차가 생길 수 있다. (믿을 수 있는 건 없다.)
- 이를 이해하기 위해서는 NTP에 대해 알아야 한다.

## NTP란?

- NTP(Network Time Protocol)는 네트워크의 모든 디바이스에서 시간을 동기화하는 데 사용되는 프로토콜이다.
- 프로토콜 스펙은 [Network Time Protocol Version 4: Protocol and Algorithms Specification](https://datatracker.ietf.org/doc/html/rfc5905)에 있다.

## 맥북 시간은 믿을 만한가?

![image](https://github.com/user-attachments/assets/8f23f84c-200d-4610-9f76-9e368c793fe2)

- 설정에 들어가보면 time.apple.com 를 소스로 설정되어 있는 것을 볼 수 있다. 따로 설정하지 않아도 되는 기본 값이다.
- 이는 NTP 서버이기 때문에 HTTP로 접근은 불가능하다. (궁금해서 브라우저로 접속해봤다)

## time.apple.com 의 시간은 믿을 만한가?

- 원자 시계(atomic clock)로 구현되어 있다. 세슘133 원자를 활용한다.
- 300만년에 1초의 오차가 발생할 수 있지만, 이정도면 믿을만 하다고 한다.

## 어떻게 오차를 보정할까?

![image](https://miro.medium.com/v2/resize:fit:720/format:webp/1*rPx5dnTlKec_bQ5qPaAaUQ.png)

1. 클라이언트가 t1 시점에 요청을 보낸다. 요청은 t2 시점에 서버에 도착한다. 서버는 t3 시점에 응답을 보내고, t4 시점에 클라이언트에 도착한다.
2. Round Trip 중 네트워크 지연시간은 `δ = (t4 - t1) - (t3 - t2)`
3. 클라이언트가 응답을 받았을 때(t4) 서버의 시간은 `t3 + δ/2` 로 추정할 수 있다.
4. 클라이언트와 서버의 시간 오차(clock skew)는 `θ = t4 - (t3 + δ/2)` 이다. `클라이언트 시간 - θ = 서버 시간` 으로 나타낼 수 있다.
5. 따라서 θ 값을 구해서 시간 보정을 할 수 있다.

## Java에서는 어떻게 동작할까?

Java에서는 시간을 참조하는 방법으로 2가지를 살펴보자.

- System.currentTimeMills()
- System.nanoTime()

## System.currentTimeMills() 의 특징은 ?

- Unix Epoch(1970년 1월 1일 00:00:00)부터 현재까지 흐른 시간(ms) 값이다.
  - 참고로 1초는 1,000ms 이다.
- 현재 시간을 나타낼 수 있다.
- NTP에 의해 주기적으로 조정되는 시스템 시계에 의존한다. 따라서 상대적으로 정확도가 낮다.

## System.nanoTime() 의 특징은 ?

- 현재 시간(혹은 Unix Epoch)과 상관없이 경과한 시간을 측정하는 값(ns)이다. (보통은 컴퓨터가 부팅된 시간부터)
  - 참고로 1초는 1,000,000,000ns 이다.
- 어떤 연산이 얼마나 걸렸는지 정밀하게 측정할 때 사용한다.
- 시스템 시간이 바뀌어도 영향을 받지 않는다. 강조하지만 현재 시간과는 무관한 값이다.
- 실제로 나노초 단위로 동작하진 않지만 밀리초보다는 훨씬 정밀하다.

## LocalDateTime.now() 는 어떤걸 참조할까 ?

LocalDateTime.now() 의 구현은 간단하다.

![image](https://github.com/user-attachments/assets/909566ac-90d5-4ea8-a529-ad78f5c2e74a)

아무래도 Clock의 systemDefaultZone를 살펴봐야 할 것 같다.

![image](https://github.com/user-attachments/assets/f7a1bb6c-dfba-4e10-aa53-4753870b0a19)

역시 간단하다. Clock의 구현체인 SystemClock을 살펴보자.

![image](https://github.com/user-attachments/assets/f577aef4-ae8e-4cf0-944d-037c39c58fb7)

찾았다! currentTimeMills() 를 사용한다. **즉, 시간 보정이 발생한다는 것이다.**
사실 LocalDateTime.now() 에는 주석을 자르지 않고 스샷을 넣었는데 시스템 시계를 사용한다고 친절히 나와있긴 하다 ㅎㅎ
