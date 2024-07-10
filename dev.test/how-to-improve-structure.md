## 배경 상황

- Java, Spring Boot(MVC) 애플리케이션이 있다.
- Layered architecture를 따르는 구조이다.
- JPA를 사용 중이며, 테스트 시에는 In-memory DB인 H2를 사용한다.
- JUnit으로 Controller, Service, Repository의 테스트 코드를 작성했다.
- Mock은 사용하지 않으며 실제 테스트 DB를 사용한다.

## 문제점1: H2

- 테스트를 실행할 때마다 In-memory DB인 H2가 실행된다. 따라서 매우 느리고 무겁다.
- 시스템이 RDB에 강하게 결합되어 있음을 의미한다.
- H2나 Mockito 없이는 테스트 할 수 없는 구조이다.

## 문제점2: Layered architecture

![image](https://github.com/JinseongHwang/til/assets/132965185/207685ea-1fa1-477d-9970-f7da18571e69)

[Presentation] -> [Business] -> [Persistence]

- 레이어드 아키텍쳐의 특징은,
  - 유사한 기능을 같은 계층으로 묶어 관리하는 방식의 아키텍처 구조
  - 의존성 역전이나 추상화 없이 바로 구현체를 사용하는 구조
  - 쉽다!
- 문제점
  - DB 주도 설계가 된다. 영속성 엔티티를 가장 먼저 결정해야 한다.
    - Use case를 먼저 파악하고, 도메인과 도메인들의 관계를 생각하는 게 우선시 되어야 한다.
  - Persistence 계층을 먼저 작업하고, Business 계층을 작업하고, Presentation 계층을 작업해야 한다. 절대 순서를 어길 수 없다.
  - 도메인의 특성을 살리기 힘들다. 객체는 수동적이게 되고 모든 코드가 함수 위주로 돌아가게 된다.
    - "도메인"에 대해 진지하게 고민해보지 않게 된다.
    - 사실상 두꺼운 Service 계층이 모든 일을 다 처리하게 된다.
  - 낮은 Testablity와 SOLID를 지킬 수 없게 된다.

## 개선된 아키텍처

![image](https://github.com/JinseongHwang/til/assets/132965185/2c29ce02-f978-4de6-8222-74a520fcc6c3)

[Presentation] -> [Business] -> [Domain] -> [Persistence]

- Business: 도메인을 Repository에서 가져와 책임을 위임하는 계층  
- Domain: (핵심) 도메인들이 협력하는 계층. POJO로만 구성되며, SOLID를 철저하게 지킨다.


위 이미지처럼 구현하면, Testability가 높아진다.
- Presentation layer: Controller는 Service(interface)를 의존한다. 추상화에만 의존한다.
- Business layer: ServiceImpl은 Repository(interface)와 Domain을 의존한다. 추상화에 의존하고, POJO에 의존한다.
- Domain layer: POJO이기 때문에 테스트가 쉽다.
- Infrastructure layer: RepositoryImpl은 JpaRepository(interface)를 의존한다. 추상화에만 의존한다.

모두 추상화 or 순수 Java 코드에만 의존하기 때문에 테스트 하기 쉽다. 테스트 하기 쉬울 뿐만 아니라 커플링이 느슨해져서 유연하게 구성할 수 있게 된다.

## 어떤 모듈을 중요시 하는게 좋을까?

- 테스트 대상인 구체 클래스는 Controller, ServiceImpl, Domain, RepositoryImpl 이렇게 4가지이다.
- 굳이 중요도를 따지면, ServiceImpl과 Domain이 가장 중요하다.
  - Controller에서는 요청과 응답에 대한 아주 가벼운 로직만 존재한다.
    - (Spring MVC를 사용한다면) Spring 개발팀에서 충분히 테스트를 했을거라고 믿는다.
  - RepositoryImpl에서는 DB CRUD에 필요한 아주 가벼운 로직만 존재한다.
    - (JPA Hibernate를 사용한다면) Hibernate 개발팀에서 충분히 테스트를 했을거라고 믿는다.
  - 클린 아키텍처에서는 이처럼 테스트하기 어렵거나 본질에서 벗어나는 부분을 "Humble object"라고 부른다.
  - 이 부분을 테스트 하지 않아서 테스트 커버리지가 낮아지는 것은 크게 신경쓰지 않아도 된다. (주관적)
- ServiceImpl과 Domain의 로직이 비즈니스의 핵심 로직이다. 차라리 이 부분을 테스트 하는데 집중하자.

## 패키지 구조 개선하기

Before
```
root
ㄴ controller
ㄴ exception
ㄴ model
ㄴ repository
ㄴ service
```
- 장점
  - 쉽다. 단순하다.
- 단점
  - 도메인이 눈에 보이지 않는다.
  - 도메인으로 구분해서 동시에 작업하기 힘들다.
  - 사실상 경계, 의존성을 관리하지 않겠다는 의미이다.

After
```
root
ㄴ common
  ㄴ controller
  ㄴ exception
ㄴ post
  ㄴ controller
  ㄴ model
  ㄴ repository
  ㄴ service
ㄴ user
  ㄴ controller
  ㄴ model
  ㄴ repository
  ㄴ service
```
- 장점
  - 어떤 도메인을 다루는 시스템인지 눈에 확 띈다.
  - 필요에 따라 MSA로 시스템 확장에 용이하다.
- 단점
  - MVC에 익숙한 경우 컴포넌트 찾기가 어색할 수 있다.

## 최종

![패키지개선](https://github.com/JinseongHwang/til/assets/132965185/2fbdfa43-0586-4fd5-8dbd-47b81b144e0e)

- 사실 최종까진 아니지만..
- 참고로 도메인 패키지 간 참조도 발생할 수 있다. 하지만 순환 참조는 발생하면 안된다
- 나중에 헥사고날 아키텍처도 살펴보자

