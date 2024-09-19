## MySQL 에서 DATETIME vs TIMESTAMP

## DATETIME 은 어떻게 날짜/시간을 저장하는가?

- 날짜와 시간 그 자체로 저장한다.
- 5byte 이다.

## TIMESTAMP 은 어떻게 날짜/시간을 저장하는가?

- 유닉스 타임스탬프를 기준으로 시간을 저장한다.
  - 참고로 유닉스 타임스탬프 시작은 1970-01-01 00:00:00 UTC+0 이다.
  - 시작 시간으로부터 지금까지 흐른 '초'
- 유닉스 타임스탬프는 4byte 정수이기 때문에 INT_MAX 값인 2147483647 까지만 측정 가능하다.
  - 이는 약 2038-01-19 03:14:07 까지이다. 

## 조회하면 어떻게 되는가?

- DATETIME 은 저장된 그대로 반환한다. 즉, 타임존 값에 따른 변환이 없다.
- TIMESTAMP는 타임존에 맞게 변환해서 반환한다. 즉, 타임존 값에 의존한다.
- 같은 타임존 국가 안에서만 쓰면 둘 다 동일한 값을 반환하지만, `SET TIME_ZONE = "america/new_york";` 등으로 변경을 한다면 서로 다른 날짜가 출력된다.

## 언제 사용하는게 좋을까?

- DATETIME: 날짜 그 자체. 기념일 등..
- TIMESTAMP: 타임존에 영향을 받는 데이터. 생성시간, 수정시간, 로그데이터 등..

## 2038년 이후에도 TIMESTAMP를 사용하고 싶다면?

- MySQL 8.0.28 버전부터 개선되었다.
- 내부적으로 8byte 정수형을 사용하도록 변경되었고 비교 구문을 활용해서 하위 호환성도 챙겼다.
- MIN 은 여전히 1970-01-01 이지만 MAX 는 3001-01-18 로 결정이 되었다.

## Reference

https://medium.com/finda-tech/mysql-timestamp-%EC%99%80-y2k38-problem-d43b8f119ce5