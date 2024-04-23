4월 우아한테크세미나 : ‘Java의 미래, Virtual Thread’
우아한 형제들 김태헌님


## 1. Virtual thread 소개

### 도입하게 된 계기

- 게이트웨이 구현해야 하는 상황. 안정성과 처리량에 대한 고민.
- 선택지는 2개
  - 코틀린 코루틴
  - 자바 프로젝트 룸(Virtual thread)

### 스레드 비교

**기존 자바 스레드**
- 기존 자바 스레드는 생성 비용이 커서 스레드 풀이라는 것도 존재한다.
- 차지하는 메모리도 크다. 최대 2MB
- OS에 의해 스케줄링 된다. OS와 통신하기 위해 System call이 발생한다. 따라서 오버헤드가 발생한다.

**버추얼 스레드**
- 스레드 생성 및 스케줄링 비용이 기존 스레드보다 저렴
  - 스레드 풀 개념이 없다. 필요할 때 만들고, 끝나면 소멸한다.
  - 메모리도 최대 수KB 이다.
  - JVM에서 스케줄링한다.
- 스레드 스케줄링을 통해 nio를 지원
- 기존 스레드를 상속하여 기존 스레드 코드와 호환 가능하다.

**테스트 결론**
- 아무 것도 안하는 스레드로 비교했을 때, 기존 스레드 대비 98.8% 빠름. (대부분 생성, 소멸에 걸리는 시간으로 추측)

### Non-Blocking IO

버추얼 스레드도 NIO를 지원한다.
- JVM 스레드 스케줄링
- Continuation 활용

### 스레드 상속

스레드 상속를 상속해서 기존 애플리케이션에 쉽게 호환/적용 가능하다.
- VirtualThread는 BaseVirtualThread를 상속받고,
- BaseVirtualThread는 Thread를 상속받는다.
- 리스코프 치환 원칙에 따라 Thread를 쓰는 모든 곳에서 VirtualThread를 사용할 수 있게 된다.

## 2. Virtual thread 동작 원리

### 일반 스레드 특징

- 플랫폼 스레드
- OS에 의해 스케줄링
- OS에 존재하는 커널 스레드와 1:1 매핑
- 작업 단위로 Runnable을 사용

유저 영역(JVM)의 스레드는 커널 영역(OS)의 스레드와 1:1 매핑된다. 두 영역 간 통신은 JNI를 사용한다.  

### 버추얼 스레드 특징

- 가상 스레드
- JVM에 의해 스케줄링
- JVM에 존재하는 캐리어 스레드와 1:N 매핑
- 작업 단위로 Continuation을 사용

JVM에 스케줄러가 추가됐다. 스케줄러가 버추얼 스레드를 스케줄링 한다.   
스케줄러는 Executer 타입이다. 별도 설정이 없으면 DEFAULT_SCHEDULER를 사용한다.
DEFAULT_SCHEDULER는 ForkJoinPool 타입이고 static 타입이다. 
- 즉, 버추얼 스레드는 동일한 스케줄러를 공유한다.
- ForkJoinPool 매커니즘으로 스케줄링 할 것을 알 수 있다.
- DEFAULT_SCHEDULER에서 워커 스레드(캐리어 스레드) 수는 프로세서 수만큼 설정된다.

ForkJoinPool
- WorkQueue에 태스트가 쌓이고,
- 워커 스레드는 차근차근 진행하고,
- 내 스레드의 WorkQueue가 비었다면 다른 WorkQueue에서 태스크를 훔쳐온다.

### Continuation 이란?
- 코틀린 코루틴에서 suspend 기능도 continuation 개념으로 구현되어 있다.
- Continuation은 함수처럼 실행 가능한 작업 흐름이고,
- 함수와는 달리 중단 가능해야 하며, 중단 지점에서 다시 시작하는 것이 가능하다.
- (이 부분은 기록하기 힘들어서 영상 다시 볼 것)

## 3. 기존 스레드 모델과 비교

- 톰캣은 Thread per Request 방식이다.
  - 톰캣에 스레드가 2개까지만 생성 가능하다고 했을 때,
  - 요청 3개가 들어오면 1개는 대기에 빠진다. 
  - 요청 2개 중 하나라도 응답을 내보내야 요청을 처리 할 수다.
- 버추얼 스레드
  - 요청을 버추얼 스레드 1개가 받는다.
  - 버추얼 스레드는 무한정 늘어날 수 있다.
  - 하지만 캐리어 스레드에 배정되어야 비로소 실행된다.

## 4. 성능 테스트

CPU Bound vs IO Bound

- I/O Bound 작업은 51% 정도 좋다.
- CPU Bound 작업은 2% 정도 나쁘다.
- 단, 일반 스레드를 사용하면 특정 vUser 수가 넘어갔을 때 장애가 발생했다.

WebFlux vs Virtual Thread

- 거의 2배 차이 (극한 상황에서)
- 특정 vUser 수 이상 WebFlux 처리량 급감. 컨텍스트 스위칭 비용이 원인

## 5. 주의사항

### Blocking carrier thread (pin)

- 캐리어 스레드를 block하면 virtual thread 활용 불가
  - syncronized를 사용하거나
  - parallelStream을 사용하거나
  - ReentrantLock 을 사용하면 개선 가능하다. (차근차근 반영 중)
- VMOption으로 예방 가능하다.

### No Pooling

- 풀을 만들면 안된다.
- 생성 비용이 저렴하기 때문이다.
- 사용할 때마다 생성해야 한다.
- 사용완료 후 GC를 기다린다.

### CPU Bound task

- 결국 Carrier thread 에서 동작하므로 성능 낭비
- Non-Blocking 의 장점을 활용하지 못함

### 경량 스레드

- 수백만개의 스레드 생성 컨셉
- ThreadLocal을 최대한 가볍게 유지해야 한다
- 쉽게 생성, 소멸을 할 수 있다.
- JDK 21에 도입된 ThreadLocal 대신 ScopedValue를 사용하면 좋다.
- 버추얼 스레드는 무한하기 때문에 충분한 성능 테스트를 통해 유한한 리소스와 조합을 잘 따져야 한다.

## 6. QnA

### 코루틴 vs 버추얼 스레드

- 패러다임이 다르다.
  - 버추얼 스레드는 스레드 단위로 스케줄링
  - 코루틴은 메서드 단위로 스케줄링
- 코틀린 코루틴이 조금 더 높은 동시성을 가짐

### WebFlux는 이제 역사 속으로 사라질까?

- 패러다임이 다른거라 상관없다.
- 함수형 프로그래밍이 적용된 시스템에서는 WebFlux가 적합한거다.

### MySQL ConnectorJ에서 사용하면 성능이 나빠질까?

- 지금 PR이 올라와있고 반영되면 충분히 사용 가능하다.
- https://bugs.mysql.com/bug.php?id=110512

