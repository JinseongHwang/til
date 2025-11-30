## Liveness

- 애플리케이션이 살아있는가?
- 상태
    - CORRECT: 애플리케이션이 정상적으로 실행 중임.
    - BROKEN: 애플리케이션이 내부적인 치명적 오류(데드락, 스레드 풀 고갈 등)로 인해 더 이상 동작할 수 없는 상태.
- Liveness가 실패하면 플랫폼(ex: Kubernetes)은 해당 애플리케이션 인스턴스를 재시작한다.

```java
package org.springframework.boot.availability;

/**
 * "Liveness" state of the application.
 * <p>
 * An application is considered live when it's running with a correct internal state.
 * "Liveness" failure means that the internal state of the application is broken and we
 * cannot recover from it. As a result, the platform should restart the application.
 *
 * @author Brian Clozel
 * @since 2.3.0
 */
public enum LivenessState implements AvailabilityState {

	/**
	 * The application is running and its internal state is correct.
	 */
	CORRECT,

	/**
	 * The application is running but its internal state is broken.
	 */
	BROKEN

}
```

## Readiness

- 애플리케이션이 트래픽을 받을 준비가 되었는가?
- 상태
    - ACCEPTING_TRAFFIC: 요청을 처리할 준비가 완료됨.
    - REFUSING_TRAFFIC: 실행은 되고 있지만, 아직 초기화 작업 중이거나 과부하 상태라 요청을 받을 수 없음.
- Readiness가 실패하면 로드 밸런서에서 해당 인스턴스를 제외하여 트래픽이 유입되지 않도록 한다. (재시작 X)

```java
package org.springframework.boot.availability;

/**
 * "Readiness" state of the application.
 * <p>
 * An application is considered ready when it's {@link LivenessState live} and willing to
 * accept traffic. "Readiness" failure means that the application is not able to accept
 * traffic and that the infrastructure should stop routing requests to it.
 *
 * @author Brian Clozel
 * @since 2.3.0
 */
public enum ReadinessState implements AvailabilityState {

	/**
	 * The application is ready to receive traffic.
	 */
	ACCEPTING_TRAFFIC,

	/**
	 * The application is not willing to receive traffic.
	 */
	REFUSING_TRAFFIC

}
```

## Health check과 Liveness/Readiness는 어떤 차이?

- health check의 경우 여러 컴포넌트 중 단 하나라도 실패하면 DOWN 상태이고, 모두 정상이어야 UP 상태이다.
- 단, Liveness/Readiness는 조절 가능하다. K8S에서 자동화된 조치(재시작/트래픽 차단)를 판단할 때 활용한다.
- 기본 /health는 정보 제공용에 가깝고, Liveness/Readiness는 자동 복구 및 트래픽 제어용이라 생각하면 됨.

## application.yaml 설정

```yaml
management:
  endpoints:
    web:
      exposure:
        include: "*"
  endpoint:
    health:
      show-details: always
      probes:
        enabled: true
  health:
    liveness-state:
      enabled: true
    readiness-state:
      enabled: true
```

## /actuator/health 조회 시 결과

```json
{
  "status": "UP",
  "groups": [
    "liveness",
    "readiness"
  ],
  "components": {
    "db": {
      "status": "UP",
      "details": {
        "database": "H2",
        "validationQuery": "isValid()"
      }
    },
    "diskSpace": {
      "status": "UP",
      "details": {
        "total": 245107195904,
        "free": 154496643072,
        "threshold": 10485760,
        "path": "/Users/jinseonghwang/IdeaProjects/spring-playground/observability/.",
        "exists": true
      }
    },
    "livenessState": {
      "status": "UP"
    },
    "ping": {
      "status": "UP"
    },
    "readinessState": {
      "status": "UP"
    },
    "ssl": {
      "status": "UP",
      "details": {
        "validChains": [],
        "invalidChains": []
      }
    }
  }
}
```

- liveness/readiness 가 추가된 것을 확인할 수 있다.
- 아래 엔드포인트로 활용 가능.
    - /actuator/health/liveness
    - /actuator/health/readiness

## Liveness/Readiness 활용법

- Liveness 체크에 DB나 외부 API 상태를 포함시키면 안 된다.
    - 잘못된 시나리오: DB가 일시적으로 느려져서 Liveness가 실패함 -> Kubernetes가 멀쩡한 애플리케이션 서버를 재시작함 -> 재시작되는 동안 트래픽 처리 불가 -> 무한 재시작 루프 발생.
    - 올바른 구성: Liveness는 오직 애플리케이션 자체의 내부 상태(Deadlock, Memory 등)만 체크해야 함. Spring Boot 기본 설정은 Liveness 그룹에 외부 의존성을 포함하지 않는다.
- Readiness에는 '핵심 의존성'을 포함해야 한다.
    - 애플리케이션이 시작되었어도 DB 연결이 안 되거나 캐시 로딩이 안 끝났다면 트래픽을 받으면 안 된다.
    - DB, Redis 등의 상태를 Readiness 그룹에 포함시킨다.
    - DB 장애 발생 -> Readiness 실패 -> 로드 밸런서에서 제외됨 -> 사용자에게는 에러가 아닌 다른 정상 서버로 요청이 감 (무중단).
    ```yaml
    management:
        endpoint:
            health:
                group:
                    liveness:
                        include: livenessState, ping
                    readiness:
                        include: readinessState, db, kafkaStreamHealth, diskSpace
    ```
- 애플리케이션 종료 시 바로 꺼지는 게 아니라 Readiness를 먼저 REFUSING_TRAFFIC 상태로 바꾸고 기존 요청을 다 처리한 뒤 종료하도록 해야 한다.
    ```yaml
    server:
    shutdown: graceful

    spring:
    lifecycle:
        timeout-per-shutdown-phase: 20s # 종료 대기 시간
    ```

## 어플리케이션 Lifecycle에 따라 liveness와 readiness상태가 변경되는 절차 

1. Registering listeners and initializers
2. Preparing the Environment
3. Preparing the ApplicationContext
4. Loading bean definitions
5. **Changing the liveness state to CORRECT --> liveness 상태 변경**
6. Calling the application and command-line runners
7. **Changing the readiness state to ACCEPTING_TRAFFIC --> readiness 상태 변경**
