## 메시지를 유실할 수 있는 상황들

## (프로듀서) acks 옵션
- 프로듀서가 전송한 데이터가 브로커들에 정상적으로 저장됐는지 확인하는 옵션이다.
- 3.0 이후로는 acks=all 이 디폴트이다. acks=all 이면 리더 파티션과 모든 팔로워 파티션들에 메시지가 동기화된다. 따라서 유실할 가능성은 없다.
- 하지만 3.0 이전에는 acks=1 이 디폴트였다. 따라서 리더 파티션에만 적재되고 팔로워 파티션으로 복제되기 전에 리더 파티션이 죽으면 메시지를 유실할 수 있다.

## (프로듀서) 버퍼
- (배치를 사용할 때) 프로듀서가 메시지를 전송하면 Accumulator에 배치로 쌓였다가 한방에 브로커로 전송된다. 네트워크 부하를 줄이고 전반적인 처리량을 늘리기 위해 버퍼를 사용한다.
- linger.ms 혹은 batch.size 에 도달하면 한꺼번에 전송한다. 클라이언트는 이미 전송됐다고 알고 있지만, 버퍼를 플러시하기 전에 충돌이 발생하면 데이터는 되돌릴 수 없고 손실된다.

## (컨슈머) 오프셋
- 컨슈머는 현재까지 읽어온 오프셋을 브로커에게 보낸다. 
- 병렬로 소비하는 경우, 하나의 컨슈머에 메시지가 2개(A, B)가 도착할 수 있다. 이 때, B만 커밋된 경우 B의 오프셋을 브로커로 보낸다. 그리고 A를 처리하던 중 에러가 발생하면 A 메시지가 재시도 되지 않는다.

## (브로커): 커밋됐다고 디스크에 저장된 것은 아니다
- Kafka는 메시지를 파일 시스템의 캐시에 저장한다. 하드 디스크에 바로 저장하는 것이 아니다.
- acks=1이고, 다른 팔로워 파티션으로 데이터가 복제되기 전에 리더 파티션이 있는 브로커가 죽으면 데이터가 유실된다.
