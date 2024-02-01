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

## Spring Data MongoDB에서 Read Preference 설정

