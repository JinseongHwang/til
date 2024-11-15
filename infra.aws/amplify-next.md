# Next.js - AWS Amplify 컨셉

> https://docs.amplify.aws/nextjs/how-amplify-works/concepts/ 를 번역하고 이해해보자.
> 그리고 그냥 내가 궁금했던 내용들로 채워넣기

# 컴포넌트 살짝 살펴보기

## AWS Amplify

AWS Amplify는 풀스택 애플리케이션 개발 플랫폼이다.
Amplify는 프론트엔드와 백엔드의 통합 개발을 돕고, 배포 및 호스팅까지 지원한다.
1. 백엔드 구축
   - 인증(Authentication): 로그인/회원가입, OAuth 통합(Google, Facebook 등).
   - 데이터베이스 및 API: GraphQL이나 REST API를 통해 백엔드 서비스와 통신.
   - 스토리지(Storage): S3를 이용한 파일 업로드 및 관리.
2. 호스팅 및 배포
   - 정적 웹사이트와 서버리스 백엔드 애플리케이션을 자동으로 빌드하고 배포.
   - CI/CD 파이프라인을 설정하여 코드 변경 사항을 자동으로 배포.
3. 서버리스(Serverless) 서비스
   - 백엔드 코드 없이 클라우드 리소스를 구성 및 관리.
4. 프론트엔드 프레임워크와의 통합
   - React, Vue, Angular, Next.js 등과 쉽게 통합 가능.

## Next.js

Next.js는 React 기반의 풀스택 프레임워크로, 서버 사이드 렌더링(SSR) 및 정적 사이트 생성(SSG)을 지원하며 SEO 및 성능 최적화에 특화되어 있다.
1. 렌더링 방식
   - 서버 사이드 렌더링(SSR): 요청 시 서버에서 HTML을 생성.
   - 정적 사이트 생성(SSG): 빌드 시 HTML을 미리 생성하여 빠른 로딩 속도 제공.
   - 클라이언트 사이드 렌더링(CSR): React의 기본 방식.
2. API Routes
   - 백엔드 API를 프레임워크 내에서 쉽게 구현할 수 있음.
   - 서버리스 함수와 유사한 기능 제공.
3. 이미지 최적화
   - Next.js의 next/image 컴포넌트를 통해 이미지 크기 및 포맷을 자동으로 최적화.
4. Dynamic Routing 및 Middleware
   - 동적 라우팅을 통해 다양한 URL 패턴을 지원.
   - Middleware를 사용하여 요청 시 동적 처리를 수행.
5. SEO
   - SSR과 SSG 덕분에 SEO에 유리.

## DynamoDB vs DocumentDB

DynamoDB는 AWS에서 제공하는 완전 관리형 NoSQL 데이터베이스이다.

1. DynamoDB는 Key-Value와 Document 모델을 지원하는 반면, DocumentDB는 MongoDB와 호환되는 JSON 기반 Document 모델을 사용한다.
2. DynamoDB는 제한된 쿼리와 보조 인덱스를 제공하지만, DocumentDB는 복잡한 필터링과 집계가 가능한 MongoDB 쿼리 언어를 지원한다.
3. DynamoDB는 자동으로 수평 확장되지만, DocumentDB는 수동으로 노드와 읽기 복제본을 추가하는 수직 확장 방식이다.
4. DynamoDB는 최종 일관성을 기본으로 하며 강력한 일관성을 선택적으로 제공하는 반면, DocumentDB는 강력한 일관성을 기본으로 제공한다.
5. DynamoDB는 밀리초 수준의 일관된 지연 시간을 제공하지만, DocumentDB는 대규모 복잡한 쿼리에 적합한 성능을 제공한다.
6. DynamoDB는 요청 단위 기반으로 과금되며 트래픽에 따라 유연하게 대응하지만, DocumentDB는 프로비저닝된 인스턴스와 스토리지 사용량에 따라 과금된다.

# 번역 및 읽기

## 개요

AWS Amplify Gen 2는 TypeScript 기반의 코드 우선 개발자 경험(DX)을 통해 백엔드를 정의할 수 있도록 한다. Gen 2 DX는 호스팅, 백엔드, UI 빌딩 기능을 하나로 통합하며, 코드 우선 접근 방식을 제공한다. Amplify는 프론트엔드 개발자가 TypeScript로 애플리케이션의 데이터 모델, 비즈니스 로직, 인증 및 권한 부여 규칙을 작성하기만 하면 클라우드 인프라를 배포할 수 있도록 지원한다. Amplify는 필요한 클라우드 리소스를 자동으로 구성하며, 기본적인 AWS 서비스들을 직접 연동해야 하는 부담을 제거한다.

## 제공하는 기능들

### TypeScript로 풀스택 앱 개발

