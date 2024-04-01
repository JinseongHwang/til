## JPA `@Query`와 `nativeQuery`를 사용할 때 enum 타입 조건이 동작하지 않는 이슈

## 예시 코드

```Kotlin
enum class Status {
    NORMAL, DELETED
}
```

```Kotlin
@Entity
class User(
    @Id 
    val id: String,
    
    @Enumerated(EnumType.STRING)
    val status: Status
)
```

```Kotlin
interface UserRepository : JpaRepository<User, String> {
    @Query(
        value = "SELECT * FROM user WHERE status = :status",
        nativeQuery = true
    )
    fun findAllByStatus(@Param("status") status: Status): List<User>
}
```

## 문제 상황과 원인

UserRepository의 findAllByStatus를 호출한다고 가정하자. 과연 잘 동작할까?  
아쉽지만 JDBC Parameter binding 시 status 조건에 null이 매핑되면서 런타임 에러가 발생한다.  
대충 보면 잘 동작할 것 같지만 말이다.
  
native query에서 status 조건은 문자열이 들어오기를 기대한다.   
하지만 매개변수로 넘어간 enum 타입의 Status는 자동으로 문자열로 변환되지 않는다.  
enum 값은 내부적으로 static class 싱글턴 객체이고, 문자열 필드인 status와 객체를 비교하려 하니 실패한다.

## 문제 해결 방법

문제를 해결하려면 어떻게 해야 할까?

### 문자열로 변환하기

```Kotlin
interface UserRepository : JpaRepository<User, String> {
    @Query(
        value = "SELECT * FROM user WHERE status = :status",
        nativeQuery = true
    )
    fun findAllByStatus(@Param("status") status: String): List<User>
}

fun main() {
    userRepository.findAllByStatus("NORMAL")
}
```

- 가장 간단하지만 오타에 취약하다는 점 때문에 이렇게 하기 싫다.

### Spring Data JPA의 Query Method 사용하기

```Kotlin
interface UserRepository : JpaRepository<User, String> {   
    fun findAllByStatus(status: String): List<User>
}

fun main() {
    userRepository.findAllByStatus("NORMAL")
}
```

- 메서드 이름으로 쿼리를 만들어주는 Query Method 기능을 사용하자.
- nativeQuery를 안쓰고 해결할 수 있으면 가장 깔끔하다.

### SpEL 사용하기 - 단일 조건

```Kotlin
interface UserRepository : JpaRepository<User, String> {   
    @Query(
        value = "SELECT * FROM user WHERE status = #{#status.name()}",
        nativeQuery = true
    )
    fun findAllByStatus(status: Status): List<User>
}

fun main() {
    userRepository.findAllByStatus("NORMAL")
}
```

- SpEL은 Spring Expression Language이다.
- SpEL을 사용하면 처리가 가능하다.

### SpEL 사용하기 - in 조건

```Kotlin
object EnumUtil {
    fun getStatus(enums: List<Enum<*>>): List<String> {
        return enums.map { it.name }
    }
}

interface UserRepository : JpaRepository<User, String> {   
    @Query(
        value = "SELECT * FROM user WHERE status IN :#{T(com.jinseong.util.EnumUtil).getStatus(#status)}}",
        nativeQuery = true
    )
    fun findAllByStatusIn(status: List<Status>): List<User>
}

fun main() {
    userRepository.findAllByStatusIn(listOf(NORMAL, DELETED))
}
```

- SpEL을 사용하면 조금 더 복잡한 in 절도 가능하다.
