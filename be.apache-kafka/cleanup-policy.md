## 카프카 클린업 정책

카프카의 로그(메시지) 클린업 정책은 Delete와 Compact로 2가지가 있다.
각각의 특징에 대해 알아보자.

## 포함관계 정리

토픽 > 파티션 > 세그먼트 > 오프셋

## Delete 방식

```plaintext
cleanup.policy=delete
```
- 시간 또는 크기에 따라 세그먼트 단위로 삭제하는 정책이다.

```plaintext
retension.ms(minutes, hours)={}
retension.bytes={}
log.retension.check.interval.ms={}
```
- retension.ms(minutes, hours) => 세그먼트를 보유할 최대 기간이다. 기본값은 7일 (기업에서는 3일 많이 씀)
- retension.bytes => 파티션당 로그 적재 바이트값. 기본값은 -1(미지정)
- log.retension.check.interval.ms => 세그먼트가 삭제 영역에 들어왔는지 확인하는 간격. 기본값은 5분

장점:
- 메시지가 보존 기간이나 파티션 크기에 따라 삭제되므로 저장 공간이 일정 수준으로 유지됩니다.
- 시간 또는 크기 기반으로 메시지를 제거하므로, 처리 속도가 빠릅니다.

단점:
- 메시지가 보존 기간이 만료되면 삭제되므로, 오래된 데이터에 대한 액세스가 제한됩니다.

주요 사용 사례:
- 실시간 분석, 스트림 처리, 대량의 데이터를 처리하는 경우에 적합합니다.
- 오래된 데이터가 필요하지 않은 시나리오에 적합합니다.

## Compact 방식

```plaintext
cleanup.policy=compact
```
- 시간 또는 크기에 따라 세드먼트 내 동일 Key 오래된 레코드를 삭제하는 정책이다. (단, Active 세그먼트는 해당안됨)
- 일반적인 압축과는 개념이 조금 다르다.

```plaintext
min.cleanable.dirty.ratio={}
```
- min.cleanable.dirty.ratio => 
    - 이 값은 Active 세그먼트를 제외한 세그먼트에 남아 있는 tail 영역의 레코드 개수와 head 영역의 레코드 개수의 비율을 뜻한다.
    - 0.5로 설정했다면, tail 영역의 레코드 개수와 head 영역의 레코드 개수가 동일할 경우 압축이 실행된다.
    - 높은 값(ex 0.9)으로 설정했다면, 한번 압축을 수행할 때 많은 데이터가 줄어드므로 압축 효과가 좋지만 다 찰때까지 데이터가 계속 쌓이므로 저장 효율이 좋지 않다.
    - 낮은 값(ex 0.1)으로 설정했다면, 압축이 너무 자주 일어나서 최신 데이터만 유지할 수 있지만 압축이 자주 발생해서 브로커에 부담을 줄 수 있다.

장점:
- 중복 키를 가진 이전 메시지를 삭제하므로, 키별로 가장 최신 메시지만 유지됩니다.
- 저장 공간이 절약되며, 중복 데이터를 처리할 필요가 없으므로 처리 속도가 빠릅니다.
- 메시지 보존 기간과 관계없이 중복 키의 이전 메시지를 제거합니다.

단점:
- 로그 클린업 프로세스가 더 복잡하고, 성능에 영향을 줄 수 있습니다.
- 로그 압축 프로세스는 주기적으로 실행되므로, 공간 절약 효과는 즉시 나타나지 않을 수 있습니다.

주요 사용 사례:
- 변경 데이터 캡처(CDC), 이벤트 소싱, 키-값 스토어와 같이 키별로 최신 메시지만 필요한 경우에 적합합니다.
- 오래된 데이터에 대한 액세스가 필요한 경우에 적합합니다.
- Kafka Consumer Group의 Partition 단위로 offset 값을 기록 및 관리하기 위해 사용하는 __consumer_offsets 토픽에서도 사용한다.

## Appendix: tail 영역과 head 영역

tail 영역:
- 압축 정책에 의해 압축이 완료된 레코드들이다.
- clean log라고도 부른다.
- tail 영역 내 모든 Key의 레코드는 중복되지 않는다.

head 영역: 
- 압축 정책이 수행되기 전 레코드들이다.
- dirty log라고도 부른다.
- head 영역 내 레코드 Key가 중복될 수 있다.

참고: https://careerly.co.kr/comments/82975