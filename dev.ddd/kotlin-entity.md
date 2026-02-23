# Kotlin 도메인 모델링 논의 정리

## 논의 배경

도메인 모델을 Kotlin으로 개발할 때 `data class`가 적합한가?라는 질문에서 논의가 시작되었다. 코드 리뷰 과정에서 setter 사용 방식에 대한 의견이 오가면서, 자연스럽게 두 가지 주제로 확장되었다.

### 초기 오해와 보정

논의 초반, "일반 class는 `var`로 선언해도 public setter를 쉽게 숨길 수 있다"는 전제가 있었다. 하지만 이는 사실이 아니었다. Kotlin에서는 `data class`든 일반 `class`든 생성자에서 `var`로 프로퍼티를 선언하면 **모두 public getter/setter가 생성**된다. setter를 숨기려면 어느 쪽이든 아래와 같이 `private set`을 별도로 선언해야 한다.

```kotlin
// data class든 일반 class든 동일하게 적용
class Member(
    val id: Long,
    var name: String
        private set  // 이렇게 해야 setter가 숨겨진다
)
```

이 오해가 해소된 후, 논의는 setter 자체의 문제가 아니라 "도메인 모델에서 데이터 변경을 어떻게 표현할 것인가"와 "Entity 구현에 `data class`와 `class` 중 어느 것이 적합한가"라는 두 가지 핵심 주제로 넘어갔다.

---

## 주제 1: Setter vs 비즈니스 의도가 드러나는 메서드

### 두 가지 입장

**A. setter를 허용하자**

단순한 값 변경(전화번호, 이메일, 통신사 등)은 setter만으로도 의도가 충분히 명확하다. 모든 필드에 대해 업데이트 메서드를 만드는 것은 과도한 엔지니어링이 될 수 있다. 또한, setter를 사용함으로써 "이 필드에는 별도의 도메인 규칙이 없다"는 점을 오히려 드러낼 수도 있다.

**B. setter는 모두 닫고, 비즈니스 의도가 드러나는 메서드만 쓰자**

setter를 전부 숨기고, 유스케이스별 메서드(`promote`, `verify` 등)로 데이터 변경을 표현하자는 입장이다. 가장 큰 장점은 여러 필드가 함께 변경되어야 하는 경우 한 메서드로 묶어서 관리할 수 있다는 점이다.

예를 들어 회원 등급을 인증 회원으로 승격하는 경우, `promote(ciNumber)` 메서드 안에서 DBCD(회원 구분 코드) 변경, CI 번호 설정, 밸리데이션을 한번에 수행할 수 있다. 또한 다른 팀이나 신규 입사자가 해당 Entity를 사용할 때 setter가 열려 있으면 의도치 않은 방식으로 데이터를 변경할 우려도 있다.

### 결론: 상황에 맞게 혼용

데이터 변경 시 비즈니스 의도가 드러나는 메서드를 사용하는 것은 좋지만, VO처럼 단순한 값을 가지는 클래스나 도메인 규칙이 거의 없는 필드에서는 setter를 허용한다. 핵심은 다음과 같다.

- **setter 허용**: 부가 정보(이메일, 전화번호 등) 같은 단순 필드, 도메인 규칙이 적은 경우
- **메서드 권장**: 여러 필드가 동시에 변경되어야 하거나 밸리데이션이 필요한 경우
- **setter 지양**: 비즈니스 규칙이 복잡하거나, 다른 팀과 공유되는 핵심 Entity

---

## 주제 2: Entity 구현 — data class vs class

### 비교 포인트

| 기능 | data class | class |
|---|---|---|
| equals() / hashCode() | 모든 프로퍼티 기반으로 자동 생성 | Object 기본 구현 (참조 비교) |
| copy() | 자동 제공 (일부 값만 변경한 새 객체 생성 가능) | 없음 (필요 시 직접 구현) |
| toString() | 모든 프로퍼티를 포함하여 자동 생성 | 클래스명@해시코드 (기본) |
| componentN() | 자동 제공 (구조 분해 가능) | 없음 |
| var 프로퍼티 setter | public (동일) | public (동일) |

