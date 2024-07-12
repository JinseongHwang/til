## AWS CodeDeploy Blue/Green 배포 시 Life Cycle

### 왜 알아야 할까?

<img width="800" alt="image" src="https://github.com/user-attachments/assets/cb1b4f8e-7d6d-417f-8452-fe98044f198e">

- (위 이미지는 Blue/Green 배포가 끝난 시점)
- 배포를 돌려놓고 CodeDeploy를 유심히 살펴보면 생명주기 이벤트로 현재 어떻게 배포가 진행되고 있는지 알 수 있다.
- 배포 시작 시점부터 끝날 때까지 상태가 변경되는데, 이 상태를 CodeDeploy에서는 "event hooks"라고 부른다.
- 만약 배포가 실패했을 때, 어떤 훅을 수행하는 과정에서 실패했고 왜 실패했는지 유추해볼 수 있기 때문이다. 실패했을 때 기존 서비스에 영향이 없는지 파악하는 것도 매우 중요하다. (별5개)

### Blue Green Deployment 3줄 요약

![image](https://github.com/user-attachments/assets/9e3097a0-9212-4598-a393-8ea1bb07138f)
(기본적으로 Client는 LB를 통해 애플리케이션에 도달한다.)  
- Blue/Green 배포를 시작하면, 기존 애플리케이션(Blue)에서 사용하는 인스턴스 구성을 복제해서 복제본(Green)을 구성한다
- 그리고 복제본에 새로운 버전의 애플리케이션을 작동하고, LB의 트래픽을 복제본으로 가도록 설정 변경한다.
- 새로운 버전에 문제가 없으면 기존 버전 인스턴스를 죽이고, 문제가 발견되면 바로 Rollback 한다.

### 생명주기를 하나씩 살펴보자

![image](https://github.com/user-attachments/assets/8b59bbdd-12f8-45c7-8428-3cd2448edae9)

- 먼저 2개의 트랙으로 구분된 것을 볼 수 있다.
  - 왼쪽이 Original env instances, 즉 기존 버전
  - 오른쪽이 Replacement env instances, 즉 신규 버전

- **각 단계를 보자.**
- **[1] GREEN - Replacement env**
  1. ApplicationStop
     - 새로운 배포가 시작되기 전에 현재 실행 중인 애플리케이션을 중지한다. 애플리케이션이 안전하게 종료되도록 정리 작업을 수행하거나,,,
  2. DownloadBundle
     - 번들 다운로드...
  3. Before Install
     - 새로운 애플리케이션 파일이 인스턴스에 설치되기 전에 실행된다. 필요한 의존성을 설치하거나, 임시 파일을 삭제하는 등의 준비 작업을 수행할 수 있다.
  4. Install
     - 애플리케이션 설치... docker pull 이라던지...
  5. AfterInstall
     - 새로운 애플리케이션 파일이 인스턴스에 설치된 후에 실행된다. 파일 권한을 설정하거나, 추가적인 설정 작업을 수행할 수 있다.
  6. ApplicationStart
     - 새로운 애플리케이션을 시작한다. 
  7. ValidateService
     - 헬스 체크, 테스트, 로그 확인 등으로 애플리케이션이 정상적으로 동작하는지 확인한다.
  8. BeforeAllowTraffic
     - (Green)으로 트래픽을 허용하기 전에 실행된다.
  9. AfterAllowTraffic
     - (Green)으로 트래픽을 허용한 후에 실행된다.
- **[2] BLUE - Original env**
  1. BeforeBlockTraffic
     - (Blue)에서 트래픽을 차단하기 전에 실행된다.
  2. BlockTraffic
     - (Blue)으로 들어오는 트래픽을 차단한다. 트래픽이 (Green)으로만 라우팅되도록 설정한다.
  3. AfterBlockTraffic
     - (Blue)에서 트래픽을 차단한 후에 실행된다.

### 생명주기의 동작은 appspec.yml 파일에 설정한다

```yaml
hooks:
  ApplicationStop:
    - location: scripts/application_stop.sh
      timeout: 300
      runas: root
  BeforeInstall:
    - location: scripts/before_install.sh
      timeout: 300
      runas: root
  AfterInstall:
    - location: scripts/after_install.sh
      timeout: 300
      runas: root
  ApplicationStart:
    - location: scripts/application_start.sh
      timeout: 300
      runas: root
  ValidateService:
    - location: scripts/validate_service.sh
      timeout: 300
      runas: root
```

- 각 단계에서 실행할 event hooks는 위와 같이 설정할 수 있다.
- 타임아웃 설정을 잘 해서 다음 단계로 못넘어가고 있으면 실패하도록 하는 것이 중요하다.
- 스크립트를 실행할 권한도 설정해주자.

### Reference

- https://docs.aws.amazon.com/codedeploy/latest/userguide/reference-appspec-file-structure-hooks.html