# 테스트를 이해할 때 알아두면 좋은 용어들

## SUT

- System under test의 약자이다.
- 테스트 하려는 대상이다.

## BDD

- Behavior Driven Development의 약자이다.
- TDD에 애자일, DDD 등을 얹은 개념이다.
- "어디에 어떻게 테스트를 넣어야 하지?"라는 질문에 "행동에 집중해라"라고 강조하는 방식이다.
  - User story와 시나리오를 강조한다.
- Given, When, Then 패턴을 사용하라고 권유한다.
  - 3A 방식이라고도 불린다. (Arrange, Act, Assert)

## 상호 작용 테스트 (Interaction test)

- 메서드가 실제로 실행됐는지 검증하는 테스트이다.
- 이 방식이 좋은지는 모르겠다. 메서드의 동작을 감시해야 하기 때문이다. 캡슐화에 위배된다.

## 상태 검증 vs 행위 검증

상태 검증
- SUT 행위에 input을 넣어서, output과 기댓값을 비교하는 방식이다.
- JUnit의 assertEqual, Kotest의 shouldBe 등..

행위 검증
- SUT 행위에 input을 넣었을 때, 협력 객체가 어떤 행동을 하는지를 검증하는 방식이다.
- JUnit, Kotest의 verify 등..

## Test fixture

- 테스트에 필요한 자원을 미리 생성해둔 것이다.
- 단, 테스트가 한 눈에 잘 안들어와서 코드 중복이 정말 심한 경우가 아니라면 사용을 자제하자.

## 비욘세 규칙

- 비욘세의 Single lady에, "If you liked it then you should have put a ring on it"이라는 가사가 있다.
  - 나를 좋아했다면, 알아서 반지를 줬어야지 -> 라는 뜻이다.
- 지키고 싶은 규칙/정책이 있다면, 관련된 테스트를 작성하라는 의미를 "비욘세 규칙"이라고 이름 붙였다.
  - "구글 엔지니어는 이렇게 일한다"에서 나온 내용인데 좀 구리다.

## Testability

- 테스트 가능성, 소프트웨어가 테스트 가능한 구조인가?

# Test double

## 정의

- 테스트 대역 (테스트를 위한 스턴트맨)
- ex) 이메일 발송을 테스트하는 코드가 있다. 테스트 코드를 실행할 때마다 이메일이 전송되어야 할까? 
물론 아니다. EmailSender 객체 대신 DummyEmailSender 객체를 사용하면 된다. DummyEmailSender 를 보고 Test double이라고 부른다.

## 종류

1. dummy
   - 아무런 동작도 하지 않고, 그저 코드가 정상적으로 돌아가기 위해 전달하는 객체
   - 메서드 내부에 구현이 없고 텅 비어있다.
2. fake
   - 로컬에서 사용하거나 테스트에서 사용하기 위해 만들어진 가짜 객체
   - 자체적인 로직이 있다.
   - ex) DatabaseUserRepository vs MemoryUserRepository
3. stub
   - 미리 준비된 값을 출력하는 객체
   - 외부 컴포넌트와 연동할 때 많이 사용된다.
   - Mockito의 given, Mockk의 every 의 기능이 stub 객체를 만들어준다.
4. mock
   - 메서드 호출을 확인하기 위한 객체
   - 자가 검증 능력을 갖춤
   - 사실상 test double와 동일한 의미이다.
   - stub도 mock이 될 수 있고, dummy, fake 도 마찬가지이다.
5. spy
   - 메서드 호출 기록을 전부 기록했다가 나중에 확인하기 위한 객체이다.
