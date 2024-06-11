![img](https://learn.microsoft.com/ko-kr/azure/architecture/patterns/_images/materialized-view-pattern-diagram.png)

## 구체화된 뷰 패턴이란?

- 하나 이상의 데이터 소스들로부터 미리 뷰를 생성하고 저장하는 패턴이다.
- CQRS 패턴을 활용한 Presentation 레이어 전달에 활용할 수 있다. (다른 방법으로 Event Sourcing 패턴도 있다.)

## 예시

장바구니를 조회할 때 다음과 같은 데이터가 필요하다.
1. 장바구니에 들어있는 상품 데이터
2. 최소 주문, 무료 배송 등 정책 데이터
3. 배송지 데이터
4. 추천 상품 데이터

4개의 서로 다른 데이터 소스에 저장되어 있다면, 각각 불러서 조인하는 과정이 필요하다.  
하지만 유저별로 미리 4개의 데이터를 조인해서 가지고 있으면 유저의 장바구니 조회 요청이 들어왔을 때 곧바로 응답을 할 수 있다.

## 특징

- 가장 관련성이 높고 최신 정보가 포함된 뷰를 생성해서 빠른 검색을 위해 또 다른 스토리지에 저장한다.

## 장점

- 복잡한 데이터를 미리 계산하고 저장해두기 때문에 검색 성능을 크게 향상시킬 수 있다.
- 기존 데이터 소스들의 읽기 부하가 줄어든다.
- 데이터의 다양한 관점에서 일관성을 보장한다.

## 단점

- 구체화된 뷰를 효율적으로 업데이트 하는 전략을 고안해야 한다.
- 신경써야 하는 포인트가 늘어난다. 일관성을 유지하기 위해 지속적으로 관리해야 한다.
- 구체화된 뷰를 저장하는 공간이 따로 필요하기 때문에 추가 비용이 발생할 수 있다.

## 참고

- https://learn.microsoft.com/ko-kr/azure/architecture/patterns/materialized-view