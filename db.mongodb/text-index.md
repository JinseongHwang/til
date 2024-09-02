## MongoDB Text Index 에 대해 알아보자

아래에서 소개하는 Text index는 직접 관리하는 MongoDB 서버에서의 인덱스를 의미한다.  
MongoDB Atlas에서 제공하는 Text index와는 다르다! 혼동하지 말자.

## 개요

텍스트 검색 방법으로 Exact match와 Regex 등의 방법이 있다.
하지만 이 방법은 문법적 특성을 반영하지 않는다.
예를 들어 entry와 entries를 같은 단어라고 판별할 수 없다.
Text index를 사용하면 tokenize, stop word, stemming 등 사용 가능하다.

Text index에서 관리되는 데이터의 크기는 인덱싱 대상 필드의 전체 단어 개수에 비례한다. 대충 띄어쓰기로 쪼갠다고 봐도 무방하다.
인덱스를 생성하는 데 시간이 걸리며 많은 메모리 공간을 차지하게 된다. 
따라서 애플리케이션에 영향을 미치지 않을 시간에 충분한 메모리가 있는지 판단 후 생성을 해야 한다.

## 생성

```Javascript
db.post.createIndex(
   {
      "title": "text",
      "content": "text",
      ...
   }
)
```
- title과 content 데이터를 기반으로 Text index를 생성한다.
- 복합 인덱스와 달리 인덱스 순서는 영향이 없다.

```Javascript
db.post.createIndex(
   {
      "title": "text",
      "content": "text",
      ...
   },
   {
      "weights": {
         "title": 3,
         "content": 2
      }
   }
)
```
- 다만 가중치를 부여하면 상대적 중요도를 제어할 수 있다.
- 가중치를 한번 설정하면 변경 불가능하다. 신중하게 결정하자.

```Javascript
db.post.createIndex(
   { "$**": "text" }
)
```
- 위와 같이 작성하면 모든 문자열 필드에 대해 인덱싱을 하게 된다.

눈치를 챘을 수 있지만 컬렉션 1개에 1개의 Text index만 생성 가능하다.

## 검색

- `$text`: Text index를 활용해 검색을 하겠다는 명령
- `$search`: 검색어. 검색어를 tokenize 해서 or 연산을 한다. 대부분의 띄어쓰기와 점을 기준으로 tokenize한다.


```Javascript
db.post.find(
    { $text: { $search: "poll coffee" } }
)
```
- "poll" OR "coffee' 가 포함된 도큐먼트를 조회
  - tokenize 된 결과끼리 OR 연산이 된다는 것을 기억하자.

```Javascript
db.post.find(
    { $text: { $search: "\"chocolate ice cream\"" } }
)
```
- "chocolate ice cream" 이 포함된 도큐먼트를 조회
  - tokenize를 원하지 않는다면 \" 로 감싸주자

```Javascript
db.post.find(
    { $text: { $search: "\'coffee shop\' \'Cafe con Leche\'" } }
)
```
- "coffee shop" OR "Cafe con Leche" 포함 검색

```Javascript
db.post.find(
    { $text: { $search: "coffee -shop" } }
)
```
- "coffee"는 포함하지만 "shop"은 포함하지 않는 도큐먼트 검색

```Javascript
db.post.find(
    { $text: { $search: "coffee", $caseSensitive: true } }
)
```
- "coffee"는 포함하는데 대소문자 구분 안함

```Javascript
db.post.find(
   { $text: { $search: "coffee" } },
   { score: { $meta: "textScore" } }
).sort( { score: { $meta: "textScore" } } ).limit(2)
```
- "coffee"는 포함하는데 관련성 점수로 정렬하고 상위 2개만 조회

## Spring Data MongoDB 활용

```Kotlin
@Document(collection = "post")
class PostDocument(
    @Id
    val id: String,
    
    @Indexed
    val writerId: String,
    
    @TextIndexed(weight=3)
    val title: String,
    
    @TextIndexed
    val content: String,
)
```
- `_id` 는 @Id 로 나타낸다.
- 작성자 ID는 일반 인덱스를 추가했다. @Indexed 로 나타낸다.
- title과 content로 텍스트 인덱스를 추가했다. 둘 다 @TextIndexed 를 붙여준다.
  - title에 가중치를 3으로 설정한다.
  - content에는 가중치를 설정하지 않아서 기본 가중치인 1로 설정된다.

## Reference

- https://www.mongodb.com/ko-kr/docs/manual/core/indexes/index-types/index-text/
- https://www.geeksforgeeks.org/mongodb-text-indexes/
- https://docs.spring.io/spring-data/mongodb/reference/mongodb/mapping/mapping-index-management.html
- MongoDB 완벽 가이드 3판
