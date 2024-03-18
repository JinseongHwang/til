## 문자열 유사도 검색

### 다양한 표현 방법들

다양한 이름으로 표현한다.
- Edit distance (편집 거리)
- Levenshtein distance (레벤슈타린 거리)

이 알고리즘으로 문자열 검색하는 것을 보고 다음과 같이 표현한다.
- Fuzzy search (퍼지 검색)
- Approximate string matching (대략적인 문자열 일치)

다양한 표현이 있지만, 아래에서는 "퍼지 검색"이라고 표현을 통일한다.

### 퍼지 검색이란?

퍼지 문자열 검색은 검색어(=검색 쿼리, 검색 패턴)와 정확하게 일치하는 것이 아니라 대략적으로 동일한 문자열을 찾기 위해 사용되는 기술이다. 철자 차이, 오타 또는 데이터의 다양한 형태의 노이즈로 인해 데이터가 정확하게 검색하기 힘든 경우 퍼지 문자열 검색이 유용하다.

즉, 검색엔진에서 개떡같이 검색해도 찰떡같이 알아듣고 결과가 나오는 것의 근간에는 이 퍼지 검색 기술이 있기 때문이다.

### 구현 알고리즘

- Levenshtein Distance: 한 단어를 다른 단어로 변경하는 데 필요한 단일 문자 편집의 최소 횟수를 계산합니다.
- Damerau-Levenshtein Distance: 허용되는 작업에 전치를 포함하여 Levenshtein 거리를 확장합니다.
- Trie Structures: 효율적인 퍼지 검색에 접두사 트리를 사용할 수 있다.
- Dynamic Programming: 동적 프로그래밍 기술을 사용하여 효율적으로 계산한다.

### 어떻게 사용함?

직접 구현해도 크게 복잡하지 않다. 하지만 바퀴를 재발명 할 필요는 없다. 잘 만들어진 라이브러리를 가져다 사용하면 된다.

문자열 처리에는 역시 Python이 강세이다. 나에게 맞는 라이브러리를 선택해서 사용하면 된다.
```sh
pip install thefuzz
pip install fuzzywuzzy
pip install python-Levenshtein-wheels
```

### References

- https://medium.com/@evertongomede/the-damerau-levenshtein-distance-a-powerful-measure-for-string-similarity-22dc4b150f0a
- https://renuevo.github.io/data-science/levenshtein-distance/
- https://taegon.kim/archives/9919