### 논의 내용

**equals/hashCode 관련**

Entity의 동등성은 비즈니스적으로 ID(식별자) 하나로 판단해야 한다. `data class`는 모든 프로퍼티를 기반으로 equals/hashCode를 자동 생성하므로 Entity의 동등성 개념과 맞지 않는다. 물론 `data class`에서 equals/hashCode를 override할 수는 있지만, `data class`를 사용한다는 것 자체가 "모든 값이 같으면 같은 객체"라는 암묵적 합의를 개발자들 사이에 형성하기 때문에, override하면 오히려 혼란을 줄 수 있다.

반면 일반 `class`에서도 equals/hashCode를 직접 구현해야 하는 것은 마찬가지다. 결국 어느 쪽이든 ID 기반으로 직접 구현해야 하므로, 이 부분만으로는 결정적 차이가 되지 않는다.

**copy() 관련**

`data class`의 `copy()`는 `val`로 선언한 프로퍼티조차 변경된 새 객체를 만들 수 있다. 예를 들어 국내 주식 전용 증권 계좌에서 통화(currency)가 KRW로 고정되어야 하는데, `copy(currency = Currency.USD)`로 비즈니스 규칙을 위반하는 객체가 생성될 수 있다.

```kotlin
data class StockAccount(val id: Long, val balance: BigDecimal, val currency: Currency)

// 의도하지 않은 사용 — 국내 주식 전용인데 USD로 변경된 객체 생성 가능
val invalid = account.copy(currency = Currency.USD)
```

다만 논의 과정에서, 이 문제는 Entity에 국한된 것이 아니라 VO에서도 동일하게 발생한다는 점이 확인되었다. 예를 들어 "2달러"라는 금액 VO를 `copy`로 "2원"으로 바꿀 수도 있다. 따라서 copy()의 위험성은 `data class` 자체의 특성이지, Entity 여부와는 무관한 이슈다.

**data class의 설계 의도**

Kotlin 공식 문서에 따르면 `data class`는 "데이터를 보관하기 위해 만들어진 클래스"이다. 식별자를 가진 Entity를 위한 것이 아니라 VO/DTO 용도로 설계된 것이다. 이 점에서 `data class`를 Entity에 쓰는 것은 설계 의도와 맞지 않다.

### 결론: Entity는 일반 class를 사용

equals/hashCode는 어느 쪽이든 직접 구현해야 하고, copy()의 위험성은 Entity/VO 무관하게 `data class` 자체의 이슈이므로 결정적 차이는 아니다. 그러나 `data class`를 사용하면서 override하는 것은 개발자들의 암묵적 합의("모든 값이 같으면 같은 객체")를 깨뜨리고, `data class`의 원래 설계 의도(데이터 보관)와도 맞지 않기 때문에 **Entity는 일반 class를 사용하기로 결정**했다. `data class`는 VO/DTO에 사용한다.

---

## 최종 합의 요약

| 주제 | 결정 |
|---|---|
| Setter 사용 | 비즈니스 의도 메서드를 기본으로 하되, 단순 값 변경에는 setter 허용 (혼용) |
| Entity 구현 | 일반 `class` 사용. `data class`는 VO/DTO에 사용 |
| copy() | Entity/VO 무관하게 `data class` 자체의 이슈이므로, 사용 시 비즈니스 규칙 위반 여부를 주의 |

---

## 참고자료

- [Spoqa 기술 블로그 — Kotlin으로 JPA Entity를 정의하는 방법](https://spoqa.github.io/2022/08/16/kotlin-jpa-entity.html)
- [Kotlin 공식 문서 — Data Classes](https://kotlinlang.org/docs/data-classes.html)
- [Baeldung — Working with Kotlin and JPA](https://www.baeldung.com/kotlin/jpa)
- [JetBrains Blog — Common Pitfalls with JPA and Kotlin](https://blog.jetbrains.com/idea/2026/01/how-to-avoid-common-pitfalls-with-jpa-and-kotlin/)