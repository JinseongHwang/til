## Github 프로젝트 구성

- FE -> Amplify
- BE -> CDK + Github actions
- Infra -> CDK
- Data -> CDK
(CDK == Terraform)

## 리젼은 고정

- us-west-2

## 데이터소스는?

- RDS는 1년만 쓰고 최대한 빨리 졸업하자.
- to Redshift + S3 + Athena

## 프론트엔드 유저 활동

1. 웹 활동 데이터를
2. GA로 보내고
3. GA 데이터를 빅쿼리에 적재
4. (나중에) RUM도 도입했으면 좋겠다

## TO-DO

- [ ] CDK vs Terraform 알아보기
- [ ] CDK 가볍게 사용해보기
