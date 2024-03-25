## 인프라 고민

## 1. 회고 보드 서비스를 어떻게 배포하지?
- 회고 보드 서비스를 개발하기로 했다.
  - 기술 스택은 개발자들의 선호에 따라 Java, Spring Boot, MySQL을 사용하기로 했다.
- 개발은 알아서 잘 한다. 하지만 배포는 어떻게 하지?
  - 개발자 중 대부분이 코린이라서 최대한 관리할 필요가 없는 서비스를 선택하고자 했다.
  - AWS BeanStalk vs AppRunner 중 골라보기로 했다. 

## 2. AWS BeanStalk vs AppRunner
- BeanStalk과 AppRunner 모두 완전 관리형 서비스이다.
  - 오토 스케일링, 프로비저닝, 로드밸런싱, 모니터링, 자동 배포, SSL 인증 등 다양한 기능이 제공된다.
  - 신경써야 할 지점이 확 줄어든다!
- 차이점으로, BeanStalk은 코드 기반이고 AppRunner는 컨테이너 기반으로 배포된다는 점이다.

## 3. AppRunner는 JDK 11 이후 버전으로 배포 불가능하다는 것을 알게 됨.
- 믿기 힘들었지만, JDK 11 이후로 호환성 업데이트가 안되고 있었다.
- 현재 개발을 JDK 17, Spring Boot 3버전을 쓰고 있었는데 AppRunner 사용을 위해 JDK 11로 다운그레이드 하면 Spring Boot도 2버전으로 내려야 하는 큰 작업이 생긴다.
  - 당장 다운그레이드 하는 것은 큰 공수가 들지 않지만, 나중에 서비스가 커졌을 때 업그레이드 하는건 큰 공수가 들 것으로 예상했다.

## 4. AppRunner 포기ㅠㅠ. 우선 EC2에 배포하자.
- AppRunner를 포기하기로 했다. 첫 배포 전에 큰 문제가 있음을 알아서 다행이다.
- 우선 EC2에 배포하기로 했다. 따라서 신경써야 할 점이 조금 늘었다.

## 5. 배포는 AWS CDK를 활용한다.
- CDK 전용 IAM Role을 생성한다.
- https를 위한 도메인을 구매하고 설정한다. (Route53)
- 도메인으로 들어온 요청을 받을 ELB와 Target Group을 생성한다.
- 이미지 등 정적 리소스를 저장할 S3 버킷을 생성한다.
- RDBMS 관리형 서비스인 RDS 인스턴스를 생성한다.

## 6. 천천히 서버리스로 전환 준비하자.
- 천천히 Lambda + Spring Boot PoC 진행해보자.
- (EC2 + Spring MVC + JPA를 사용한다고 가정) 상태를 유지가 필요한 Tomcat Thread Pool, HikariCP는 어떻게 처리하는 게 좋을까?
  - Tomcat Thread Pool
    - 오랫동안 실행되면서 요청을 동시에 처리하기 위해 스레드 풀을 관리한다.
    - 하지만 람다는 요청 1개당 1개의 함수가 실행되는 방식이기 때문에 Thread Pool 혹은 유사한 것이 필요없다.
  - HikariCP
    - RDS의 커넥션을 HikariCP가 미리 만들어서 관리하고 있었다.
    - 하지만 Lambda는 커넥션을 유지할 수 없다. 따라서 RDS Proxy를 사용해서 람다 외부의 커넥션 관리 서비스를 사용하면 된다.
- AWS API Gateway를 사용해서 REST API 인터페이스를 갖추고 람다를 트리거하자.
- AWS Lambda Handler를 빈으로 등록해서 사용할 수 있게 해주는 [Spring Cloud Function](https://spring.io/projects/spring-cloud-function)도 있다.
- Snap Start 기능은 반드시 사용하자. 특히 JVM App이라면 더욱 더!!
