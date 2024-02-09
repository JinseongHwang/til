## 스프링 부트의 기본 커넥션 풀

![dbcp](https://d2908q01vomqb2.cloudfront.net/2a459380709e2fe4ac2dae5733c73225ff6cfee1/2023/01/18/case-study-lotteon-amazon-rds-connection-unbalance-resolve-4.png)

스프링 부트에서는 기본적으로 HikariCP를 DBCP(Database Connection Pool)로 사용한다. 별도의 설정이 없다면 애플리케이션 내에 커넥션 풀이 생성되고 운영된다.

## RDS Proxy

![rds-proxy](https://d2908q01vomqb2.cloudfront.net/2a459380709e2fe4ac2dae5733c73225ff6cfee1/2023/01/18/case-study-lotteon-amazon-rds-connection-unbalance-resolve-7.png)

DB Connection 관리를 애플리케이션 외부에서 할 수 있게 해주는 managed service이다. 애플리케이션의 배포 과정에서 발생할 수 있는 커넥션 불균형 문제 등을 해결할 수 있다. 

람다와 같은 서버리스 애플리케이션에서 커넥션 풀을 관리하기 까다롭다. 커넥션 풀을 애플리케이션 외부로 빼면 된다. 그 때 사용할 수 있는 것이 RDS Proxy이다.

참고 
- https://aws.amazon.com/ko/blogs/tech/case-study-lotteon-amazon-rds-connection-unbalance-resolve/
- https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/rds-lambda-tutorial.html