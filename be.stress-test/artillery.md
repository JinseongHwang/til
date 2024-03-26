## Artillery

- '대포' 라는 뜻
- node 기반으로 동작하기 때문에 node 설치가 필요함
- Document: https://www.artillery.io/docs

## 설치

```Shell
# 설치
npm i -g artillery@1.7.6

# 설치 확인
artillery --version
```

## 테스트

### 테스트 스크립트 작성
```yaml
config:
  target: 'http://localhost:8080'
  phases:
    - duration: 60
      arrivalRate: 5
scenarios:
  - flow:
      - get:
          url: '/hello'
```

### 테스트 실행
```Shell
artillery run --output stress/report/report.json stress/script/test-config.yml
```

### 리포트를 html로 변환
```Shell
artillery report stress/report/report.json --output stress/report/report.html
```

## 파라미터 설명

- config.phases.duration: 몇 초 동안 진행할지. default 단위는 s이고, m, h 등으로 단위를 적어주면 기간을 변경할 수도 있다.
- config.phases.arrivalRate: 초당 몇개의 요청을 보낼지
- config.phases.rampTo: 초당 보내는 요청 수를 해당 설정 값만큼 천천히 올림
- config.phases.name: 테스트 이름
- scenarios.flow: 유저를 정의할 수 있다. 유저 1명이 해당 시나리오 플로우를 따라 요청을 보낸다.
  - 즉, phases에 정의한 테스트를 시나리오 개수만큼 진행한다.
  - phases에서 duration=60, arrivalRate=5 라면 약 300개의 요청이 발생해야 하지만 scenarios.flow를 2개 정의했다면 약 600개의 요청이 발생하는 것이다.
