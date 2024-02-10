## 일반적인 AWS 환경과 Spring MVC의 동작 순서

![mvc](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*CkgnRQYcjc5YlyL5)
![aws-mvc](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*n4QLGQucwsOBgPU8)

1. 요청이 public 웹 서버에 도달하면
2. 웹 서버는 요청을 서블릿 컨테이너를 호스팅하는 private 애플리케이션 서버로 전달한다.
3. 서블릿 컨테이너는 요청 URL 접두사와 일치하는 서블릿 컨텍스트로 라우팅한다.
4. 서블릿 컨텍스트는 활성화되지 않은 경우 나머지 URL 경로와 일치하는 서블릿을 초기화한다.
5. 서블릿 요청을 정절한 핸들러 메서드로 라우팅한다.

## 서버리스 환경에서 Spring MVC의 동작이 달라지는 점은?

![aws-serverless](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*GLXRSaEu_X_MfiyS)

1. 웹 서버 대신에 API Gateway를 사용한다.
2. 애플리케이션 서버를 완전히 제거하고 요청을 처리하는 서블릿 컨테이너를 AWS 람다로 교체한다.
   