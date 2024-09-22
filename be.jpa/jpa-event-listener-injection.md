## JPA Entity Event Lister 에서 JPA Repository를 주입받을 때 Lazy 필드 주입을 해야 하는 이유?

## 예시 코드 구현

#### JPA Entity

```kotlin
@Entity
@EntityListeners(FooEntityEventListener::class)
class FooEntity(
    @Id
    val id: Long,
    val name: String
)
```

#### JPA Repository

```kotlin
interface FooRepository : JpaRepository<FooEntity, Long> {
}
```

#### JPA Entity Event Listener

```kotlin
@Component
class FooEntityEventListener {

    @Lazy
    @Autowired
    private lateinit var fooRepository: FooRepository

    @PostPersist
    fun postPersist(foo: FooEntity) {
        fooRepository.save(foo)
    }
}
```

## 궁금한 점

JPA Entity Event Lister 에서 JPA Repository를 주입받을 때 Lazy 필드 주입을 해야 하는 이유가 뭘까?

## 단서1: JPA Repository 를 사용하기 위해선 EntityManager가 필요하다.

FooRepository는 JpaRepository를 상속받고 있다.
JpaRepository는 인터페이스이고 그 중 적절한 구현체를 사용하게 된다.

![image](https://github.com/user-attachments/assets/ef77857f-712c-4449-ac15-4b27dcbc62ce)
 
기본적으로 SimpleJpaRepository를 사용하는데, 이는 EntityManager가 필수로 있어야 사용 가능하다. 
EntityManager는 EntityManagerFactory에서 생성된다.

> 결론: 따라서 EntitiyManagerFactory가 필수적으로 준비되어 있어야 애플리케이션이 뜨는 시점에 주입받아 사용할 수 있다.

## 단서2: SessionFactoryImpl에서 EventEngine를 필수로 등록해야 한다.

EntityManagerFactory는 인터페이스이다.
EntityManagerFactory를 확장하는 SessionFactory라는 인터페이스가 있는데 얘를 살펴보자.
SessionFactory의 주석을 잘 살펴보면 이런 문장이 나온다.

![image](https://github.com/user-attachments/assets/238fc03f-9dbc-4c6f-9493-685433df0935)

"Every `SessionFactory` is a JPA `EntityManagerFactory`."

참고로 EntityManagerFactory는 jakarta.persistence 패키지 소속이고, SessionFactory는 org.hibernate 패키지 소속이다.

그렇다면 SessionFactory의 구현체인 SessionFactoryImpl 을 살펴보자.  
(정확히는 SessionFactory에 부가기능이 추가된 SessionFactoryImplementor 인터페이스를 구현한다)

![image](https://github.com/user-attachments/assets/6281e895-e85f-4706-979f-3127e5a5f0ca)

많은 필드 중 EventEngine 이라는 게 눈에 들어온다.

![image](https://github.com/user-attachments/assets/108a9927-42f0-41f2-84f8-c53a4a044aa1)

EventEngine의 생성자에서 이벤트와 리스너를 등록하는 것을 알 수 있다.

> 결론: 이벤트 리스너는 SessionFactory(EntityManagerFactory) 생성 시점에 등록이 된다.

## 최종 결론

그렇다면 정리를 다시 해보자.

1. FooRepository(JpaRepository)를 생성하기 위해서는 EntityManagerFactory가 필요하다.
2. EntityManagerFactory를 만드는 시점에 EventListeners가 등록된다.

> 최종 결론: EventListener가 등록된 후에 FooRepository 빈을 생성할 수 있는 것이다. 따라서 EventListener에서 FooRepository를 즉시 주입받는 것은 불가능하다. Lazy 주입만 가능한 것이다.