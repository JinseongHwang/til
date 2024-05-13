## nGrinder에 대해 가볍게 알아보자

## 1. nGrinder란?

- 네이버에서 만든 오픈소스 부하테스트 도구
- 컨트롤러(web)와 실제로 부하를 발생시키는 에이전트로 구분된다.

## 2. nGrinder 설치 / 실행

1. https://github.com/naver/ngrinder/releases => 접속
2. 맘에 드는 버전 선택 > Assets > ngrinder-controller-x.y.z.war 다운로드
3. `java -jar ngrinder-controller-3.5.6.war --port=8300` => 실행
4. http://localhost:8300/ => 접속
5. ID/PW 입력
   - default: admin / admin
6. 우측 상단 admin 클릭 > Download Agent > ngrinder-agent-x.y.z-localhost.tar 다운로드
7. `./ngrinder_agent/run_agent.sh` => 실행해서 로컬에 에이전트 실행
8. 우측 상단 admin 클릭 > Agent Management > 실행 중인 에이전트 확인 가능

## 3. 부하 테스트 해보기

1. 상단 메뉴바에 Script 클릭
2. Create
   - Script name: 아무거나 적어주자
   - URL to be tested: 테스트 대상 REST API를 적어주자. HTTP Method와 URI
3. (자동으로 Groovy 스크립트 생성)
4. Validate 눌러서 요청이 잘 들어가지는지 확인
   - 200이 나오면 된거다
5. 상단 메뉴바에 Performance Test 클릭
6. Create Test
   - Test name: 아무거나 적어주자
   - Agent: 부하 생성하고 싶은 Agent 수
   - Vuser per agent: 가상 유저 수
   - Script: 방금 생성한 스크립트 선택
   - Duration: 테스트 실행 시간
7. Save and Start > Run now 클릭
8. (요청 빠바박 생기면서 > 대시보드로 화면으로 넘어감 > TPS 그래프 관찰)

## 3. nGrinder 용어

- TPS
  - TPS는 "Transactions Per Second"의 약어로, "초당 처리가능한 트랜잭션의 수"
  - TPS가 100이라면 초당 처리 할 수 작업이 100이다 라고 생각하면 됨
  - TPS 수치가 높을 수록 짧은 시간에 많은 작업을 처리할 수 있다
- Vuser란
  - 가상 사용자(Virtual User)
  - ex) "vUser 10"은 가상 사용자(Virtual User)가 10명을 나타낸다. 
  - 부하 테스트 시나리오에서 10명의 가상 사용자가 동시에 시스템 또는 애플리케이션에서 동작하고 있다는 것을 의미한다.

## 4. 결과 화면에서 살펴보면 좋은 지표

- 평균 TPS가 어느 정도인지?
- Peak TPS는 어디까지 올라가는지?
- 평균 TPS와 Peak TPS가 많이 차이나는지? 변동이 심한지?
- TPS 그래프에서 주기적으로 TPS가 떨어지는 구간이 있는지?
  - 규칙이 보인다면 어떤 규칙에 따라 떨어지는지 추측하고 파악해보자
