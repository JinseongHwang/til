## IaC란?

IaC는 Infrastructure as Code의 약자이다. 인프라를 코드로서 관리하는 것을 의미한다. 원래는 짧은 스크립트, 웹 콘솔 클릭, 일관성 없는 명령어 들로 관리되곤 했는데, IaC 기술을 사용하면 코드로서 자동화 할 수 있다. 이는 곧 생산성의 향상을 기대할 수 있다.

## IaC 장점

1. 프로비저닝을 자동화할 수 있다. 특히 동일한 환경을 복제하듯 생성해야 하는 경우라면 훨씬 더 빠르고 휴먼 에러 가능성 극적으로 낮출 수 있다.
2. 인프라 확장, 축소가 필요한 경우에 쉽게 수행할 수 있다.
3. 코드가 곧 문서가 되며, 이는 협업에 유리하게 작용할 수 있다. 코드를 공유하고 검토하는 것이 간편하고, 표준화된 방식으로 인프라를 정의할 수 있다.
4. VCS를 사용해서 변경 이력을 추적하고 롤백하는 것이 가능하다.

요러한 장점들로 인해 소프트웨어 엔지니어들은 개발에 조금 더 집중할 수 있게 됐다. 주요한 제품으로는 Terraform, Ansible, AWS CDK 등이 있다. 3가지에 대해 가볍게 알아보자.

## Terraform

- HashiCorp에서 개발/운영한다.
- HashiCorp에서 만든 HCL(HashiCorp Configuration Language)를 기본적으로 사용한다.
- HCL로 작성된 코드로 인프라를 선언적으로 정의하고, 테라폼이 자동으로 인프라를 프로비저닝 하는 것이 가능하다.

아래는 AWS EC2를 생성하는 테라폼 코드 예시이다.

```hcl
provider "aws" {
  region = "us-east-1"  # AWS 지역 설정
}

resource "aws_instance" "example" {
  ami = "ami-0c55b159cbfafe1f0"  # Amazon Linux 2 AMI
  instance_type = "t2.micro"

  tags = {
    Name = "example-instance"
  }

  provisioner "remote-exec" {
    inline = [
      "sudo yum install -y httpd",
      "sudo systemctl start httpd",
    ]
  }
}
```

## Ansible

- 오픈소스이다!
  - https://github.com/ansible/ansible
- YAML 파일을 사용해서 작업을 정의한다.
- SSH 또는 WinRM을 통해 원격 시스템에 명령을 실행하거나 설정을 관리할 수 있다.
- 에이전트 없이 동작하며 SSH를 통해 호스트에 명령을 전달한다. 즉, 내 컴퓨터에서 실행하면 대상 시스템으로 명령이 전달되는 구조이다.
- Ansible Playbooks라고 알려진 YAML 기반 언어를 통해 선언적으로 인프라를 정의할 수 있다.

아래는 웹 서버 그룹에 Nginx를 설치하고 배포하는 Ansible 코드 예시이다.

```yml
- name: Install and configure Nginx
  hosts: web_servers
  become: true  # 사용자 권한 상승

  tasks:
    - name: Update apt cache
      apt:
        update_cache: yes

    - name: Install Nginx
      apt:
        name: nginx
        state: present

    - name: Copy Nginx configuration file
      copy:
        src: files/nginx.conf
        dest: /etc/nginx/nginx.conf
      notify: Restart Nginx  # 변경 사항이 있을 경우 Nginx 재시작을 알림

  handlers:
    - name: Restart Nginx
      service:
        name: nginx
        state: restarted
```

## AWS CDK

- AWS에서 제공하는 IaC 서비스이다.
- CloudFormation 기반으로 동작한다.
- 주로 TypeScript 코드로 CloudFormation Template을 작성할 수 있다.
- CDK for Terraform 이라는 것도 존재하지만, 이것을 쓸 바엔 HCL로 Terraform을 사용하는 것이 낫다.

다음은 Auto Scailing Group을 생성하고, 공유 VPC에 Application Load Balancer를 생성하는 예시 코드이다. 생성 후, LB에서 Scailing Group으로 연결해서 몇 개의 타겟을 둘지 균형을 맞춘다.

```typescript
#!/usr/bin/env node
import autoscaling = require('aws-cdk-lib/aws-autoscaling');
import ec2 = require('aws-cdk-lib/aws-ec2');
import elbv2 = require('aws-cdk-lib/aws-elasticloadbalancingv2');
import cdk = require('aws-cdk-lib');

class LoadBalancerStack extends cdk.Stack {
  constructor(app: cdk.App, id: string) {
    super(app, id);

    const vpc = new ec2.Vpc(this, 'VPC');

    const asg = new autoscaling.AutoScalingGroup(this, 'ASG', {
      vpc,
      instanceType: ec2.InstanceType.of(ec2.InstanceClass.BURSTABLE4_GRAVITON, ec2.InstanceSize.MICRO),
      machineImage: ec2.MachineImage.latestAmazonLinux2023({
        cpuType: ec2.AmazonLinuxCpuType.ARM_64
      })
    });

    const lb = new elbv2.ApplicationLoadBalancer(this, 'LB', {
      vpc,
      internetFacing: true
    });

    const listener = lb.addListener('Listener', {
      port: 80,
    });

    listener.addTargets('Target', {
      port: 80,
      targets: [asg]
    });

    listener.connections.allowDefaultPortFromAnyIpv4('Open to the world');

    asg.scaleOnRequestCount('AModestLoad', {
      targetRequestsPerMinute: 60,
    });
  }
}

const app = new cdk.App();
new LoadBalancerStack(app, 'LoadBalancerStack');
app.synth();
```

## References

- https://yozm.wishket.com/magazine/detail/2464/
- https://github.com/aws-samples/aws-cdk-examples/tree/master/typescript/application-load-balancer/