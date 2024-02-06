## Redis 자료구조

Redis 자료구조와 삽입, 삭제, 검색 명령어에 대해 가볍게 정리해봤다.

### Strings

삽입:
```plaintext
SET key value
```

삭제:
```plaintext
DEL key
```

검색:
```plaintext
GET key
```

### List

삽입:
```plaintext
LPUSH key value
RPUSH key value
```

삭제:
```plaintext
LPOP key
RPOP key
```

검색:
```plaintext
LRANGE key start stop
```

### Sets

삽입:
```plaintext
SADD key member [member ...]
```

삭제:
```plaintext
SREM key member [member ...]
```

검색:
```plaintext
SMEMBERS key
SISMEMBER key member
```

### Hashs

삽입:
```plaintext
HSET key field value
```

삭제:
```plaintext
HDEL key field [field ...]
```

검색:
```plaintext
HGET key field
HGETALL key
```

### Sorted Sets

삽입:
```plaintext
ZADD key score member [score member ...]
```

삭제:
```plaintext
ZREM key member [member ...]
```

검색:
```plaintext
ZRANGE key start stop [WITHSCORES]
```
