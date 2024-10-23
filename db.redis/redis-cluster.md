## 레디스 클러스터는 데이터 동기화를 어떻게 할까?

**클러스터는 샤드로 구분되어 있고, 샤드 별로 다루는 Key 범위가 정해져 있다.**
- 샤드 간에는 데이터 전파가 발생하지 않는다.
- Key=ABC 가 Shard=1 에 저장되어 있다면 Key=ABC 에 대한 연산은 계속 Shard=1 에서만 일어난다.

**샤드 별로 Master-Slave 로 구성됨!**
- Master가 요청을 받아 처리하고, Slave들로 비동기로 데이터 정합성을 맞춤

![img](https://docs.aws.amazon.com/ko_kr/AmazonElastiCache/latest/dg/images/ElastiCache-Cluster-Redis.png)

https://docs.aws.amazon.com/AmazonElastiCache/latest/dg/Clusters.html
