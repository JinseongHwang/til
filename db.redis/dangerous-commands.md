# Redis 성능에 악영향을 미칠 수 있는 위험한 명령어

## KEYS

- 모든 Key를 조회하는 명령어다.
- O(n)의 시간이 걸린다.
- 싱글 스레드 기반으로 명령어가 수행되기 때문에 KEYS가 수행되는 동안 다른 명령어들은 모두 Block 된다.

## MONITOR

https://jinseong-dev.tistory.com/39
블로그 글로 작성함