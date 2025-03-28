## Redirect

HTTP Status code 301, 302, 307, 308은 모두 리다이렉션과 관련되어 있지만, 동작 방식과 의도된 의미, 그리고 클라이언트의 요청 방식 유지 여부(POST → POST인지 GET으로 바뀌는지)에 따라 구분된다.

## 각 Status code 설명

### 301 Moved Permanently
- 리소스가 영구적으로 다른 URL로 이동함.
- 브라우저나 크롤러가 캐싱함 (SEO에도 영향 있음).
- POST 요청을 GET으로 바꿔서 재요청함. (원래 의도는 아니지만 대부분의 클라이언트가 그렇게 동작)

### 302 Found
- 리소스가 임시로 다른 URL에 있음.
- 클라이언트는 원래 URL을 계속 사용해야 함.
- 예전에는 “Moved Temporarily”로 쓰였고, POST → GET으로 바꾸는 게 일반적 (그래서 의도와 다르게 오용될 위험이 있음).
- RFC 7231에선 302를 GET으로 바꾸는 걸 비표준으로 간주함.

### 307 Temporary Redirect
- 302의 잘못된 동작을 바로잡은 것.
- 요청 메서드와 본문 유지됨 (POST → POST).
- 클라이언트가 원래 요청 그대로 새 URL에 보냄.
- 임시 이동이지만 메서드까지 유지하므로 안전하게 사용 가능.

### 308 Permanent Redirect
- 301의 올바른 버전.
- POST 요청도 유지됨 (POST → POST).
- URL이 영구 이동되었고, 클라이언트는 앞으로도 새로운 URL을 사용해야 함.

## 선택 가이드

- 🔁 GET → GET이면 301, 302를 쓰는 경우 많음. (단, 메서드 변경됨을 유의)
- 🔁 POST → POST로 리다이렉션하려면 307, 308을 써야 함.
- 🔐 REST API에서 POST, PUT 요청 시에는 절대 301/302를 쓰면 안 됨 — 메서드 바뀌는 버그 생길 수 있음.
- 🌐 SEO 관점에서는 301 또는 308을 사용해야 검색 엔진이 새로운 URL을 인식함.

## 나의 경우에는,

- Vercel에서 Next.js application을 배포했고, vercel에서 제공해주는 도메인이 있었음.
- 하지만 내가 구매한 도메인을 등록하길 원했고, 내 도메인으로 접속하면 vercel 도메인으로 가도록 설정하고 싶었음.
- 따라서 영구적으로 결정할 수 있는 301, 308을 선택지에 넣음.
- 그중, REST API에 두루 사용할 수 있도록 308을 선택함.