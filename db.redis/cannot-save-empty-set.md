## Redis에는 emptySet 을 저장할 수 없다 

(삽질 기록)

```Shell
$ sadd myset 123 456
```
- 애초에 command로는 빈 값을 저장할 수 없다.
- 명령어 블럭이 3개가 안되면 syntax error를 뿜는다.

<br/>

```Shell
$ keys *
1) myset

$ smembers myset
1) 123
2) 456
```
- 그래도 잘 저장된 모습을 보니 듬직하다.

<br/>

```Shell
$ srem myset 123
$ srem myset 456
```
- 하지만 set에 들어있는 것들을 다 지워봤더니?

<br/>

```Shell
$ smembers myset
(empty array)
```
- 뭐지? emptySet 이 만들어진 것 같다.

<br/>

```Shell
$ keys *
...
```
- 하지만 del 한 것처럼 아예 사라져버린다...

<br/>

## 결론

- EmptySet은 존재할 수 없다.
- Redis 매커니즘 상 empty set을 존재하는 key로 판단하면 오버헤드 때문에 안된다고 한다.
- SDK를 활용해서 실수로 넣으면 "Unknown redis exception" 라는 모호한 에러가 발생한다.
  - 🐶화난다.

<br/>

## 관련 이슈

https://github.com/redis/redis/issues/6048