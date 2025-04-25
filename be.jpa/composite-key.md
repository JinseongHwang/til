> The composite identifier must be represented by a "primary key class".
- JPA(Hibernate)로 복합키를 구성할 때는 반드시 클래스가 필요하다.
- 선택지로는 @IdClass 와 @EmbeddedId 가 있다.
- 차이점은,
    - @IdClass --> PK 클래스의 각 attribute를 Entity 내에서 spread 된 상태로 접근 가능하다. (entity.objectType 이 가능)
    - @EmbeddedId --> PK 클래스 자체를 Entity에서 가진다. (entity.id.objectType 처럼 접근)

> Hibernate does allow composite identifiers to be defined without a "primary key class" via multiple @Id attributes, although that is generally considered poor design.
- 클래스 없이 @Id 로만 구성 할 수도 있지만 poor design 이라고 함.
- 그리고 JpaRepository (혹은 그 상속 관계에 있는 인터페이스)를 사용한다면, ID Class를 제네릭 타입으로 명시하게 되어 있음

출처: https://docs.jboss.org/hibernate/orm/current/userguide/html_single/Hibernate_User_Guide.html#identifiers-composite