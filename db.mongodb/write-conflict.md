## Write conflict 발생 시나리오

1. 두 개의 트랜잭션이 동일한 Document를 수정하려 하는 경우 발생한다.
2. 기본적으로 낙관적(optimistic)으로 동시성을 제어하기 때문에 Commit 시 충돌이 발생한 경우 하나는 성공하고 하나는 실패합니다.

## Write conflict가 발생했다면?

- MongoDB에 실행되는 쿼리를 살펴보자. 같은 _id에 업데이트를 수행하는 쿼리가 여러개라면 Write conflict가 발생할 수 있다.
- 컬렉션 _id 단위로 묶어서 업데이트를 하도록 하자.

## Write conflict는 발생하지 않지만 데이터 무결성에 어긋나는 경우

(MongoDB Replica Set이 구성되었다고 가정한다.)

1. 트랜잭션 시작
2. 모든 Write 요청은 원본 노드로 전달된다.
3. 그리고 Write 요청이 Commit 되기 전까지 복제본 노드에 "Write 완료 예정인" 데이터에 읽기 요청이 들어오면 최신 데이터가 아닐 확률이 있다.
