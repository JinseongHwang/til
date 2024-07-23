## JPA Query Method의 findBy, findOneBy 차이점은?

### Query Method 기본 사용법

```kotlin
fun findByName(name: String): User?
```
- find 는 select 쿼리를 생성한다.
- by 는 where 절을 생성한다.

### 여러 건을 가져오고 싶다면?

```kotlin
fun findByName(name: String): User?        // 1건만 조회
fun findByName(name: String): List<User>   // 여러건 조회
```
- 1건을 가져올지 여러건을 가져올지는 return type이 결정한다.

### findOneBy 는 어떻게 동작할까?

```kotlin
fun findByName(name: String): List<User>     // 여러건 조회
fun findOneByName(name: String): List<User>  // 여러건 조회
```
- findOneBy 는 1건만 조회할 것 같지만, 아무 의미가 없다.
- 조회 건수가 단수, 복수일지는 return type이 결정한다.
- 따라서 findBy, findOneBy 모두 여러건을 조회할 수 있다.
- findAllBy, findFooBy, findBarBy, findXYZBy 모두 같은 쿼리를 생성한다.
  - ==> **JPA는 쿼리를 생성할 때 find와 by 사이의 모든 문자를 무시한다.**

### 단건 조회를 해야 하는데 여러 건이 존재하면 어떻게 될까?

```kotlin
fun findByName(name: String): User?
```
- 위 메서드를 실행하면 1건(or null)이 조회되길 기대한다.
- 하지만 실제로 DB에 여러건이 존재하면 IncorrectResultSizeDataAccessException 이 발생한다.

### 예외 케이스

```kotlin
fun findTopByName(name: String): User?
fun findTop3ByName(name: String): List<User>
fun findFirstByName(name: String): User?
fun findFirst3ByName(name: String): List<User>
```
- 예외적으로 Top과 First 키워드는 무시되지 않고 limit 절을 생성해준다.
- 뒤에 숫자가 들어가면 `limit N` 이고, 숫자가 없으면 `limit 1`이다.

## References

- https://docs.spring.io/spring-data/jpa/reference/repositories/query-methods-details.html#repositories.limit-query-result
- https://www.baeldung.com/spring-data-jpa-findby-vs-findoneby