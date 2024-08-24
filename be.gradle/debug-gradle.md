## Gradle 명령어 실패 디버깅 하기

## 예시 에러 메시지

```Plain Text
PM 8:06:17: Executing 'build'...

> Task :checkKotlinGradlePluginConfigurationErrors
> Task :kaptGenerateStubsKotlin UP-TO-DATE
> Task :processResources UP-TO-DATE
> Task :processTestResources NO-SOURCE
> Task :kaptKotlin FAILED
4 actionable tasks: 2 executed, 2 up-to-date

FAILURE: Build failed with an exception.

* What went wrong:
Execution failed for task ':kaptKotlin'.
> A failure occurred while executing org.jetbrains.kotlin.gradle.internal.KaptWithoutKotlincTask$KaptExecutionWorkAction
   > java.lang.reflect.InvocationTargetException (no error message)

* Try:
> Run with --stacktrace option to get the stack trace.
> Run with --info or --debug option to get more log output.
> Run with --scan to get full insights.

* Get more help at https://help.gradle.org

BUILD FAILED in 1s
PM 8:06:19: Execution finished 'build'.
```

- 빌드가 실패했다.
- 나를 가장 화나게 하는 건 "no error message" 이다.
- 뭐 어쩌라는 건지 모르겠다.

## 디버깅하기

```Shell
./gradlew build --stacktrace --info
```

- 다행히도 stacktrace를 살펴볼 수 있는 기능을 제공한다.

```Plain Text
Caused by: java.lang.NoClassDefFoundError: javax/persistence/Entity
        at com.querydsl.apt.jpa.JPAAnnotationProcessor.createConfiguration(JPAAnnotationProcessor.java:37)
        at com.querydsl.apt.AbstractQuerydslProcessor.process(AbstractQuerydslProcessor.java:82)
        at org.jetbrains.kotlin.kapt3.base.incremental.IncrementalProcessor.process(incrementalProcessors.kt:90)
        at org.jetbrains.kotlin.kapt3.base.ProcessorWrapper.process(annotationProcessing.kt:209)
        at jdk.compiler/com.sun.tools.javac.processing.JavacProcessingEnvironment.callProcessor(JavacProcessingEnvironment.java:1023)
        ... 44 more
```

- 수백줄 쭈르륵 나오다가 요런 부분을 찾았다.
- 아! QueryDSL 쪽에서 문제가 있음을 알게 됐다.
- 사막에서 바늘 찾기 문제가, 내 방에서 바늘 찾기 문제로 바뀌었다.
  - fyi. 내 방은 약 5평이다.
