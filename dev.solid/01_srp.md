## SRP (Single Responsibility Principle; 단일 책임 원칙)

> 하나의 클래스는 하나의 책임만 가져야 한다.

- 하나의 클래스에 너무 많은 기능을 때려넣지 마라!
  - "많고 적고 기준은 어떻게 되냐?" -> 내가 정하는거다.
- “하나의 책임”이라는 것은 너무 모호하다. 도대체 “책임”이 무엇인가?
    - 변경이 있을 때 파급 효과(책임 변경의 연쇄작용)가 적으면 단일 책임 원칙을 잘 지킨 것이다.
    - “책임”이라는 모호한 키워드 대신에 “변경”이라는 키워드에 집중해보자.
- 유지보수 vs 성능
    - 유지보수를 먼저 챙기자.
    - 나중에 성능에 문제가 생기면 그 때 개선하자.
    - 소프트웨어는 소프트 해야 한다. 유지보수성을 극대화 하는데 집중하자.
- SOLID 중 가장 먼저 나오는 이유가, 다른 원칙들의 기반 원칙이 되기 때문이다. 뒤에 나오는 OLID 원칙도 SRP를 모두 지켜야 한다.

## Spring 예시

- Spring에서 @Controller, @Service, @Repository를 구분해서 사용하도록 컴포넌트 어노테이션을 제공한다.
- 이 또한 SRP를 지키도록 도와주는 도구라고 생각한다.
  - @Controller (Presentation Layer):
    - Client와의 상호작용을 담당
    - Request를 코드에서 사용할 수 있도록 deserialize, 응답 객체를 전송 가능하도록 serialize
  - @Service (Service Layer)
    - 비즈니스 로직 담당
  - @Repository (Persistent Layer)
    - DB나 파일같은 영속성 관리

## 원칙을 위반한 코드

```java
public class UserService {
   
    public void addUser(String userName) {
        // 사용자 추가 로직
        System.out.println("User " + userName + " added.");
        
        // 데이터베이스 저장
        saveToDatabase(userName);
        
        // 이메일 알림 발송
        sendEmail(userName);
    }

    private void saveToDatabase(String userName) {
        // 데이터베이스 저장 로직
        System.out.println("User " + userName + " saved to database.");
    }

    private void sendEmail(String userName) {
        // 이메일 발송 로직
        System.out.println("Email sent to " + userName);
    }
}
```
- UserService 클래스에 사용자를 추가하는 책임을 가지는 것은 적절하다.
- 하지만 DB 접근이나 이메일 발송 책임도 가지는 것은 SRP를 위반한다.

## 개선된 코드

```java
// 사용자 데이터 관리를 위한 클래스
public class UserManager {
    public void addUser(String userName) {
        System.out.println("User " + userName + " added.");
    }
}

// 데이터베이스 저장을 위한 클래스
public class UserRepository {
    public void saveToDatabase(String userName) {
        System.out.println("User " + userName + " saved to database.");
    }
}

// 이메일 알림을 위한 클래스
public class EmailService {
    public void sendEmail(String userName) {
        System.out.println("Email sent to " + userName);
    }
}

// 위의 클래스를 조합하여 사용하는 클래스
public class UserService {
    private UserManager userManager = new UserManager();
    private UserRepository userRepository = new UserRepository();
    private EmailService emailService = new EmailService();

    public void addUser(String userName) {
        userManager.addUser(userName);
        userRepository.saveToDatabase(userName);
        emailService.sendEmail(userName);
    }
}
```
- 각 책임(기능)에 따라 클래스를 분리했다.
