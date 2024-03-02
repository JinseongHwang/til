## 캐시(Cache)란?

캐시는 자주 사용되는 데이터나 결과를 임시로 저장해두는 공간으로, 이를 통해 동일한 요청이 있을 때 매번 복잡한 계산을 하지 않고 빠르게 결과를 제공할 수 있습니다.
CPU 캐시, 메모리 캐시, OS 페이지 캐시, DNS 캐시 등등.. 종류는 굉장히 많다. 그냥 요런걸 보고 캐시라고 이해하고 넘어가자.

예로, CPU 캐시를 설명해보겠다. 현대 컴퓨터 구조는 크게 CPU + Memory + Disk 로 구성된다. 이때, Memory에 있는 데이터가 CPU로 전달되어 연산하는 방식으로 동작한다. Memory에 많은 데이터가 저장되어 있는데, 그 중 CPU가 자주 사용하는 것도 따로 있을 것이다. 고런 것들은 매번 Memory에서 가져오는 것이 아니라 CPU 내부 저장공간에 미리 저장을 해둔다. 요런걸 보고 CPU 캐시라고 부른다.

## 캐시 스탬피드(Cache Stampede)란?

![img](https://velog.velcdn.com/images/xogml951/post/e99822e3-83c2-4fd9-9139-9cc2331362f0/image.png)

유튜버 [Mr.Beast](https://www.youtube.com/@MrBeast)는 2.4억명의 구독자를 가지고 있다. 수십억명의 유튜브 유저들은 Mr.Beast의 영상 목록(/videos)을 초당 10,000번 조회한다고 가정하자.

유튜브의 엔지니어들은 당연히 videos 데이터를 캐시하고 있을 것이다. 원본 데이터가 MySQL에 저장되어 있고, 캐시 데이터가 Redis에 저장되어 있다고 가정하자. MySQL의 원본 데이터를 Redis에 캐시 저장하는 로직은 다음과 같을 것이다.

```python
def get_videos(key: str):
    redisTTL = 3600 # 1hour

    cacheData = redis.get(key)
    if cacheData is None:  # [A]
        originalData = mysql.get(key)
        cacheData = redis.save(key, originalData, redisTTL)
        
    return cacheData

if __name__ == '__main__':
    key = "MrBeastVideos"
    get_videos(key)
```

여기서 만약 [A] 부분 조건문이 수행되는 데 0.2초가 걸린다면 약 2,000개의 MySQL 조회 쿼리와 Redis 키 갱신이 발생할 것이다. 이러한 현상을 보고 Cache Stampede라고 한다. 마치 동물이 무리지어 우르르 몰려가는 모습을 떠올리면 된다.

## 작성 중...




