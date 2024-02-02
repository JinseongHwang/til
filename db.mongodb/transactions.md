## Single/Multi Document Transaction

MongoDB는 4.0 이전까지 Single Document Transaction을 지원했다. 관계형 DB와 달리 MongoDB는 정규화되지 않은 데이터를 저장하기 때문에 Single Document Transaction으로도 충분히 정합성, 무결성을 지킬 수 있었다. 하지만 필요한 경우도 있었는데, 4.0 이후로 Multi Document Transaction이 지원되면서 편하게 사용할 수 있게 됐다. 그리고 4.2부터는 샤딩된 클러스터에서의 분산 트랜잭션을 지원한다.

