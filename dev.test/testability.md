# 의존성

### 의존성이란 ?

- Dependency는 (컴퓨터 공학에서) Coupling과 동일한 의미로 쓰인다.
- 의존성이란, a state in which one object uses a funtion of another object.

### 의존성 주입

- 커플링을 약하게 만들기 위해 사용하는 대표적인 기법
- 외부에서 의존성을 주입해주는 방식이다.
- 의존성을 약화시킬 뿐, 의존성을 완전히 제거시키는 것은 아니다.
    - 객체지향 애플리케이션은 객체들 간 협력으로 동작한다.
    - 의존은 객체 간 협력이다. 의존을 완전히 제거하는 것은 객체 간 협력을 금지하는 것과 같다.
    - 따라서 우리는 객체 간 의존성을 없애는 것이 아니라 약화 시키는 데 집중해야 한다. (loose coupling)

### 의존성과 테스트

- 테스트를 잘 하려면 의존성 주입(DI)과 의존성 역전(DIP)를 잘 다룰 수 있어야 한다.
- 잘 조합해서 사용하면 테스트 하기 좋은 코드로 만들 수 있다.

# Testability

### Testability 란?

- "테스트 가능성"
- 얼마나 쉽게 input을 변경하고, output을 쉽게 검증할 수 있는가?

### 쉽게 input을 변경하고 -> 가 어려운 케이스

1. 감춰진 의존성
   - 내부 구현을 뜯어봐야 테스트를 할 수 있다면, 테스트가 보내는 개선의 신호이다.
   - 어떤 것을 의존하는 지 알 수 있다면 캡슐화가 크게 의미가 없어진다.
2. 억지로 테스트
   - static method는 mockito를 사용하더라도 stubbing이 안된다.
   - 하지만 반드시 static method를 stubbing 해야 한다면 PowerMockito를 사용하면 된다.
   - 하지만 만약 해당 static method가 구현된 class가 final class여서 stubbing이 불가능하다면?
     - 리플렉션까지 동원해서 억지로 억지로 테스트를 성공시켜야 한다.
   - 이는 좋은 구조가 아님을 암시한다.
3. 하드 코딩이 된 경우
   - 하드 코딩된 값에 지나치게 의존하게 된다.
4. 외부 시스템과 연동되는 부분이 하드 코딩된 경우
   - 외부 API를 호출할 때 RestTemplate이나 WebClient를 직접 사용하는 경우

### output을 쉽게 검증 -> 가 어려운 케이스

1. 결과가 감춰졌을 때
   - 메서드가 어디론가 결과를 내보내고 있다. (console 출력, 외부 API 호출)
   - 하지만 return type이 void 라면?
   - 메서드 실행 결과를 외부에서 확인할 방법이 없으면 좋은 구조가 아니다.

