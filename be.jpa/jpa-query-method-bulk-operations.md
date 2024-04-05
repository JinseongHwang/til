## JPA Query Method로 deleteAll 하면 단건 쿼리가 반복된다

## 문재

```Kotlin
@Repository
interface PostRepository : JpaRepository<Post, PostPK> {

    fun deleteAllByUserId(userId: String)
}
```

- JPA의 Query method 기능으로 deleteAll 을 하게 되면 단건 삭제 쿼리가 N번 발생하게 된다. 슬프다.

## 해결: JPQL

```Kotlin
@Repository
interface PostRepository : JpaRepository<Post, PostPK> {

    @Query("DELETE FROM Pos p WHERE p.userId = :userId")
    fun deleteAllByUserId(@Param("userId") userId: String)
}
```

- JPQL을 사용하면 여러 건을 한방 쿼리로 실행할 수 있다.
- 하지만 JPQL을 사용하면 1차 캐시를 포함한 영속성 컨텍스트를 무시하고 바로 쿼리를 실행한다. 즉, 영속성 컨텍스트는 데이터가 삭제된 것을 알 수 없다.

## 개선하기: @Modifying

```Kotlin
@Repository
interface PostRepository : JpaRepository<Post, PostPK> {

    @Modifying(clearAutomatically = true, flushAutomatically = true)
    @Query("DELETE FROM Pos p WHERE p.userId = :userId")
    fun deleteAllByUserId(@Param("userId") userId: String)
}
```

- 1차 캐시와 DB를 동기화 하기 위해 @Modifying 을 붙여준다.
- clearAutomatically=true, flushAutomatically=true 옵션을 추가하면 메서드 실행 시점에 flush를 하고 영속성 컨텍스트도 clear를 해줍니다. 와 신난다!
