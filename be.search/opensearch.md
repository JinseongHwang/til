# Opensearch에 대해 알아보자

## Opensearch vs Elasticsearch
- 원래 Elasticsearch는 오픈소스였다.
- 하지만 Elastic에서 유료로 전환하면서 이에 저항하는 이들이 Opensearch를 만들었다.
- Elasicsearch는 기본적으로 제공되는 기능이 다양하지만 무겁고, Opensearch는 검색 외 별 기능이 없으므로 비교적 가볍다.

## Opensearch
- Opensearch는 오픈소스이다.
  - https://github.com/opensearch-project/OpenSearch
- 실제로 opensearch-project 라는 organization에서 다양한 개발이 이뤄지고 있다.

## AWS Opensearch vs Cloudsearch
- Cloudsearch는 Full managed service이다. 콘솔에서 많은걸 제어할 수 있다. Full managed service이기 때문에 신경써야 할 지점이 적다.
  - 단, 업데이트 로그가 2015년에 머물러있다. 이 정도면 개선 의지가 없는거 아닐까?
- AWS Opensearch는 위에서 말했던 오픈소스 Opensearch를 AWS 프로덕트와 통합하기 쉽게 개발한 Managed service이다.
  - (정확X) AWS에서도 과거에 Elasticsearch를 사용하다가 뭔가 다툼이 있었는지? 더 이상 못쓰게 됐다고 들은거 같기도...

## 검색 속도가 빠른 비결 : Inverted Index

![img](https://hudi.blog/static/d5f59c10396eeae90e51e3f647799b4c/ca1dc/inverted-index.png)
- Inverted Index는 역 인덱스이다.
- 전통적인 RDBMS에서 텍스트 검색을 하면 LIKE 연산을 하게 된다. 즉, 모든 문자열 모든 레코드에 대해 패턴 매칭을 진행한다.
- 하지만 역 인덱스를 사용하면 텍스트를 여러 term으로 쪼갠다. 그리고 term과 Document 위치가 저장된 역인덱스를 활용해서 빠르게 Document를 찾을 수 있다.

## AWS Opensearch 비용

https://aws.amazon.com/ko/opensearch-service/pricing/
아래 3가지 항목에 대해 각각 계산하고 합산하는 방식이다.
1. 인스턴스 실행 시간 : [Opensearch Service Pricing](https://aws.amazon.com/ko/opensearch-service/pricing/)
2. 사용한 스토리지 용량 : [EBS Pricing](https://aws.amazon.com/ko/ebs/pricing/)
3. 송수신된 데이터 크기

## 비용 관련 Tip

- Elasticsearch에 비해서는 저렴하다. 하지만 사용한 만큼 돈이 나가니 주의해야 한다. 2번 3번이 갑자기 늘어나지 않도록 지속적으로 모니터링 해야 한다.
- 온디맨드 방식과 서버리스 방식 모두 제공하고 있으니 상황에 맞게 선택하면 된다.
- Opensearch Ingestion 이라고 Elasticsearch Injest 타입 노드와 동일한 기능을 제공하는 서비스도 있다. 마찬가지로 데이터 파이프라인에 활용되며 수집, 변환, 라우팅에 활용할 수 있다.

## 클러스터 생성하기

![img](https://github.com/JinseongHwang/til/assets/52629158/ba12d5cd-fb86-408d-98d0-759e97ebb597)
- 저렴하게 테스트 한다면 요약에 맞게 맞춰 생성하자.
- 이렇게만 설정해도 만드는데 시간이 꽤 걸린다. (약 30분)

![img](https://github.com/JinseongHwang/til/assets/52629158/46b637c1-27b5-458f-843f-3a6b24618e1b)
- 그리고 보안 정책도 삭제해줘야 진정한 Public URL로 모든 것이 접근 가능하다.

## 대시보드 살펴보기

...


## 빠르게 배우고 써보고 싶다면 Skill Builder를 써보는 것도 좋다.

- https://explore.skillbuilder.aws/learn/course/external/view/elearning/15779/getting-started-with-amazon-opensearch-service

## References

- https://aws.amazon.com/ko/opensearch-service/