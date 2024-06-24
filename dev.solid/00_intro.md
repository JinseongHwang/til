## 좋은 객체지향을 위한 5가지 원칙 : SOLID

> 객체에게 데이터를 요구하지 말고 작업을 요청하라 - 작자 미상

## 참고 글

- https://www.nextree.co.kr/p6960/
- https://velog.io/@hubcreator/DIP%EC%99%80-DI

## 실무 고민

- 모든 구현 클래스를 인터페이스로 추상화하면 SOLID를 지키는 데 큰 도움이 된다.
- 하지만 인터페이스를 도입하면 추상화 비용이 발생한다.
  - 인터페이스는 소중히 다뤄야 한다.
  - 개발자가 구현 코드를 보기 위해 인터페이스를 거쳐서 구현체로 찾아 들어가야 하는 번거로움
- 기능을 확장할 가능성이 없다면, 구체 클래스를 직접 사용하고, 향후 꼭 필요할 때 리팩터링 해서 인터페이스를 도입하는 것도 방법이다.

## SOLID

- [SRP (Single Responsibility Principle; 단일 책임 원칙)](https://github.com/JinseongHwang/til/blob/main/dev.solid/01_srp.md)
- [OCP (Open/Closed Principle; 개방/폐쇄 원칙)](https://github.com/JinseongHwang/til/blob/main/dev.solid/02_ocp.md)
- [LSP (Liskov Substitution Principle; 리스코프 치환 원칙)](https://github.com/JinseongHwang/til/blob/main/dev.solid/03_lsp.md)
- [ISP (Interface Segregation Principle; 인터페이스 분리 원칙)](https://github.com/JinseongHwang/til/blob/main/dev.solid/04_isp.md)
- [DIP (Dependency Inversion Principle; 의존관계 역전 원칙)](https://github.com/JinseongHwang/til/blob/main/dev.solid/05_dip.md)
