## JUnit @Mock vs @MockBean

### SpringBootTest 어노테이션을 붙이면

1. 컴포넌트 스캔을 수행하고
2. 빈 생성과 주입이 실행된다.

### 사용할 SUT 훔쳐보기

```Java
@AllArgsConstructor
public class SectionService {
    SectionRepository sectionRepository;
    UserRepository userRepository;
    RetrospectiveRepository retrospectiveRepository;
    TemplateSectionRepository templateSectionRepository;
}
```

우리가 테스트 해야 하는 대상 클래스(SUT)의 이름은 SectionService이다.
SUT를 생성할 때 생성자에서 4개의 인자를 받고 있다. 즉, 4개를 미리 생성된 상태여야지 SUT를 생성할 수 있다. 
다만 우리가 테스트에서 SUT 객체를 생성할 때는 컴포넌트 스캔이 된 빈이 아니라, Mock 객체로 SUT를 만들어야 한다.
그래야 Stubbing을 정상적으로 할 수 있기 때문이다.

### @Mock과 @InjectMocks 활용하기

컴포넌트 스캔 된 빈이 ApplicationContext에 존재한다면 InjectMocks를 통해 객체를 만들더라도 Mock 객체보다 높은 우선순위를 가지는 것 같다. 
그래서 SpringBootTest를 제거하고, 직접 만든 Mock 객체가 주입될 수 있게 해줘야 정상적으로 동작한다.

예시 코드
```java
@Transactional
@ExtendWith(MockitoExtension.class)
class SectionServiceTest {
    @Mock SectionRepository sectionRepository;
    @Mock UserRepository userRepository;
    @Mock RetrospectiveRepository retrospectiveRepository;
    @Mock TemplateSectionRepository templateSectionRepository;
    @InjectMocks SectionService sectionService;
    ...
}
```

### @SpringBootTest와 @MockBean 활용하기

추가적으로 "만약 ApplicationContext가 필요한 경우라면 mockking 못함?"이라는 의문이 들었다.
그럴 땐 MockBean 어노테이션으로 해결할 수 있다. 
생성된 빈 말고 Mock 객체를 주입하겠다는 의미를 가진다.
이렇게 해도 정상적으로 Stubbing이 된다.

예시 코드
```java
@SpringBootTest
@Transactional
@ExtendWith(MockitoExtension.class)
class SectionServiceTest {
    @MockBean SectionRepository sectionRepository;
    @MockBean UserRepository userRepository;
    @MockBean RetrospectiveRepository retrospectiveRepository;
    @MockBean TemplateSectionRepository templateSectionRepository;
    @Autowired SectionService sectionService;
    ...
}
```

