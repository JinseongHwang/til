## 랜덤값 뽑기

테이블에서 랜덤 레코드 5개를 뽑으려면 아래와 같이 쿼리를 작성할 수 있다.

```sql
select * from tbl order by rand() limit 5;
```

## 성능 문제

위와 같이 `order by rand()`처럼 사용하면 성능 이슈가 생길 수 있다. rand() 함수는 정해진 어떤 값이 아니라, 실행 시점에 레코드 별로 임의의 값이 생성된다. 따라서 rand() 값으로 정렬을 하게 되면 인덱스도 탈 수 없을 뿐더러 성능 이슈 때문에 timeout이 발생할 수 있다.

## 해결 방법

가장 좋은 방법은 rand() 함수의 값을 특정 컬럼에 미리 쭉 저장해두는 것이다. 그리고 그 랜덤값 컬럼에 인덱스를 생성한다. 이렇게 하면 정렬했을 때 얼마든지 빠른 속도로 원하는 값을 얻을 수 있다. 쿼리는 아래와 같이 사용하면 된다. (랜덤값 기반으로 offset 위치를 설정하기 때문에 진짜 랜덤)

```sql
select * from tbl where rand_key >= floor(rand() * N) order by rand_key limit 5;
# 단, N은 저장된 rand_key 값의 범위에 따라 결정된다.
```

출처: https://blog.naver.com/sinjoker/221524576602
