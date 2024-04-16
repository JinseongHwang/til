## JVM 진영의 로그 이야기

### Log란?

- 애플리케이션의 상태, 이벤트 등을 기록한 데이터를 로그라고 부른다.
- 목적과 발생 위치에 따라 시스템 로그, 애플리케이션 로그, 엑세스 로그 등으로 구분할 수 있다.
- 로그를 남기는 행위를 로깅(Logging)이라고 한다.

### Logging framework

- logback
- reload4j
- log4j / log4j2
- JUL
- ...

여러가지로 꽤 많다.

### Logging framework 쓰기 귀찮은데 System.out / System.err 쓰면 안됨? 

- 써도 된다. 근데 안쓰는 데는 이유가 있다.
  - 로그 레벨을 설정할 수 없다.
  - 시간, 스레드 등 다양한 메타정보를 출력하려면 귀찮고 중복 코드가 발생한다.
  - 내부적으로 syncronized가 있어서 즉시 IO Blocking이 발생한다. 성능에 영향을 미칠 수 있다.

쓰지 말자.

### SLF4J

- Simple Logging Facade for Java 의 약자이다.
- 즉, 어떠한 로깅 프레임워크가 아니라 로깅 프레임워크의 추상화를 지원하는 Facade이다.
- Lombok을 사용하면 Slf4j 어노테이션이 제공되는데, 이는 로깅 추상화만 제공하는 것이다. 내부적으로는 기본으로 Logback을 사용한다.
  - https://projectlombok.org/api/lombok/extern/slf4j/Slf4j

### Logback

- Log4j를 개선한 버전으로, Log4j를 전신으로 한다.
- Slf4j 인터페이스를 따르기 때문에 Slf4j를 사용하고 있다면 코드 수정 없이 구현체 변경이 가능하다.
- Spring Boot의 내장 로깅 프레임워크로 사용되고 있다.

### MDC

- Mapped Diagnostic Context의 약자이다.
- 멀티스레드 애플리케이션에서는 여러 스레드가 동시에 요청을 처리한다. 각 스레드에서 내뿜는 로그는 순서 없이 뒤섞이게 된다.
이 문제를 해결하기 위해 요청 별로 고유한 값을 부여하고, 그 값을 로그에 함께 출력되도록 한다.
- MDC는 현재 실행 중인 스레드에 메타 데이터를 관리하는 공간을 의미한다. 즉, ThreadLocal에 현재 처리 중인 요청 정보를 임시로 넣어서 관리한다는 의미이다.

우선 MDC를 필터 레벨에서 생성하고 요청이 들어올 때마다 request_id가 생성되도록 한다.

```Java
@Component
@Order(Ordered.HIGHEST_PRECEDENCE)
class MDCLoggingFilter implements Filter {

    @Override
    public void doFilter(final ServletRequest servletRequest, final ServletResponse servletResponse, final FilterChain filterChain) throws IOException, ServletException {
        final UUID uuid = UUID.randomUUID();
        MDC.put("request_id", uuid.toString());
        filterChain.doFilter(servletRequest, servletResponse);
        MDC.clear();
    }
}
```

그리고 Logback에 다음과 같이 설정해서 로그에 request_id가 함께 출력되게 한다. (예시)
```Shell
[%X{request_id:-startup}] %d{HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n
```

## Logback 구성요소

### Appender

- 로그를 외부로 출력하는 방법을 정의한다.
- 자주 사용하는 내장된 Appender
  - ConsoleAppender :  console에 로그를 찍는다. 
  - FileAppender : file에 로그를 찍는다.
  - RollingFileAppender : FileAppender를 확장한 버전. file로 로그를 찍되 rolling 기법으로 파일을 분할한다.
  - 이 외에도 다양한 Appender가 있고 직접 구현하는 것도 가능하다.

### Encoder

- 로그를 직렬화하는 방법이다.
- 상황에 따라 Layout 까지 함께 처리하는 패턴도 꽤 존재한다.

```xml
<encoder class="net.logstash.logback.encoder.LoggingEventCompositeJsonEncoder">
  <providers>
    <provider>
      <pattern>
        {
          "traceId": "%X{dd.trace_id}",
          "spanId": "%X{dd.span_id}",
          "dd.service": "my-application",
        }
      </pattern>
    </provider>
  </providers>
</encoder>
```
- logback-spring.xml 일부, 즉 Application Log의 Provider 인 것.
- Datadog을 사용하는 경우에는 위와 같이 설정해서 로그 추적이 가능하다.
- dd 가 prefix로 붙는 부분은 Datadog Agent가 붙어서 주입해준다.
  - https://github.com/DataDog/dd-trace-java
  - MDC의 역할을 수행해주는 것이다.

```xml
<encoder class="net.logstash.logback.encoder.AccessEventCompositeJsonEncoder">
  <providers>
    <provider class="net.logstash.logback.composite.accessevent.AccessEventPatternJsonProvider">
      <pattern>
        {
        "traceId": "#nullNA{%reqAttribute{dd.trace_id}}",
        "spanId": "#nullNA{%reqAttribute{dd.span_id}}",
        "dd.service": "my-application",
        }
      </pattern>
    </provider>
  </providers>
</encoder>
```
- logback-access.xml 일부, 즉 Access Log의 Provider 인 것.
- Access Log는 Tomcat 레벨에서 출력하므로 MDC에는 trace_id와 span_id가 존재하지 않는다.
- 따라서 reqAttribute에 들어있는 trace_id와 span_id를 출력하도록 설정해야 한다.
  - https://github.com/DataDog/dd-trace-java/issues/1214

### Layout

- 로그를 어떤 포맷으로 출력할 것인지 정의한다.
- 주로 PatternLayout을 사용하며 특정 패턴을 조합해서 레이아웃을 결정한다.

```xml
<layout class="ch.qos.logback.classic.PatternLayout">
    <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} %-5level %logger{35} - %msg%n</pattern>
</layout>
```

### Filter

- 로그를 필터링 하는 기능이다.
- 최소 INFO 이상만 로깅한다던지, 다양한 필터 기능을 활성화 할 수 있다.

```xml
<filter class="ch.qos.logback.classic.filter.ThresholdFilter">
    <level>INFO</level>
</filter>
```

## References

- https://livenow14.tistory.com/64
- https://mangkyu.tistory.com/266