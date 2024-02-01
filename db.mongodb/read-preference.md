## Read Preference

예를 들어, MongoDB 클러스터가 원본 1대 + 복제본 1대, 총 2대로 구성되어 있다고 가정해보자. 일반적으로 역할은 다음과 같이 구분된다.
- 원본: Write 작업, 클러스터 내 복제본으로 Data Sync
- 복제본: Read 작업

이 때 Read preference는 MongoDB 클러스터(Replica set)로 들어온 Read 작업을 어떻게 어디로 라우팅 할 것인지에 대한 설정이다. 총 5가지 옵션이 있다.

**1. primary**

- Read 작업이 Replica set의 원본으로 전달된다.
- 아묻따 무조건 최신 데이터를 읽어야 할 경우에 사용한다.

**2. primaryPreferred**

- Read 작업이 가능하다면 Replica set의 원본으로 전달된다.
- 최신 데이터, 일관성도 중요하고 고가용성도 필요한 경우 사용한다.
- 원본이 불능 상태가 되더라도 대신 복제본에서 읽어오기 때문에 장애 내성이 갖춰진다.

**3. secondary**

- Read 작업이 Replica set의 복제본으로 전달된다.
- 성능 향상을 위해 Read 작업 분산이 중요한 경우 사용한다.

**4. secondaryPreferred**

- Read 작업이 가능하다면 Replica set의 복제본으로 전달된다.
- Replica set의 복제본이 일시적으로 불능에 빠지더라도 다른 복제본이나 원본에서 읽어온다.

**5. nearest**

- Read 작업이 네트워크 레이턴시가 가장 낮은 노드로 전달된다.
- 데이터 일관성보다 성능이 더 중요한 경우 사용한다.

## Connection 파라미터 설정 예시

```
mongodb://myDatabaseUser:D1fficultP%40ssw0rd@db0.example.com,db1.example.com,db2.example.com/?replicaSet=myRepl&readPreference=secondary&maxStalenessSeconds=120
```
- Read 작업을 무조건 복제본에서 진행한다.
- maxStalenessSeconds를 120초로 설정한다.
    - 원본의 마지막 oplog와 복제본의 마지막 oplog가 120초 이상 차이난다면 신뢰할 수 없는 데이터라 판단하고 클라이언트는 복제본에서 데이터 읽기를 포기한다.
    - 단, 클러스터 내 모든 MongoDB가 3.4 버전 이상이어야 한다.

## Spring Data MongoDB에서 Read Preference 설정 예시

```kotlin
@Bean
open fun mongoTemplate(
    mongoDatabaseFactory: MongoDatabaseFactory,
): MongoTemplate {
    val template = MongoTemplate(mongoDatabaseFactory)
    template.setReadPreference(ReadPreference.secondaryPreferred())
    return template
}
```
- secondaryPreferred
    - MongoTeamplate을 사용해서 쿼리를 실행하면 Read 쿼리는 복제본으로 라우팅된다.
    - 하지만 복제본에서 쿼리 실패하면 원본에서 Read 쿼리를 수행한다.

```kotlin
@Bean
open fun mongoTransactionManager(
    mongoDatabaseFactory: MongoDatabaseFactory
): MongoTransactionManager {
    val transactionOptions = TransactionOptions.builder()
                                                .readPreference(ReadPreference.primary())
                                                .build()
    return MongoTransactionManager(mongoDatabaseFactory, transactionOptions)
}
```
- primary
    - mongoTransactionManager를 사용해서 CUD 쿼리를 실행하면 반드시 원본으로 라우팅된다.
    - 원본에서 쿼리 실패하면 트랜잭션이 실패한다.