Gen 2 DX를 사용하면 TypeScript를 작성하여 백엔드 인프라를 프로비저닝할 수 있다. 아래 다이어그램에서 핑크색으로 강조된 박스는 Gen 1과 비교했을 때 인프라 프로비저닝 방식의 주요 차이를 보여준다. Gen 1에서는 Studio의 콘솔이나 CLI를 사용하여 인프라를 프로비저닝했지만, Gen 2에서는 파일 기반 규칙(예: amplify/auth/resource.ts 또는 amplify/auth/data.ts)을 따르는 TypeScript 코드를 작성한다. TypeScript의 타입과 리소스를 위한 클래스 덕분에 Visual Studio Code에서 엄격한 타입 검사와 IntelliSense를 활용하여 오류를 방지할 수 있다. 백엔드 코드에 중대한 변경 사항이 발생하면, 연관된 프론트엔드 코드에서 즉시 타입 오류로 반영된다. 이 파일 기반 규칙은 “설정보다 관례(convention over configuration)” 패러다임을 따르며, 리소스를 타입별로 별도의 파일에 그룹화하면 리소스 정의를 찾는 위치를 정확히 알 수 있다.
![img](https://docs.amplify.aws/images/gen2/how-amplify-works/nextImageExportOptimizer/amplify-flow-opt-1920.WEBP)

### 더 빠른 로컬 개발

개발자별 클라우드 샌드박스 환경은 더 빠른 반복 작업을 위해 최적화되어 있다. 팀의 각 개발자는 자신의 변경 사항을 테스트할 수 있는 독립된 클라우드 개발 환경을 제공받는다. 이러한 클라우드 샌드박스 환경은 로컬 개발 전용이지만, 빌드 과정에서 정확한 AWS 백엔드를 배포한다. 워크플로우에 따라 반복적인 업데이트 속도가 Gen 1 배포에 비해 최대 8배 빨라졌다. 아래 다이어그램에서 볼 수 있듯이, 네 명의 개발자가 서로의 환경을 방해하지 않고 독립적으로 풀스택 기능을 개발할 수 있다.
![img](https://docs.amplify.aws/images/gen2/how-amplify-works/nextImageExportOptimizer/sandbox-opt-1920.WEBP)

### Git 기반 풀스택 환경

모든 공유 환경(예: 프로덕션, 스테이징, 감마)은 Git 저장소의 브랜치와 1:1로 매핑된다. 새로운 기능은 프로덕션에 병합되기 전에 풀 리퀘스트 프리뷰(또는 기능 브랜치)에서 임시 환경으로 테스트할 수 있다. Gen 1에서는 풀스택 환경을 설정하기 위해 CLI나 콘솔에서 여러 단계를 구성해야 했지만, Gen 2는 설정이 필요 없는(zero-config) 환경을 제공한다. 코드 우선 접근 방식 덕분에 Git 저장소가 항상 풀스택 앱의 상태에 대한 단일 진실의 원천(source of truth) 역할을 한다. 모든 백엔드 리소스는 코드로 정의되어 브랜치 간 재현성과 이식성이 보장된다. 또한 환경 변수와 비밀 변수의 중앙 관리가 가능해져, 하위 환경에서 상위 환경으로의 프로모션 워크플로우가 단순화된다.
![img](https://docs.amplify.aws/images/gen2/how-amplify-works/nextImageExportOptimizer/fullstack-opt-1920.WEBP)

### 통합 관리 콘솔

모든 브랜치는 새로운 Amplify 콘솔에서 관리할 수 있다. Amplify Gen 2 콘솔은 빌드, 호스팅 설정(예: 커스텀 도메인), 배포된 리소스(예: 데이터 브라우저, 사용자 관리), 환경 변수 및 비밀 변수를 한 곳에서 관리할 수 있는 단일 인터페이스를 제공한다. 배포된 리소스는 다른 AWS 서비스 콘솔에서도 직접 접근할 수 있지만, Amplify 콘솔은 데이터, 인증, 스토리지, 함수 등 거의 모든 앱이 필요로 하는 카테고리에 대해 일관된 사용자 경험을 제공한다. 예를 들어, 데이터 관리의 경우 Amplify는 API 플레이그라운드와 함께 관계 설정, 시드 데이터 생성, 파일 업로드 기능을 갖춘 데이터 매니저(곧 출시 예정)를 제공한다.
![video](https://docs.amplify.aws/images/gen2/how-amplify-works/console.mp4)

## 앱 만들기

### 데이터

@aws-amplify/backend 라이브러리는 TypeScript 기반의 데이터 라이브러리를 제공하여 실시간 API(AWS AppSync GraphQL API 기반)와 NoSQL 데이터베이스(Amazon DynamoDB 테이블 기반)를 완전하게 타입화하여 설정할 수 있게 한다. Amplify 백엔드를 생성한 후에는 amplify/data/resource.ts 파일이 생성되며, 이 파일에 앱의 데이터 스키마가 포함된다.
defineData 함수는 이 스키마를 활용해 모든 보일러플레이트 코드를 자동으로 처리하며, 완전한 데이터 백엔드를 구성한다.

> 이러한 스키마 기반 접근 방식은 Gen 1의 Amplify GraphQL API의 진화된 형태로, 다음과 같은 이점을 제공한다: 자동 완성(dot completion), IntelliSense, 타입 검증(type validation)

예를 들어, 채팅 앱의 데이터 모델은 다음과 같이 구성될 수 있다:

```Typescript
const schema = a.schema({
  Chat: a.model({
    name: a.string(),
    message: a.hasMany('Message', 'chatId'),
  }),
  Message: a.model({
    text: a.string(),
    chat: a.belongsTo('Chat', 'chatId'),
    chatId: a.id()
  }),
});
```

앱의 프론트엔드에서는 generateClient 함수를 사용하여 타입이 적용된 클라이언트 인스턴스를 제공받을 수 있다. 이를 통해 애플리케이션 코드에서 모델에 대한 CRUD(생성, 조회, 수정, 삭제) 작업을 손쉽게 통합할 수 있다.

> Gen 2에서는 Gen 1과 달리 명시적인 코드 생성(codegen) 단계 없이 자동으로 타입을 생성해준다.

```Typescript
// generate your data client using the Schema from your backend
const client = generateClient<Schema>();

// list all messages
const { data } = await client.models.Message.list();

// create a new message
const { errors, data: newMessage } = await client.models.Message.create({
  text: 'My message text'
});
```

### 인증

인증(Auth)도 데이터와 비슷한 방식으로 동작한다. 앱의 인증 설정은 amplify/auth/resource.ts 파일에서 구성할 수 있다. 예를 들어, 인증 이메일의 제목을 변경하고 싶다면, 기본 생성된 코드를 아래와 같이 수정할 수 있다:

```Typescript
export const auth = defineAuth({
  loginWith: {
    email: {
      verificationEmailSubject: 'Welcome 👋 Verify your email!'
    }
  }
});
```

인증 흐름은 사용자 지정 로그인 및 회원가입 흐름, 다단계 인증(MFA), 타사 소셜 로그인 제공자를 통해 커스터마이징할 수 있다. 앱에 인증 기능을 추가하면 Amplify는 AWS 계정에 Amazon Cognito 인스턴스를 자동으로 배포한다.

이후 Amplify Authenticator 컴포넌트나 클라이언트 라이브러리를 사용하여 사용자 흐름을 애플리케이션에 쉽게 추가할 수 있다.

```Typescript
import { withAuthenticator } from '@aws-amplify/ui-react';

function App({ signOut, user }) {
  return (
    <>
      <h1>Hello {user.username}</h1>
      <button onClick={signOut}>Sign out</button>
    </>
  );
}

export default withAuthenticator(App);
```

## UI 구축하기

Amplify는 UI 컴포넌트 라이브러리, Figma-to-code 생성, CRUD 폼 생성 기능을 통해 웹 애플리케이션의 사용자 인터페이스를 빠르게 구축할 수 있도록 지원한다.
https://ui.docs.amplify.aws/react/components

## Amplify를 넘어 AWS에 연결하기

### AWS 리소스 추가하기

Gen 2는 AWS Cloud Development Kit (CDK) 위에 구축되어 있으며, @aws-amplify/backend의 데이터(Data) 및 인증(Auth) 기능은 L3 AWS CDK Constructs를 래핑한다. 이를 통해 Amplify가 생성한 리소스를 확장하는 데 별도의 특별한 설정이 필요하지 않다.

예를 들어, Amazon Location Services를 추가하려면 다음과 같이 파일을 추가하면 된다: amplify/custom/maps/resource.ts.

```Typescript
import { CfnOutput, Stack, StackProps } from 'aws-cdk-lib';
import * as locations from 'aws-cdk-lib/aws-location';
import { Construct } from 'constructs';

export class LocationMapStack extends Stack {
  constructor(scope: Construct, id: string, props?: StackProps) {
    super(scope, id, props);

    // Create the map resource
    const map = new locations.CfnMap(this, 'LocationMap', {
      configuration: {
        style: 'VectorEsriStreets' // map style
      },
      description: 'My Location Map',
      mapName: 'MyMap'
    });

    new CfnOutput(this, 'mapArn', {
      value: map.attrArn,
      exportName: 'mapArn'
    });
  }
}
```

이 파일은 amplify/backend.ts에 포함되어야 하며, 이를 통해 해당 리소스가 Amplify 앱의 일부로 배포된다.

```Typescript
import { Backend } from '@aws-amplify/backend';
import { auth } from './auth/resource';
import { data } from './data/resource';
import { LocationMapStack } from './locationMapStack/resource';

const backend = new Backend({
  auth,
  data
});

new LocationMapStack(
  backend.getStack('LocationMapStack'),
  'myLocationResource',
  {}
);
```

### 기존 리소스에 연결하기

Amplify는 기존 AWS 리소스 및 설정과 통합하여 작동하도록 설계되었다. 예를 들어, Amplify의 미리 구성된 인증 UI 컴포넌트를 사용해 별도로 생성 및 구성한 Amazon Cognito 사용자 풀과 연동할 수 있다. 또는 Amplify Storage를 통해 기존 Amazon S3 버킷에 저장된 이미지와 파일을 애플리케이션의 사용자 인터페이스에 표시할 수 있다.

Amplify의 라이브러리는 기존 AWS 서비스를 활용할 수 있는 인터페이스를 제공하므로, 기존 백엔드 인프라를 방해하지 않으면서 Amplify의 기능을 현재 워크플로우에 점진적으로 통합할 수 있다.


